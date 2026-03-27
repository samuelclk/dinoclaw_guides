# Practical Guide: Main + Work OpenClaw on One Machine (Two Linux Users, Two ChatGPT OAuth Accounts)

## Goal
Run two isolated OpenClaw instances on one host:

- **main** instance under user `huatyou` on port `18789`
- **work** instance under user `workclaw` on port `18790`
- separate files, services, and OAuth accounts

This is the operator-safe method we validated.

---

## Architecture (final state)

- Main state: `/home/<user>/.openclaw`
- Work state: `/home/<work-user>/.openclaw`
- Main service: `openclaw-gateway.service` (user `huatyou`, port `18789`)
- Work service: `openclaw-gateway.service` (user `workclaw`, port `18790`)
- OAuth accounts differ by `accountId` in each user’s `auth-profiles.json`

---

## Prerequisites

### Required
- SSH/shell access as `huatyou`
- `sudo` access
- OpenClaw binary available (we used `/home/<user>/.npm-global/bin/openclaw`)
- Two ChatGPT accounts for OAuth

### Recommended
- Separate Telegram bot token for work bot

---

## 10-minute quickstart (experienced operators)

```bash
# 0) backup existing work profile state
TS=$(date +%F-%H%M%S)
BK="/home/<user>/migration-backups/openclaw-work-$TS"
mkdir -p "$BK"
cp -a /home/<user>/.openclaw-work "$BK"/ 2>/dev/null || true

# 1) create/prepare work user
sudo adduser --disabled-password --gecos "" workclaw || true
sudo setfacl -m u:workclaw:x /home/huatyou

# 2) copy work state into workclaw home
sudo -u workclaw -H mkdir -p /home/<work-user>/.openclaw
sudo rsync -a /home/<user>/.openclaw-work/ /home/<work-user>/.openclaw/
sudo chown -R workclaw:workclaw /home/<work-user>/.openclaw

# 3) switch to workclaw for the rest of this block
sudo -iu workclaw

# 4) set core config (as workclaw)
/home/<user>/.npm-global/bin/openclaw config set agents.defaults.workspace /home/<work-user>/.openclaw/workspace
/home/<user>/.npm-global/bin/openclaw config set gateway.port 18790
/home/<user>/.npm-global/bin/openclaw config set agents.defaults.model.primary openai-codex/gpt-5.4
/home/<user>/.npm-global/bin/openclaw config set agents.defaults.model.fallbacks '[]'

# 5) ensure .env exists and load token var expected by CLI (as lidoclaw)
[ -f ~/.openclaw/.env ] || touch ~/.openclaw/.env
set -a; source ~/.openclaw/.env; set +a
export OPENCLAW_GATEWAY_TOKEN="${OPENCLAW_GATEWAY_TOKEN_WORK:-$OPENCLAW_GATEWAY_TOKEN}"

# 6) install/reinstall service from workclaw session
/home/<user>/.npm-global/bin/openclaw gateway install --force
systemctl --user daemon-reload
systemctl --user enable openclaw-gateway.service
systemctl --user restart openclaw-gateway.service

# 7) OAuth (work account)
/home/<user>/.npm-global/bin/openclaw models auth login --provider openai-codex

# 8) verify
/home/<user>/.npm-global/bin/openclaw gateway status
/home/<user>/.npm-global/bin/openclaw channels status
```

---

## Step-by-step migration (careful path)

## Step 1 — Snapshot before changes (required)

```bash
TS=$(date +%F-%H%M%S)
BK="/home/<user>/migration-backups/openclaw-work-$TS"
mkdir -p "$BK"
cp -a /home/<user>/.openclaw-work "$BK"/ 2>/dev/null || true
cp -a /home/<user>/.config/systemd/user/openclaw-gateway-work.service "$BK"/ 2>/dev/null || true
cp -a /home/<user>/.bashrc "$BK"/bashrc.before
```

## Step 2 — Create `workclaw` user

