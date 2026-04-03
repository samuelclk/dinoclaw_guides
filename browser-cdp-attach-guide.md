# OpenClaw Browser Access via Attached CDP Endpoint

This guide is for:
- new OpenClaw users who want browser control without using their main daily Chrome profile directly
- OpenClaw agents that need a reliable, documented path to attach to an already-running Chromium browser via CDP

It describes the clean supported pattern:
- run a dedicated Chromium-based browser with a real DevTools endpoint
- point an OpenClaw browser profile at that endpoint with `cdpUrl`
- use `attachOnly: true` so OpenClaw does not try to launch or manage that browser itself

---

## What this setup is for

Use this when:
- you want OpenClaw to control a browser that is already running
- you want persistent login state for sites like X or LinkedIn
- you want a dedicated automation browser/profile separate from your normal Chrome
- `existing-session` is not the path you want

Do **not** use this guide if:
- you want OpenClaw to manage its own clean browser profile automatically
- you specifically want Chrome MCP `existing-session` attach to your real signed-in Chrome

---

## Supported model

OpenClaw supports a sane CDP attach path when the browser already exposes a normal Chromium DevTools endpoint.

That means the target browser must serve a working endpoint like:
- `http://127.0.0.1:9333/json/version`
- `http://127.0.0.1:9333/json/list`

If those do not work, OpenClaw does not have a clean CDP target yet.

---

## Recommended pattern

Use a **dedicated Chrome/Chromium profile** just for OpenClaw browser work.

Benefits:
- login state persists across runs
- avoids touching your main browser session
- avoids weird conflicts with an existing port already used by another process
- easier to debug because the browser is clearly separate

Recommended example:
- browser app: Google Chrome
- profile dir: `~/.openclaw/chrome-debug-profile`
- CDP port: `9333`
- CDP URL: `http://127.0.0.1:9333`

Avoid reusing `9222` blindly. In real environments it is often already occupied by something weird or non-standard.

---

## OpenClaw config

Edit `~/.openclaw/openclaw.json` and add a browser profile like this:

```json
{
  "browser": {
    "enabled": true,
    "defaultProfile": "chrome-debug",
    "profiles": {
      "chrome-debug": {
        "cdpUrl": "http://127.0.0.1:9333",
        "attachOnly": true,
        "color": "#7B61FF"
      }
    }
  }
}
```

Important:
- `cdpUrl` must point to the actual reachable DevTools HTTP endpoint
- `attachOnly: true` means OpenClaw will attach only; it will not try to launch the browser
- for remote CDP profiles, use `cdpUrl`; do **not** mix this with `driver: "existing-session"`

---

## Launch the dedicated browser

### macOS example

This launch pattern was verified to work cleanly:

```bash
open -n -a "Google Chrome" --args \
  --user-data-dir=/Users/sam/.openclaw/chrome-debug-profile \
  --remote-debugging-address=127.0.0.1 \
  --remote-debugging-port=9333 \
  --no-first-run \
  --no-default-browser-check \
  --new-window about:blank
```

What each flag does:
- `open -n -a "Google Chrome"` launches a new app instance on macOS
- `--user-data-dir=...` creates a separate browser profile directory
- `--remote-debugging-address=127.0.0.1` binds CDP to loopback only
- `--remote-debugging-port=9333` exposes the DevTools endpoint on a clean port
- `--new-window about:blank` opens a visible window immediately

### Why `open -n -a` is preferred on macOS

Launching the Chrome binary directly can sometimes print `DevTools listening ...` and then die.

Using the macOS-native launcher:
- is more stable
- keeps the browser alive properly
- is better for a visible interactive automation profile

---

## Verify the browser before touching OpenClaw

Do this first.

### Check the listener

```bash
lsof -nP -iTCP:9333 -sTCP:LISTEN
```

You want to see Chrome listening on `127.0.0.1:9333`.

### Check DevTools discovery

```bash
curl http://127.0.0.1:9333/json/version
curl http://127.0.0.1:9333/json/list
```

Success looks like:
- `/json/version` returns JSON with a `webSocketDebuggerUrl`
- `/json/list` returns JSON listing tabs/pages

Example good `/json/version` shape:

```json
{
  "Browser": "Chrome/146.0.7680.178",
  "Protocol-Version": "1.3",
  "webSocketDebuggerUrl": "ws://127.0.0.1:9333/devtools/browser/..."
}
```

If these endpoints do not work, stop there and fix the browser first.

---

## Restart OpenClaw after config changes

After editing `openclaw.json`, restart the gateway:

```bash
openclaw gateway restart
openclaw gateway status
```

Then verify the gateway is healthy before testing browser actions.

---

## Log into sites inside the dedicated profile

If you need X, LinkedIn, or other authenticated sites:
1. launch the dedicated browser profile
2. log in manually in that dedicated browser window
3. keep using the same `user-data-dir` for future runs

