---
type: sop
domain: n8n
source: session
status: active
tags: [n8n, webhook, cloudflare, docker, agent-bridge]
created: 2026-05-12
updated: 2026-05-21
---

# SOP: Agent-to-Local n8n Webhook Bridge

**Goal**: Securely expose local self-hosted n8n to external AI agents without router port-forwarding or exposing public IPs.

## 1. Local Infra (Docker)
- **Image**: `docker.n8n.io/n8nio/n8n:latest`
- **Compose layout**:
```yaml
version: '3.8'
services:
  n8n:
    image: docker.n8n.io/n8nio/n8n:latest
    restart: unless-stopped
    ports:
      - "<N8N_PORT>:<N8N_PORT>" # Default is 5678
    environment:
      - GENERIC_TIMEZONE=<YOUR_TIMEZONE>
    volumes:
      - n8n_data:/home/node/.n8n
volumes:
  n8n_data:
```

## 2. Cloudflare Tunnel (Network)
- **Install (Windows)**: `winget install Cloudflare.cloudflared`
- **Install (Linux/PikaOS)**: Download binary from Cloudflare repo or use apt.
- **Run Command**: `cloudflared tunnel --url http://localhost:<N8N_PORT>` 
- *Troubleshooting (Windows Loopback Block)*: If localhost fails, route to the host's LAN IP:
  `cloudflared tunnel --url http://<YOUR_LAN_IP>:<N8N_PORT>`

## 3. n8n Security & Node Config
- **Node Type**: Webhook
- **Method**: GET (trigger) or POST (payloads)
- **Authentication**: Header Auth -> Create New.
- **Credentials**: 
  - Name: `<CUSTOM_HEADER_NAME>` (e.g., x-api-key)
  - Value: `<YOUR_SECRET_KEY>`
- **Listen Mode**: Switch from *Test URL* (`/webhook-test/`) to *Production URL* (`/webhook/`).

## 4. Activation
- Click **Publish** (or set to Active) in n8n.
- Test webhooks are one-time use; Production webhooks listen continuously in the background.

## 5. Agent Trigger (Perplexity Bypassing)
AI Agents (like Perplexity) often use headless browsers for `fetch_url`, which Cloudflare blocks (403/502).
**Bypass method**: The Agent must use raw Python `urllib` to hit the endpoint directly.

**Agent Python Template**:
```python
import urllib.request
import json

url = "<YOUR_TUNNEL_URL>/webhook/<WEBHOOK_PATH>"

req = urllib.request.Request(
    url, 
    headers={
        'User-Agent': 'Custom-Agent/1.0',
        '<CUSTOM_HEADER_NAME>': '<YOUR_SECRET_KEY>'
    }
)

try:
    with urllib.request.urlopen(req, timeout=10) as response:
        print(f"Status: {response.status}")
        print(f"Body: {response.read().decode('utf-8')}")
except Exception as e:
    print(f"Error: {e}")
```
