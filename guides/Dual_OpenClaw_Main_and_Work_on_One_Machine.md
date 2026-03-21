# Practical Guide: Main + Work OpenClaw on One Machine (Two Linux Users, Two ChatGPT OAuth Accounts)

## Goal
Run two isolated OpenClaw instances on one host:

- **main** instance under user `huatyou` on port `18789`
- **work** instance under user `lidoclaw` on port `18790`
- separate files, services, and OAuth accounts

This is the operator-safe method we validated.

---

## Architecture (final state)

- Main state: `/home/huatyou/.openclaw`
- Work state: `/home/lidoclaw/.openclaw`
- Main service: `openclaw-gateway.service` (user `huatyou`, port `18789`)
- Work service: `openclaw-gateway.service` (user `lidoclaw`, port `18790`)
- OAuth accounts differ by `accountId` in each user’s `auth-profiles.json`

---

## Prerequisites

### Required
- SSH/shell access as `huatyou`
- `sudo` access
- OpenClaw binary available (we used `/home/huatyou/.npm-global/bin/openclaw`)
- Two ChatGPT accounts for OAuth

### Recommended
- Separate Telegram bot token for work bot

---

## 10-minute quickstart (experienced operators)

```bash
# 0) backup existing work profile state
TS=$(date +%F-%H%M%S)
BK="/home/huatyou/migration-backups/openclaw-work-$TS"
mkdir -p "$BK"
cp -a /home/huatyou/.openclaw-work "$BK"/ 2>/dev/null || true

# 1) create/prepare work user
sudo adduser --disabled-password --gecos "" lidoclaw || true
sudo setfacl -m u:lidoclaw:x /home/huatyou

# 2) copy work state into lidoclaw home
sudo -u lidoclaw -H mkdir -p /home/lidoclaw/.openclaw
sudo rsync -a /home/huatyou/.openclaw-work/ /home/lidoclaw/.openclaw/
sudo chown -R lidoclaw:lidoclaw /home/lidoclaw/.openclaw

# 3) login as lidoclaw and set core config
sudo -iu lidoclaw
/home/huatyou/.npm-global/bin/openclaw config set agents.defaults.workspace /home/lidoclaw/.openclaw/workspace
/home/huatyou/.npm-global/bin/openclaw config set gateway.port 18790
/home/huatyou/.npm-global/bin/openclaw config set agents.defaults.model.primary openai-codex/gpt-5.4
/home/huatyou/.npm-global/bin/openclaw config set agents.defaults.model.fallbacks '[]'

# 4) ensure .env exists and load token var expected by CLI
[ -f ~/.openclaw/.env ] || touch ~/.openclaw/.env
set -a; source ~/.openclaw/.env; set +a
export OPENCLAW_GATEWAY_TOKEN="${OPENCLAW_GATEWAY_TOKEN_WORK:-$OPENCLAW_GATEWAY_TOKEN}"

# 5) install/reinstall service from lidoclaw session
/home/huatyou/.npm-global/bin/openclaw gateway install --force
systemctl --user daemon-reload
systemctl --user enable openclaw-gateway.service
systemctl --user restart openclaw-gateway.service

# 6) OAuth (work account)
/home/huatyou/.npm-global/bin/openclaw models auth login --provider openai-codex

# 7) verify
/home/huatyou/.npm-global/bin/openclaw gateway status
/home/huatyou/.npm-global/bin/openclaw channels status
```

---

## Step-by-step migration (careful path)

## Step 1 — Snapshot before changes (required)

```bash
TS=$(date +%F-%H%M%S)
BK="/home/huatyou/migration-backups/openclaw-work-$TS"
mkdir -p "$BK"
cp -a /home/huatyou/.openclaw-work "$BK"/ 2>/dev/null || true
cp -a /home/huatyou/.config/systemd/user/openclaw-gateway-work.service "$BK"/ 2>/dev/null || true
cp -a /home/huatyou/.bashrc "$BK"/bashrc.before
```

## Step 2 — Create `lidoclaw` user

