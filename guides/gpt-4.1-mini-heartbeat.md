# Using GPT-4.1 mini for heartbeat checks

This guide is for people using the **ChatGPT OAuth** method for OpenAI models in OpenClaw.

It shows the simplest practical setup for running heartbeat checks on:

- `openai/gpt-4.1-mini`

This guide is written for normal operators, not for debugging sessions.

## What this changes

You will:

1. edit your OpenClaw config
2. set heartbeat to use `openai/gpt-4.1-mini`
3. put heartbeat on its own session name
4. make `HEARTBEAT.md` small and practical
5. restart the gateway
6. verify the config is live

## Recommended final settings

Use these heartbeat settings:

- `every`: `30m`
- `model`: `openai/gpt-4.1-mini`
- `session`: `heartbeat-main`
- `target`: `telegram`
- `prompt`: `Check: Any blockers, opportunities, or progress updates needed?`
- `lightContext`: `true`

Use this `HEARTBEAT.md` content:

```md
# HEARTBEAT.md

- Check for blockers, opportunities, or progress updates worth sending.
- Keep it short.
- If nothing needs attention, reply exactly: HEARTBEAT_OK
```

## Which file to edit

You need to edit:

- OpenClaw config: `/home/<user>/.openclaw/openclaw.json`
- Heartbeat checklist: `/home/<user>/.openclaw/workspace/HEARTBEAT.md`

## Step 1 — edit `openclaw.json`

Open a terminal and run:

```bash
cd /home/<user>/.openclaw
nano openclaw.json
```

Find the heartbeat block under your agent defaults and make it look like this:

```json
"heartbeat": {
  "every": "30m",
  "model": "openai/gpt-4.1-mini",
  "session": "heartbeat-main",
  "target": "telegram",
  "prompt": "Check: Any blockers, opportunities, or progress updates needed?",
  "lightContext": true
}
```

### How to save in nano

- Press `CTRL+O`
- Press `ENTER`
- Press `CTRL+X`

## Step 2 — edit `HEARTBEAT.md`

Run:

```bash
cd /home/<user>/.openclaw/workspace
nano HEARTBEAT.md
```

Replace the contents with:

```md
# HEARTBEAT.md

- Check for blockers, opportunities, or progress updates worth sending.
- Keep it short.
- If nothing needs attention, reply exactly: HEARTBEAT_OK
```

### Save and exit

- Press `CTRL+O`
- Press `ENTER`
- Press `CTRL+X`

## Step 3 — restart OpenClaw

Run:

```bash
systemctl --user restart openclaw-gateway
systemctl --user status openclaw-gateway --no-pager
```

You want to see the service come back as:

- `active (running)`

## Step 4 — verify the heartbeat config is live

Run:

```bash
cd /home/<user>/.openclaw/workspace
openclaw config get agents.defaults.heartbeat
```

You want to see:

```json
{
  "every": "30m",
  "model": "openai/gpt-4.1-mini",
  "session": "heartbeat-main",
  "target": "telegram",
  "prompt": "Check: Any blockers, opportunities, or progress updates needed?",
  "lightContext": true
}
```

## Optional: verify the heartbeat session later

If you want to confirm that heartbeat is using its own session instead of your main DM session, run:

```bash
cd /home/<user>/.openclaw/workspace
openclaw status
```

Or inspect the session store directly:

```bash
python3 - <<'PY'
import json
with open('/home/<user>/.openclaw/agents/main/sessions/sessions.json') as f:
    data=json.load(f)
for key in sorted(data):
    if 'heartbeat-main' in key or key.endswith(':main'):
        print(key)
PY
```

You want to see a separate heartbeat session key such as:

- `agent:main:heartbeat-main`

not just:

- `agent:main:main`

## Why this setup is the sane default

### Why `gpt-4.1-mini`

It is a good fit for heartbeat because it is:

- cheaper than heavier models
- strong enough for short periodic checks
- good enough to read a tiny checklist and either alert you or stay quiet

### Why `heartbeat-main`

Use a separate heartbeat session so background checks do not mix with your normal direct-message history.

This makes:

- debugging easier
- session behavior clearer
- heartbeat less likely to muddy normal chat state

### Why `30m`

Use `30m` for normal operation.

Do **not** use `1m` long-term unless you are actively testing.

## What not to do

### Do not use debug session names in a normal setup

Use:

- `heartbeat-main`

Do not use:

- `heartbeat-main-v2`
- `heartbeat-ollama-v3`

Those are debugging names, not good long-term config values.

### Do not overload the heartbeat prompt

Keep the configured prompt short.

Put the actual checklist in `HEARTBEAT.md`.

### Do not assume every heartbeat problem is a model problem

If one clean heartbeat already ran on `gpt-4.1-mini`, the model is probably fine.

Sometimes the flaky part is:

- scheduler behavior
- session bookkeeping
- transcript persistence

not the model itself.

## Troubleshooting

### Heartbeat does not seem to run

Check the live config:

```bash
cd /home/<user>/.openclaw/workspace
openclaw config get agents.defaults.heartbeat
```

### Gateway did not come back cleanly

Run:

```bash
systemctl --user status openclaw-gateway --no-pager
journalctl --user -u openclaw-gateway -n 100 --no-pager
```

### You want to test more aggressively

Temporarily change:

```json
"every": "1m"
```

Restart the gateway, verify one or two clean heartbeat runs, then put it back to:

```json
"every": "30m"
```

Do not leave it on `1m` in production unless you intentionally want that token burn.

## Bottom line

If you want the practical default, do exactly this:

1. set heartbeat model to `openai/gpt-4.1-mini`
2. set heartbeat session to `heartbeat-main`
3. keep the prompt short
4. keep `HEARTBEAT.md` tiny
5. run heartbeat every `30m`
6. restart the gateway
7. verify the live config

That is the simplest workable setup for GPT-4.1 mini heartbeat checks on ChatGPT OAuth.
