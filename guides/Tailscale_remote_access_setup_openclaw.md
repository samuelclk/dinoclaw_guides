# OpenClaw Gateway over Tailscale Serve

A streamlined checklist for exposing the OpenClaw Control UI to your tailnet with HTTPS.

## 1. Prerequisites

- OpenClaw Gateway + openclaw CLI installed on the host.
- Tailscale CLI installed, signed in to the tailnet.
- Ability to run `sudo tailscale …` on the gateway box.

## 2. Enable MagicDNS (one-time per tailnet/host)

1. MagicDNS
   - Visit <https://login.tailscale.com/admin/dns>.
   - Toggle MagicDNS on (and keep “Override local DNS” enabled unless you manage DNS elsewhere).

Verify both are enabled:

```bash
# MagicDNS + HTTPS check
tailscale status --json | jq '{MagicDNSSuffix, MagicDNSEnabled: .CurrentTailnet.MagicDNSEnabled, CertDomains}'
```

> MagicDNS gives every node a stable `<hostname>.tailXXXX.ts.net` name AND lets Tailscale issue the matching TLS cert, so Serve can present HTTPS without you juggling 100.x IPs or certificates.

## 3. Clean slate (optional but recommended)

```bash
sudo tailscale serve reset
```

Removes previous Serve/Funnel mappings so you start fresh.

## 4. Start Tailscale Serve (HTTPS)

Run Serve yourself so it stays active in the background:

```bash
sudo tailscale serve --bg --https=18789 http://127.0.0.1:18789
```

This publishes `https://<magicdns-hostname>:18789/` on the tailnet while the Gateway keeps listening only on loopback.

## 5. Configure the Gateway

Edit `~/.openclaw/openclaw.json` (or use `openclaw config patch`) and set:

```json5
{
  gateway: {
    bind: "loopback",
    tailscale: { mode: "serve" },
    controlUi: {
      allowedOrigins: [
        "http://127.0.0.1:18789",
        "https://127.0.0.1:18789",
        "http://localhost:18789",
        "https://localhost:18789",
        "https://<magicdns-hostname>:18789"
      ]
    }
  }
}
```

Find your MagicDNS hostname with:

```bash
tailscale status --json | jq -r '.Self.DNSName'
```

Strip the trailing dot.

Save the file; the gateway will hot-reload automatically. (If it doesn’t, restart with `openclaw gateway restart`.)

> 💡 Hint: You can also paste the above JSON content to your Openclaw and ask it to make the edits to the `openclaw.json` file for you.

## 6. Verify Serve is active

```bash
openclaw status
# Look for: Tailscale: serve · <magicdns-hostname>

tailscale serve status --json
# Should show HTTPS true on port 18789, proxying to http://127.0.0.1:18789
```

## 7. Connect from another tailnet device

Open the Control UI at:

```text
https://<magicdns-hostname>:18789/
```

Because Serve terminates TLS, always use `https://`. The raw 100.x IP will throw a certificate warning.

## 8. Approve the browser once

Remote browsers must be paired the first time:

```bash
openclaw devices list
openclaw devices approve <requestId>
```

After approval, that browser profile stays authorized until you revoke it or wipe its storage.

## 9. Future maintenance

- New browsers/devices → repeat step 8.
- Need to revoke a device → `openclaw devices revoke --device <id> --role operator`.
- Want Serve to undo itself → `sudo tailscale serve reset` (then stop Serve or set `gateway.tailscale.mode` back to `off`).
