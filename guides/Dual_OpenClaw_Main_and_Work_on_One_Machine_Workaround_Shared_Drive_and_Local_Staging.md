# Practical Guide: Main + Work OpenClaw on One Machine (Workaround Model: Shared Google Drive + Local Staging Handoff)

## Purpose
This guide documents the **pragmatic workaround** version of the dual-instance setup.

Use this when you want:
- two OpenClaw instances on one host,
- separate Linux users and separate runtime/service isolation,
- but **not yet full identity separation** for every external system.

In this model:
- **main** and **work** still run under different Linux users,
- the instances may still use different chat/OAuth identities where needed,
- but the workflow allows:
  - a **local shared staging folder** for handoff between users, and
  - **the same Google account / same Drive workspace** to be used by both instances.

This is a **workaround guide**, not the ideal long-term boundary model.

---

## When to use this guide
Use this guide if:
- you need the dual-instance system working now,
- you want operational isolation at the Linux/service level,
- you still want both instances to access the same Google Drive content,
- and you accept that Google-side separation is intentionally relaxed.

Do **not** use this guide as the default architecture if you want strict work/personal data boundaries. In that case, use the clean split model instead:
- separate Linux users
- separate provider identities
- separate GitHub account
- separate Google account / Drive boundary

---

## Architecture (workaround state)

- Main state: `/home/huatyou/.openclaw`
- Work state: `/home/lidoclaw/.openclaw`
- Main service: `openclaw-gateway.service` under user `huatyou` on port `18789`
- Work service: `openclaw-gateway.service` under user `lidoclaw` on port `18790`
- Shared handoff path: `/srv/openclaw-shared/staging/shared`
- Google Drive: shared/overlapping access via the **same Google account / same Drive workspace**

Recommended mental model:
- **runtime isolation** is real,
- **filesystem handoff** is explicit through staging,
- **Google boundary** is intentionally shared in this workaround.

---

## Key tradeoff
This setup is useful because it keeps the two OpenClaw runtimes from stepping on each other while still allowing easy file exchange and a common Google workspace.

But the tradeoff is important:
- Linux/service isolation is strong
- Google data isolation is weak by design

So this is best treated as:
- operationally practical,
- security-compromised compared with a full identity split,
- acceptable only when that compromise is intentional.

---

## Prerequisites

### Required
- SSH/shell access as `huatyou`
- `sudo` access
- OpenClaw installed and callable (example used here: `/home/huatyou/.npm-global/bin/openclaw`)
- Existing main instance
- Existing or planned work instance under `lidoclaw`

### Recommended
- Separate Telegram bot token for the work bot
- Separate OpenAI/Codex auth for work if you want model/account separation
- Shared Google Drive content organized under a controlled folder (for example, `Lido`)

---

## High-level design rules

1. **Keep each instance in its own Linux user**
   - `huatyou` for main
   - `lidoclaw` for work

2. **Keep each instance’s OpenClaw state in its own home directory**
   - never point both at the same `.openclaw`

3. **Use a neutral staging folder for cross-user handoff**
   - `/srv/openclaw-shared/staging/shared`

4. **Allow same Google account / Drive where needed**
   - this is the workaround layer
   - document it clearly so nobody mistakes it for strict separation

5. **Prefer explicit handoff over hidden cross-reading**
   - if one side generates something for the other, place it in staging intentionally

---

## Setup summary

### Runtime split
You still want the dual-user setup:
- main instance owned by `huatyou`
- work instance owned by `lidoclaw`
- different ports
- different service lifecycles
- separate `.openclaw` directories

That gives you the important operational separation.

### Shared staging
You then add a shared group and shared directory so the two instances can intentionally exchange files.

### Shared Google Drive
Instead of splitting Google identities, both instances are allowed to use the same Google account / Drive workspace.

This means:
- both may see the same Drive files,
- both may work within the same designated Drive folder,
- the Drive layer is not a security boundary in this mode.

---

## Step 1 — Keep the dual-user OpenClaw setup

If the dual-user setup is already working, keep it as-is.

Expected state:
- `/home/huatyou/.openclaw` for main
- `/home/lidoclaw/.openclaw` for work
- port `18789` for main
- port `18790` for work

If needed, verify as each user:

```bash
/home/huatyou/.npm-global/bin/openclaw gateway status
/home/huatyou/.npm-global/bin/openclaw channels status
```

For the work user, run from a `lidoclaw` shell.

---

## Step 2 — Create the shared staging group

```bash
sudo groupadd openclaw-shared 2>/dev/null || true
sudo usermod -aG openclaw-shared huatyou
sudo usermod -aG openclaw-shared lidoclaw
```

This allows both users to collaborate through one explicit filesystem boundary.

---

## Step 3 — Create the shared staging directory

```bash
sudo mkdir -p /srv/openclaw-shared/staging/shared
sudo chown -R root:openclaw-shared /srv/openclaw-shared
sudo chmod -R 2770 /srv/openclaw-shared
sudo setfacl -m g:openclaw-shared:rwx /srv/openclaw-shared/staging
sudo setfacl -d -m g:openclaw-shared:rwx /srv/openclaw-shared/staging
sudo setfacl -m g:openclaw-shared:rwx /srv/openclaw-shared/staging/shared
sudo setfacl -d -m g:openclaw-shared:rwx /srv/openclaw-shared/staging/shared
```

Why this matters:
- `2770` keeps group ownership sticky
- ACLs make inherited group write reliable
- both sides can drop handoff artifacts into the same place

---

## Step 4 — Refresh runtime group membership correctly

