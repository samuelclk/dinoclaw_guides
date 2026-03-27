# Retrospective: Main + Work OpenClaw Split to Two Linux Users

## Scope
We migrated from a single-user dual-profile setup to a dual-user setup:

- `huatyou` = main instance
- `workclaw` = work instance

Goal: isolate files, runtime, and OAuth accounts while keeping one host.

---

## What worked well

1. **Dual-user boundary is cleaner than dual-profile-only isolation**
   - Filesystem ownership became explicit.
   - Less accidental cross-profile state bleed.

2. **Port split is straightforward once service ownership is clean**
   - Main: `18789`
   - Work: `18790`

3. **OAuth separation was verifiable and stable**
   - Checking `auth-profiles.json` `accountId` per user worked reliably.

4. **Running install commands from target user session matters**
   - `openclaw gateway install --force` worked best when run directly as `workclaw`.

---

## Main failure modes we hit

1. **User bus/systemd context mismatch**
   - `sudo -u workclaw ... openclaw gateway install` failed with user-bus errors.
   - Fix: run from real `workclaw` shell or use `systemctl --machine=workclaw@.host --user`.

2. **Port ownership drift / stray listener**
   - Service looked active while CLI reported unreachable gateway.
   - Fix: explicit stop + kill stale process + start clean supervised service.

3. **State path drift after migration**
   - Errors still referenced `/home/<user>/.openclaw-work/...`.
   - Fix: replace stale paths under `/home/<work-user>/.openclaw` and re-check recursively.

4. **Model fallback auth mismatch**
   - `openai/gpt-5.3-codex` fallback required OpenAI API key when we only wanted Codex OAuth.
   - Fix: set fallbacks to `[]` (or use only providers with configured auth).

5. **CLI token env mismatch**
   - Gateway runtime and CLI checks diverged when token var names differed.
   - Fix: explicitly export token var in shell for diagnostics; normalize config for runtime.

---

## Key operating rules going forward

1. **Always execute work-instance operations as `workclaw`**
   - Avoid cross-user command wrappers for install/restart unless absolutely needed.

2. **Do not trust “service active” alone**
   - Always verify with:
     - `openclaw gateway status` (RPC probe), and
     - `ss -ltnp | grep <port>`

3. **After migration, scan for stale paths immediately**
   - `grep -R "/home/<user>/.openclaw-work" /home/<work-user>/.openclaw`

4. **Keep model config auth-consistent**
   - If using Codex OAuth only, don’t leave fallbacks requiring different provider credentials.

5. **Hardening comes last**
   - Migrate first, verify runtime, then apply strict chmod/chown/ACL lock-down.

---

## Final stable baseline

- Main (`huatyou`):
  - config: `/home/<user>/.openclaw/openclaw.json`
  - service: `openclaw-gateway.service`
  - port: `18789`

- Work (`workclaw`):
  - config: `/home/<work-user>/.openclaw/openclaw.json`
  - service: `openclaw-gateway.service` (in workclaw user manager)
  - port: `18790`

- OAuth:
  - verified separate `accountId` values per user profile

---

## What we would do first next time

1. Create target user first.
2. SSH directly into target user session.
3. Copy state and fix paths before any restarts.
4. Force-install service from target user shell.
5. Verify RPC + channel status before touching hardening.
6. Apply hardening as final step.

This sequence would have reduced most churn in this migration.