Do **not** log in using your normal Chrome and expect the dedicated CDP profile to inherit those sessions.

The login must happen inside the dedicated browser launched with the dedicated `--user-data-dir`.

---

## Operational checks for agents

Before claiming browser attach works, verify all of these:

1. OpenClaw config has a named profile with:
   - `cdpUrl`
   - `attachOnly: true`
2. Browser is running and listening on the expected port
3. `/json/version` works
4. `/json/list` works
5. OpenClaw gateway is running after the config change
6. OpenClaw browser tool or CLI can see the profile/tabs

If any of 2-4 fail, this is a browser/CDP problem, not an OpenClaw profile problem.

---

## Common failure modes

### 1. `connection refused` on `/json/version`

Meaning:
- nothing is listening at that address/port
- browser died after launch
- wrong address family or wrong port

Fix:
- verify the process is alive
- verify the exact bind address
- relaunch the browser cleanly

### 2. Port is listening, but `/json/version` returns 404

Meaning:
- something is on that port, but it is not a normal Chromium DevTools HTTP endpoint
- this often happens when reusing a polluted port like `9222`

Fix:
- stop fighting that port
- launch a dedicated browser on a clean port like `9333`

### 3. Browser log says `DevTools listening ...` but the port is dead a moment later

Meaning:
- browser started, then exited

Fix:
- on macOS, prefer `open -n -a "Google Chrome" --args ...`
- use a separate `user-data-dir`
- verify the process remains alive with `ps` + `lsof`

### 4. IPv6 vs IPv4 confusion

If the browser binds to `[::1]` but you probe `127.0.0.1`, or vice versa, it will look broken.

Fix:
- probe the exact address family the browser actually bound
- if you want predictable behavior, bind explicitly to `127.0.0.1`

### 5. OpenClaw browser tool times out after the endpoint is already healthy

Meaning:
- stale gateway/browser control state
- gateway restart churn
- transient browser-control timeout rather than bad CDP

Fix:
- confirm `/json/version` and `/json/list` still work
- restart the gateway cleanly
- retry after the gateway is healthy

### 6. No visible movement in the browser window

Meaning:
- attach/focus operations may be happening without obvious animation
- some tool actions do not visibly change the page much

Fix:
- open visible search tabs or navigate pages to make activity obvious
- verify through tabs/snapshots, not just visual motion

---

## Recommended troubleshooting sequence

When debugging, use this order:

1. **Browser process**
   - is Chrome/Chromium actually running?
2. **Socket listener**
   - is the CDP port listening?
3. **DevTools discovery**
   - do `/json/version` and `/json/list` work?
4. **OpenClaw config**
   - does the named profile use the exact live `cdpUrl`?
5. **Gateway**
   - did OpenClaw reload the updated config?
6. **Browser tool / CLI verification**
   - can OpenClaw list tabs or snapshot a page?

Do not skip straight to blaming OpenClaw if step 3 fails.

---

## Best-practice recommendation for new users

If you want a stable setup quickly:

1. pick a fresh port like `9333`
2. create a dedicated profile directory like `~/.openclaw/chrome-debug-profile`
3. launch Chrome with that profile and remote debugging enabled
4. verify `/json/version`
5. point OpenClaw `browser.profiles.chrome-debug.cdpUrl` at it
6. set `attachOnly: true`
7. restart the gateway
8. log into your target sites in that dedicated browser

This is the simplest reliable attach-only CDP pattern.

---

## Example agent-ready summary

If an agent needs the short version, use this checklist:

- Launch a dedicated Chromium browser with:
  - separate `--user-data-dir`
  - `--remote-debugging-address=127.0.0.1`
  - `--remote-debugging-port=9333`
- Confirm:
  - `lsof -nP -iTCP:9333 -sTCP:LISTEN`
  - `curl http://127.0.0.1:9333/json/version`
  - `curl http://127.0.0.1:9333/json/list`
- Configure OpenClaw:
  - `browser.profiles.chrome-debug.cdpUrl = "http://127.0.0.1:9333"`
  - `browser.profiles.chrome-debug.attachOnly = true`
- Restart gateway
- Use `profile="chrome-debug"`
- If the browser dies, relaunch the dedicated profile instead of changing OpenClaw config blindly

---

## Known-good macOS command

```bash
open -n -a "Google Chrome" --args \
  --user-data-dir=/Users/sam/.openclaw/chrome-debug-profile \
  --remote-debugging-address=127.0.0.1 \
  --remote-debugging-port=9333 \
  --no-first-run \
  --no-default-browser-check \
  --new-window about:blank
```

---

## Final rule

A CDP profile is only real when the browser itself is real.

If `/json/version` is broken, fix the browser.
If `/json/version` works, then fix OpenClaw attach/config.
Do not mix those two layers.