After adding users to `openclaw-shared`, do **not** assume existing OpenClaw processes immediately inherit the new group.

First test from shell:

```bash
id && ls -ld /srv/openclaw-shared /srv/openclaw-shared/staging /srv/openclaw-shared/staging/shared && touch /srv/openclaw-shared/staging/shared/.group-access-test && ls -l /srv/openclaw-shared/staging/shared/.group-access-test && rm /srv/openclaw-shared/staging/shared/.group-access-test
```

Expected:
- `id` shows `openclaw-shared`
- `touch`, `ls`, and `rm` all succeed

If shell access works but OpenClaw still gets permission errors, the running service context is stale.

The validated fix is to restart the entire user session manager for that user, not just the service unit.

Example for `lidoclaw`:

```bash
loginctl terminate-user lidoclaw
```

Then log back in as `lidoclaw` and start the user service again:

```bash
systemctl --user start openclaw-gateway
```

Then verify the new process has the expected groups:

```bash
ps -ef | grep -i openclaw-gateway | grep lidoclaw | grep -v grep
cat /proc/<NEW_PID>/status | grep '^Groups:'
```

Expected result includes the shared group.

---

## Step 5 — Use staging as the handoff contract

Recommended convention:
- files produced for the other instance go into:
  - `/srv/openclaw-shared/staging/shared`
- receiving side copies them into its own workspace intentionally

Example:

```bash
# as huatyou
echo "from-main $(date -Is)" > /srv/openclaw-shared/staging/shared/main-note.txt

# as lidoclaw
cat /srv/openclaw-shared/staging/shared/main-note.txt
echo "from-work $(date -Is)" >> /srv/openclaw-shared/staging/shared/main-note.txt
cat /srv/openclaw-shared/staging/shared/main-note.txt
```

Then each side can import into its own workspace:

```bash
cp /srv/openclaw-shared/staging/shared/main-note.txt /home/huatyou/.openclaw/workspace/from-shared.txt
cp /srv/openclaw-shared/staging/shared/main-note.txt /home/lidoclaw/.openclaw/workspace/from-shared.txt
```

This keeps workspaces isolated while still enabling deliberate exchange.

---

## Step 6 — Shared Google Drive model

In this workaround, both instances may use the **same Google account** or at least the same Drive workspace/folder set.

Recommended boundary rule even in workaround mode:
- keep Dino’s Google work inside one designated Drive folder
- for example, the `Lido` folder

That gives you at least an organizational boundary, even though it is not a true account boundary.

### What this means operationally
- main and work can both access the same Drive data set
- outputs do not need to be duplicated across two separate Google accounts
- the Drive folder becomes the shared cloud workspace

### What this means security-wise
- Google Drive is **not** separating personal vs work access here
- mistakes or broad automations may affect the same Drive content from both instances
- auditing who changed what becomes less clean than with separate accounts

So the safe phrasing is:
- shared Google Drive is allowed here because this is the workaround guide,
- not because it is the ideal long-term architecture.

---

## Recommended Google Drive discipline in workaround mode

If both instances share the same Google account / Drive, use stricter folder discipline:

- Define one primary Drive folder for Dino work
- Avoid letting either instance write broadly across unrelated Drive locations
- Treat that folder as the allowed workspace
- Keep naming conventions obvious, such as:
  - `main/`
  - `work/`
  - `handoff/`
  - `exports/`

Example folder layout inside the shared Drive folder:

```text
Lido/
  main/
  work/
  handoff/
  exports/
```

This does not create a real security boundary, but it reduces confusion.

---

## GitHub stance in this workaround guide

This guide is specifically about the shared Drive + local staging compromise.

For GitHub, you should still prefer as much separation as practical.

Recommended order:
1. best: separate GitHub account for work
2. acceptable temporary compromise: shared GitHub identity only if explicitly intentional and understood

Reason:
- local staging is a contained filesystem compromise,
- shared Google Drive is a contained cloud-workspace compromise,
- but shared GitHub identity affects authorship, tokens, repo access, and audit trails more broadly.

So even in workaround mode, I would still lean toward a separate GitHub account if possible.

---

## Verification checklist

### Runtime isolation
- main service runs as `huatyou`
- work service runs as `lidoclaw`
- ports differ (`18789` / `18790`)
- each has its own `.openclaw`

### Shared staging works
- both users are in `openclaw-shared`
- both can create/read/update files in `/srv/openclaw-shared/staging/shared`
- active OpenClaw runtime also has the shared group in its process context

### Google workaround is explicit
- operators know the same Google account / same Drive workspace is being shared intentionally
- the Drive folder boundary is documented
- nobody mistakes this for strict separation

---

## Common pitfalls

- Assuming Linux user separation automatically means Google data separation
- Forgetting that old running services may not pick up new group membership
- Testing shell access successfully and concluding the agent runtime must also have the same permissions
- Letting the shared staging folder become an unstructured junk drawer
- Treating shared Google Drive as “good enough security” instead of what it actually is: a convenience compromise

---

## Recommended wording for operators

If you hand this to another operator, the clean description is:

> We run main and work OpenClaw as separate Linux users with separate state and services. For pragmatic collaboration, we also maintain a shared local staging folder for handoff and currently allow both instances to use the same Google Drive workspace. This is an intentional workaround for convenience, not the strict final security model.

---

## Bottom line

This workaround model is valid when the priorities are:
- stable dual-instance runtime separation,
- easy handoff between instances,
- shared cloud workspace convenience.

It is **not** the best model for strict identity/data isolation.

Use it when you want practical operations now.
Move to separate Google identity / Drive boundaries later if you want the cleaner long-term design.