```bash
sudo adduser --disabled-password --gecos "" workclaw
```

If you already created `openclawwork` and want rename:

```bash
sudo usermod -l workclaw openclawwork
sudo groupmod -n workclaw openclawwork
sudo usermod -d /home/workclaw -m workclaw
```

## Step 3 — Allow traversal to shared binary path

```bash
sudo setfacl -m u:workclaw:x /home/huatyou
```

## Step 4 — Copy work state into new user home

```bash
sudo -u workclaw -H mkdir -p /home/<work-user>/.openclaw
sudo rsync -a /home/<user>/.openclaw-work/ /home/<work-user>/.openclaw/
sudo chown -R workclaw:workclaw /home/<work-user>/.openclaw
```

## Step 5 — Login as `workclaw` and fix config

```bash
sudo -iu workclaw
/home/<user>/.npm-global/bin/openclaw config set agents.defaults.workspace /home/<work-user>/.openclaw/workspace
/home/<user>/.npm-global/bin/openclaw config set gateway.port 18790
/home/<user>/.npm-global/bin/openclaw config set agents.defaults.model.primary openai-codex/gpt-5.4
/home/<user>/.npm-global/bin/openclaw config set agents.defaults.model.fallbacks '[]'
```

## Step 6 — Ensure work `.env` is present

```bash
[ -f ~/.openclaw/.env ] || touch ~/.openclaw/.env
```

Make sure it contains at least:

```text
TELEGRAM_BOT_TOKEN=<work_bot_token>
OPENCLAW_GATEWAY_TOKEN_WORK=<work_gateway_token>
```

## Step 7 — Install service from workclaw session

```bash
set -a
source ~/.openclaw/.env
set +a
export OPENCLAW_GATEWAY_TOKEN="${OPENCLAW_GATEWAY_TOKEN_WORK:-$OPENCLAW_GATEWAY_TOKEN}"

/home/<user>/.npm-global/bin/openclaw gateway install --force
systemctl --user daemon-reload
systemctl --user enable openclaw-gateway.service
systemctl --user restart openclaw-gateway.service
```

## Step 8 — OAuth for work account

```bash
/home/<user>/.npm-global/bin/openclaw models auth login --provider openai-codex
```

Sign in with the **work** ChatGPT account.

## Step 9 — Verify service, channels, and account split

### As `workclaw`
```bash
/home/<user>/.npm-global/bin/openclaw gateway status
/home/<user>/.npm-global/bin/openclaw channels status
ss -ltnp | grep 18790
```

### Check account split
```bash
jq -r '.profiles["openai-codex:default"].accountId // "missing"' /home/<user>/.openclaw/agents/main/agent/auth-profiles.json
jq -r '.profiles["openai-codex:default"].accountId // "missing"' /home/<work-user>/.openclaw/agents/main/agent/auth-profiles.json
```

Expected: two different `accountId` values.

---

## Final permissions hardening (do this last)

After everything is working, lock it down:

```bash
# ownership
sudo chown -R workclaw:workclaw /home/workclaw

# directory permissions
sudo chmod 700 /home/workclaw
sudo chmod 700 /home/<work-user>/.openclaw
sudo chmod 700 /home/<work-user>/.ssh 2>/dev/null || true

# sensitive files
sudo chmod 600 /home/<work-user>/.openclaw/.env 2>/dev/null || true
sudo find /home/<work-user>/.openclaw -type f -name '*.json' -exec chmod 600 {} \;
# if any tooling later fails due to strict JSON permissions, relax only non-sensitive JSON files to 640.

# remove old ACLs you no longer need
sudo setfacl -x u:openclawwork /home/huatyou 2>/dev/null || true
```

Optional stricter boundary: remove unnecessary sudo rights from users that should not cross-read each other.

---

## Shared staging between users (optional, recommended for handoff)

Keep each agent’s workspace isolated in its own home directory, and add one neutral shared handoff path.

### Create shared staging

