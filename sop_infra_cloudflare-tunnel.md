---
type: sop
domain: infra
source: session
status: active
tags: [cloudflare, tunnel, n8n, public-url, networking, trycloudflare]
created: 2026-05-12
updated: 2026-05-21
---

# SOP — Cloudflare Tunnel (Quick Tunnel)

**Purpose:** Expose local n8n (or any service) to the internet without port-forwarding. Required for n8n webhooks to receive external calls.

---

## Key Facts

- **URL changes every restart** — always note the new URL after starting
- Uses `trycloudflare.com` subdomains (free, no account needed for quick tunnels)
- Handles TLS automatically
- n8n is the primary use case (port 5678), Langfuse on-demand (port 3000)

---

## Start Tunnel for n8n

```powershell
cloudflared tunnel --url http://localhost:5678
```

Wait for output containing:
```
Your quick Tunnel has been created! Visit it at:
https://XXXX-XXXX-XXXX.trycloudflare.com
```

📋 **Copy that URL immediately.** It's needed for:
- n8n webhook triggers from external services
- Sharing n8n UI access
- AI agent calls to n8n workflows

---

## Start Tunnel for Langfuse (On-Demand)

```powershell
cloudflared tunnel --url http://localhost:3000
```

Use when you need external access to Langfuse dashboard. Not required for normal operation.

---

## Automated via env-launch.ps1

The environment launcher starts the tunnel automatically:

```powershell
powershell -File "D:\aidirectory\scripts\env-launch.ps1"
```

To skip tunnel (local-only work):
```powershell
powershell -File "D:\aidirectory\scripts\env-launch.ps1" -SkipTunnel
```

---

## If localhost Doesn't Work

Windows loopback restriction can block `localhost`. Use the machine's local IP instead:

```powershell
# Find local IP
ipconfig | findstr "IPv4"

# Use it in the tunnel command
cloudflared tunnel --url http://192.168.X.X:5678
```

---

## Named Tunnel (Persistent URL — Future)

For a permanent URL that doesn't change on restart, set up a named Cloudflare tunnel:

1. Create account at `dash.cloudflare.com`
2. Add a domain or use a free subdomain
3. `cloudflared tunnel create workflow-lab`
4. Configure route and run: `cloudflared tunnel run workflow-lab`

Not currently configured — quick tunnels cover current needs.

---

## Troubleshooting

| Problem | Fix |
|---|---|
| `connection refused` | Service not running on that port — start the container first |
| Tunnel starts but URL unreachable | Wait 30s for DNS propagation |
| `localhost` fails | Use machine local IP (see above) |
| Tunnel dies mid-session | Restart the command in a new terminal |
| env-launch.ps1 shows tunnel `failed` | Run manually in separate terminal |

---

## Notes

- Running in foreground: leave the terminal open while tunnel is needed
- Running in background (PowerShell): `Start-Job { cloudflared tunnel --url http://localhost:5678 }`
- The cloudflared binary is installed system-wide and in PATH