```bash
sudo adduser --disabled-password --gecos "" lidoclaw
```

If you already created `openclawwork` and want rename:

```bash
sudo usermod -l lidoclaw openclawwork
sudo groupmod -n lidoclaw openclawwork
sudo usermod -d /home/lidoclaw -m lidoclaw
```

## Step 3 — Allow traversal to shared binary path

```bash
sudo setfacl -m u:lidoclaw:x /home/huatyou
```

## Step 4 — Copy work state into new user home

```bash
sudo -u lidoclaw -H mkdir -p /home/lidoclaw/.openclaw
sudo rsync -a /home/huatyou/.openclaw-work/ /home/lidoclaw/.openclaw/
sudo chown -R lidoclaw:lidoclaw /home/lidoclaw/.openclaw
```

## Step 5 — Login as `lidoclaw` and fix config

```bash
sudo -iu lidoclaw
/home/huatyou/.npm-global/bin/openclaw config set agents.defaults.workspace /home/lidoclaw/.openclaw/workspace
/home/huatyou/.npm-global/bin/openclaw config set gateway.port 18790
/home/huatyou/.npm-global/bin/openclaw config set agents.defaults.model.primary openai-codex/gpt-5.4
/home/huatyou/.npm-global/bin/openclaw config set agents.defaults.model.fallbacks '[]'
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

## Step 7 — Install service from lidoclaw session

```bash
set -a
source ~/.openclaw/.env
set +a
export OPENCLAW_GATEWAY_TOKEN="${OPENCLAW_GATEWAY_TOKEN_WORK:-$OPENCLAW_GATEWAY_TOKEN}"

/home/huatyou/.npm-global/bin/openclaw gateway install --force
systemctl --user daemon-reload
systemctl --user enable openclaw-gateway.service
systemctl --user restart openclaw-gateway.service
```

## Step 8 — OAuth for work account

```bash
/home/huatyou/.npm-global/bin/openclaw models auth login --provider openai-codex
```

Sign in with the **work** ChatGPT account.

## Step 9 — Verify service, channels, and account split

### As `lidoclaw`
```bash
/home/huatyou/.npm-global/bin/openclaw gateway status
/home/huatyou/.npm-global/bin/openclaw channels status
ss -ltnp | grep 18790
```

### Check account split
```bash
jq -r '.profiles["openai-codex:default"].accountId // "missing"' /home/huatyou/.openclaw/agents/main/agent/auth-profiles.json
jq -r '.profiles["openai-codex:default"].accountId // "missing"' /home/lidoclaw/.openclaw/agents/main/agent/auth-profiles.json
```

Expected: two different `accountId` values.

---

## Final permissions hardening (do this last)

After everything is working, lock it down:

```bash
# ownership
sudo chown -R lidoclaw:lidoclaw /home/lidoclaw

# directory permissions
sudo chmod 700 /home/lidoclaw
sudo chmod 700 /home/lidoclaw/.openclaw
sudo chmod 700 /home/lidoclaw/.ssh 2>/dev/null || true

# sensitive files
sudo chmod 600 /home/lidoclaw/.openclaw/.env 2>/dev/null || true
sudo find /home/lidoclaw/.openclaw -type f -name '*.json' -exec chmod 600 {} \;

# remove old ACLs you no longer need
sudo setfacl -x u:openclawwork /home/huatyou 2>/dev/null || true
```

Optional stricter boundary: remove unnecessary sudo rights from users that should not cross-read each other.

---

## Common pitfalls we hit

- Running `openclaw gateway install` for another user from wrong context (`sudo -u ...`) without a proper user bus.
- Leaving stale workspace paths (`/home/huatyou/.openclaw-work`) in migrated config/state.
- Keeping non-Codex fallback (`openai/gpt-5.3-codex`) without OpenAI API key.
- Token/env mismatch between runtime and CLI when using custom token var names.

---

## Bottom line

For this use case, dual Linux users is stable and practical:

- cleaner isolation than dual profiles under one user,
- easier mental model once service ownership is clean,
- straightforward security hardening at the end.
