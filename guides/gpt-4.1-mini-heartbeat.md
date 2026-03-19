# Using GPT-4.1 mini for heartbeat checks

This guide shows a practical, low-drama way to run OpenClaw heartbeats on `openai/gpt-4.1-mini`.

## When to use GPT-4.1 mini

`openai/gpt-4.1-mini` is a good heartbeat model when you want:

- lower cost than heavier flagship models
- enough capability for short periodic check-ins
- a model that can read a tiny checklist and either alert you or stay quiet

For heartbeat work, it is strong enough.

## Recommended setup

Use a separate heartbeat session so background checks do not share history with your normal direct messages.

Recommended heartbeat config:

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

## Why these settings

### `every: "30m"`
Use a normal production interval.

- `1m` is useful for testing only
- `30m` is a better default for real use
- lower frequency reduces noise and token burn

### `model: "openai/gpt-4.1-mini"`
This keeps heartbeat cheap and lightweight while still being capable enough for a short checklist.

### `session: "heartbeat-main"`
This keeps heartbeat on its own session instead of mixing with your normal DM session.

Benefits:

- easier debugging
- cleaner chat history
- less accidental session-state confusion

### `prompt`
Keep the configured prompt short:

```text
Check: Any blockers, opportunities, or progress updates needed?
```

The prompt should be a nudge, not a full instruction block.

### `lightContext: true`
This keeps heartbeat lean by avoiding unnecessary context.

## What to put in `HEARTBEAT.md`

Keep `HEARTBEAT.md` short and operational.

Recommended contents:

```md
# HEARTBEAT.md

- Check for blockers, opportunities, or progress updates worth sending.
- Keep it short.
- If nothing needs attention, reply exactly: HEARTBEAT_OK
```

This is the part you tweak over time. The config prompt can stay stable.

## How it should behave

A healthy heartbeat run should:

1. run on the heartbeat session, not your main DM session
2. use `gpt-4.1-mini`
3. read `HEARTBEAT.md`
4. either:
   - send a short useful alert, or
   - return `HEARTBEAT_OK`

## What not to do

### Don’t use debug-style session names in docs
For new-user documentation, use:

- `heartbeat-main`

Avoid names like:

- `heartbeat-main-v2`
- `heartbeat-ollama-v3`

Those are testing labels, not good long-term names.

### Don’t overload the prompt
Put checklist behavior in `HEARTBEAT.md`, not in the config prompt.

### Don’t keep heartbeat on a 1-minute interval
That is fine for testing, not for normal use.

## Troubleshooting

If heartbeat seems flaky, check these in order:

1. Is heartbeat still configured for `openai/gpt-4.1-mini`?
2. Is it using a separate session such as `heartbeat-main`?
3. Does the heartbeat transcript show actual runs on the expected model?
4. Are you seeing scheduler/session-store weirdness rather than model failure?

Important distinction:

- model problems mean the model itself is failing to do the task
- scheduler/session problems mean the model may be fine, but the run lifecycle is flaky

If one clean run already completed on `gpt-4.1-mini`, that is strong evidence the model is not too weak for heartbeat.

## Bottom line

For most users, the sane setup is:

- `openai/gpt-4.1-mini`
- `30m`
- separate session (`heartbeat-main`)
- tiny `HEARTBEAT.md`
- `lightContext: true`

That gives you useful heartbeat checks without paying flagship-model costs for a simple background task.