```bash
# one-time setup
sudo groupadd openclaw-shared 2>/dev/null || true
sudo usermod -aG openclaw-shared huatyou
sudo usermod -aG openclaw-shared workclaw

sudo mkdir -p /srv/openclaw-shared/staging
sudo chown -R root:openclaw-shared /srv/openclaw-shared
sudo chmod -R 2770 /srv/openclaw-shared

# ACL for immediate + inherited group write
sudo setfacl -m g:openclaw-shared:rwx /srv/openclaw-shared/staging
sudo setfacl -d -m g:openclaw-shared:rwx /srv/openclaw-shared/staging
```

### Create explicit handoff folder

```bash
sudo mkdir -p /srv/openclaw-shared/staging/shared
sudo chown root:openclaw-shared /srv/openclaw-shared/staging/shared
sudo chmod 2770 /srv/openclaw-shared/staging/shared
sudo setfacl -m g:openclaw-shared:rwx /srv/openclaw-shared/staging/shared
sudo setfacl -d -m g:openclaw-shared:rwx /srv/openclaw-shared/staging/shared
```

### Smoke test

```bash
# as huatyou
echo "from-main $(date -Is)" > /srv/openclaw-shared/staging/shared/main-note.txt

# as workclaw
cat /srv/openclaw-shared/staging/shared/main-note.txt
echo "from-work $(date -Is)" >> /srv/openclaw-shared/staging/shared/main-note.txt
cat /srv/openclaw-shared/staging/shared/main-note.txt

# copy into each isolated workspace (manual merge/handoff)
cp /srv/openclaw-shared/staging/shared/main-note.txt /home/<user>/.openclaw/workspace/from-shared.txt
cp /srv/openclaw-shared/staging/shared/main-note.txt /home/<work-user>/.openclaw/workspace/from-shared.txt
```

### Critical runtime refresh check (important)

After adding users to `openclaw-shared`, **new login alone may not refresh already-running OpenClaw agent processes**.

Use this exact test from shell first:

```bash
id && ls -ld /srv/openclaw-shared /srv/openclaw-shared/staging /srv/openclaw-shared/staging/shared && touch /srv/openclaw-shared/staging/shared/.group-access-test && ls -l /srv/openclaw-shared/staging/shared/.group-access-test && rm /srv/openclaw-shared/staging/shared/.group-access-test
```

Expected:
- `id` shows `openclaw-shared` in groups
- touch/list/rm succeeds with no permission errors

If shell works but OpenClaw still gets `Permission denied`:
1. First try restarting the user gateway service:
   ```bash
   systemctl --user restart openclaw-gateway.service
   ```
2. Re-test from the agent by checking `id` and a real copy/write into `/srv/openclaw-shared/staging/shared`.
3. If the agent-side `id` still does not include `openclaw-shared`, restart the parent OpenClaw process/session manager (not just user login).
4. Only proceed once the agent-side `id` includes `openclaw-shared`.

Observed-good state from validation:
- both users can create files under `/srv/openclaw-shared/staging`
- file group is `openclaw-shared`
- `workclaw` hardening remains intact (`700` dirs, `600` secrets/json)
- agent-side access started working only after process context reflected the new group membership

---

## Common pitfalls we hit

- Running `openclaw gateway install` for another user from wrong context (`sudo -u ...`) without a proper user bus.
- Leaving stale workspace paths (`/home/<user>/.openclaw-work`) in migrated config/state.
- Keeping non-Codex fallback (`openai/gpt-5.3-codex`) without OpenAI API key.
- Token/env mismatch between runtime and CLI when using custom token var names.
- Forgetting to create `/srv/openclaw-shared/staging/shared` before handoff test (results in `No such file or directory`).
- Assuming user re-login is enough for long-running agent processes to pick up new group membership; verify agent-side `id` before concluding permissions are fixed.

---

## Bottom line

For this use case, dual Linux users is stable and practical:

- cleaner isolation than dual profiles under one user,
- easier mental model once service ownership is clean,
- straightforward security hardening at the end.
