# Practical Guide: Run Main + Work OpenClaw on One Machine (Two Different ChatGPT OAuth Accounts)

## Goal
Run two OpenClaw instances on the same Linux user account, with clean isolation:

- **main** instance (default profile) on port `18789`
- **work** instance (`--profile work`) on port `18790`
- separate Telegram bot identities (optional but recommended)
- separate ChatGPT OAuth accounts for `openai-codex`

This guide is the streamlined method we validated.

---

## Prerequisites

### Required
- OpenClaw installed at:
  - `/home/huatyou/.npm-global/bin/openclaw`
- systemd user services available (`systemctl --user ...`)
- Two ChatGPT accounts you can OAuth-login separately

### Recommended
- Two Telegram bots (one for main, one for work)

---

## 10-minute quickstart

Use this if you already know what you’re doing and just need the working command flow.

```bash
# 1) shell aliases (put in ~/.bashrc, then source ~/.bashrc)
alias openclaw='/home/huatyou/.npm-global/bin/openclaw'
alias openclaww='OPENCLAW_GATEWAY_TOKEN_WORK=$(grep "^OPENCLAW_GATEWAY_TOKEN_WORK=" ~/.openclaw-work/.env | cut -d= -f2-) env -u OPENCLAW_GATEWAY_PORT -u OPENCLAW_SYSTEMD_UNIT -u OPENCLAW_CONFIG_PATH -u OPENCLAW_STATE_DIR /home/huatyou/.npm-global/bin/openclaw --profile work'

# 2) main/work token refs
openclaw config set gateway.auth.mode token
openclaw config set gateway.auth.token '{"source":"env","provider":"default","id":"OPENCLAW_GATEWAY_TOKEN_PLAY"}'
openclaww config set gateway.auth.mode token
openclaww config set gateway.auth.token '{"source":"env","provider":"default","id":"OPENCLAW_GATEWAY_TOKEN_WORK"}'

# 3) work isolation settings
openclaww config set gateway.port 18790
openclaww config set agents.defaults.workspace /home/huatyou/.openclaw-work/workspace
openclaww config set agents.defaults.model.primary openai-codex/gpt-5.4
openclaww config set agents.defaults.model.fallbacks '["openai/gpt-5.3-codex"]'

# 4) OAuth each profile with different ChatGPT account
openclaw models auth login --provider openai-codex
openclaww models auth login --provider openai-codex

# 5) restart services
openclaw gateway restart
openclaww gateway restart

# 6) verify
openclaw gateway status
openclaww gateway status
jq -r '.profiles["openai-codex:default"].accountId // "missing"' ~/.openclaw/agents/main/agent/auth-profiles.json
jq -r '.profiles["openai-codex:default"].accountId // "missing"' ~/.openclaw-work/agents/main/agent/auth-profiles.json
```

---

## Final layout (what you want)

- Main state dir: `~/.openclaw`
- Work state dir: `~/.openclaw-work`
- Main service: `openclaw-gateway.service` (port `18789`)
- Work service: `openclaw-gateway-work.service` (port `18790`)

---

## Step 1 — Set shell aliases

Edit `~/.bashrc` and make sure you have:

```bash
alias openclaw='/home/huatyou/.npm-global/bin/openclaw'
alias openclaww='OPENCLAW_GATEWAY_TOKEN_WORK=$(grep "^OPENCLAW_GATEWAY_TOKEN_WORK=" ~/.openclaw-work/.env | cut -d= -f2-) env -u OPENCLAW_GATEWAY_PORT -u OPENCLAW_SYSTEMD_UNIT -u OPENCLAW_CONFIG_PATH -u OPENCLAW_STATE_DIR /home/huatyou/.npm-global/bin/openclaw --profile work'
```

Apply changes:

```bash
source ~/.bashrc
```

**Why this matters:** `openclaww` must avoid inheriting main gateway env vars.

---

## Step 2 — Configure main `.env`

Edit:

```bash
nano ~/.openclaw/.env
```

Ensure it has a main token variable:

```text
OPENCLAW_GATEWAY_TOKEN_PLAY=<main_gateway_token>
TELEGRAM_BOT_TOKEN=<main_bot_token>
```

Save and exit:
- `CTRL+O`, `ENTER`, `CTRL+X`

---

## Step 3 — Configure work `.env`

Edit:

```bash
nano ~/.openclaw-work/.env
```

Ensure it has a work token variable:

```text
OPENCLAW_GATEWAY_TOKEN_WORK=<work_gateway_token>
TELEGRAM_BOT_TOKEN=<work_bot_token>
```

Save and exit:
- `CTRL+O`, `ENTER`, `CTRL+X`

---

## Step 4 — Wire gateway auth tokens via env SecretRef

### Main
```bash
openclaw config set gateway.auth.mode token
openclaw config set gateway.auth.token '{"source":"env","provider":"default","id":"OPENCLAW_GATEWAY_TOKEN_PLAY"}'
```

### Work
```bash
openclaww config set gateway.auth.mode token
openclaww config set gateway.auth.token '{"source":"env","provider":"default","id":"OPENCLAW_GATEWAY_TOKEN_WORK"}'
```

---

## Step 5 — Set work port + workspace

```bash
openclaww config set gateway.port 18790
openclaww config set agents.defaults.workspace /home/huatyou/.openclaw-work/workspace
```

(Leave main at default `18789` unless you intentionally changed it.)

---

## Step 6 — Pin work model to OpenAI Codex (avoid Anthropic auth errors)

```bash
openclaww config set agents.defaults.model.primary openai-codex/gpt-5.4
openclaww config set agents.defaults.model.fallbacks '["openai/gpt-5.3-codex"]'
```

---

## Step 7 — OAuth login each profile with different ChatGPT account

### Main login
```bash
openclaw models auth login --provider openai-codex
```

### Work login
```bash
openclaww models auth login --provider openai-codex
```

Important: in the browser flow, sign in to **different ChatGPT accounts**.

---

## Step 8 — Install/restart services

```bash
openclaw gateway install
openclaw gateway restart

openclaww gateway install
systemctl --user restart openclaw-gateway-work.service
```

---

## Verify

### 1) Gateway health
```bash
openclaw gateway status
openclaww gateway status
```

Expected:
- main: `RPC probe: ok`, port `18789`
- work: `RPC probe: ok`, port `18790`

### 2) Service mapping
```bash
systemctl --user --no-pager --type=service | grep openclaw-gateway
```

Expected:
- `openclaw-gateway.service` active
- `openclaw-gateway-work.service` active

### 3) OAuth accounts are different
```bash
jq -r '.profiles["openai-codex:default"].accountId // "missing"' ~/.openclaw/agents/main/agent/auth-profiles.json
jq -r '.profiles["openai-codex:default"].accountId // "missing"' ~/.openclaw-work/agents/main/agent/auth-profiles.json
```

Expected: two different `accountId` values.

---

## Troubleshooting

### Symptom: `unauthorized: gateway token mismatch`
Cause: token/env collision between main and work command context.

Fix:
1. Confirm token refs:
   ```bash
   openclaw config get gateway.auth.token
   openclaww config get gateway.auth.token
   ```
2. Confirm aliases loaded:
   ```bash
   source ~/.bashrc
   alias openclaww
   ```
3. Restart both:
   ```bash
   openclaw gateway restart
   openclaww gateway restart
   ```

---

## Important notes

- Do **not** reuse the same gateway token variable name for both instances when commands share shell environment.
- Keep secrets in `.env`; do not commit them to git.
- If status output behaves unexpectedly, run from a fresh shell (`exec bash`) and retry.

---

## Bottom line

This method gives stable side-by-side operation on one machine with:
- one main OpenClaw instance,
- one work OpenClaw instance,
- separate gateway/token context,
- separate ChatGPT OAuth account identities.
