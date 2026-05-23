---
type: sop
domain: infra
source: session
status: active
tags: [langfuse, observability, docker, openrouter, tracing, llm-monitoring, postgres]
created: 2026-05-19
updated: 2026-05-21
---

# SOP — Langfuse Setup & LLM Observability

**Purpose:** Deploy and configure Langfuse for LLM call tracing, cost monitoring, and debugging. Covers self-hosted Docker setup + OpenRouter Broadcast integration.

---

## What Langfuse Does

- Traces every LLM call: input, output, latency, cost
- Dashboard at `http://localhost:3000`
- Integrates with OpenRouter via **Broadcast** (zero-code change — intercepts all API calls automatically)
- Version: `2.95.11`

---

## Credentials

| Setting | Value |
|---|---|
| Admin email | `admin@local` |
| Admin password | `admin2026` (change in production) |
| Public key | `pk-lf-3d66cf4c-cee8-402c-8509-7f984c3b7e61` |
| Secret key | `sk-lf-24e35c3e-fbc2-463f-8a62-acfc089c8a21` |
| Host | `http://localhost:3000` |

⚠️ Keys are also in `D:\aidirectory\.env` as `LANGFUSE_PUBLIC_KEY` / `LANGFUSE_SECRET_KEY`.

---

## 1. Start Langfuse

```powershell
cd D:\lab\workflow-lab\agents
docker compose -f langfuse-compose.yml up -d
```

Wait ~30 seconds, then access: `http://localhost:3000`

---

## 2. Verify Health

```powershell
curl http://localhost:3000/api/health
```

Expected: `{"status":"ok"}`

---

## 3. Connect OpenRouter Broadcast (Zero-Code Integration)

OpenRouter **Broadcast** mode automatically forwards all API call metadata to Langfuse — no SDK changes needed in application code.

**In OpenRouter dashboard:**
1. Go to Settings → Integrations → Broadcast
2. Add Langfuse endpoint: `http://localhost:3000`
3. Enter Public Key + Secret Key from above
4. Save

All subsequent OpenRouter calls will appear as traces in Langfuse automatically.

---

## 4. SDK Integration (for api_with_langfuse.py)

The Flask API uses the Langfuse Python SDK for detailed span tracing:

```python
from langfuse import Langfuse
langfuse = Langfuse(
    public_key=os.getenv("LANGFUSE_PUBLIC_KEY"),
    secret_key=os.getenv("LANGFUSE_SECRET_KEY"),
    host=os.getenv("LANGFUSE_HOST", "http://localhost:3000")
)
```

Required env vars in `.env`:
```
LANGFUSE_PUBLIC_KEY=pk-lf-...
LANGFUSE_SECRET_KEY=sk-lf-...
LANGFUSE_HOST=http://localhost:3000
LANGFUSE_ENABLED=true
```

---

## 5. Public Access via Cloudflare Tunnel (Optional)

To expose Langfuse externally (manual, on-demand):

```powershell
cloudflared tunnel --url http://localhost:3000
```

Note the generated URL — it changes every restart. For permanent access, configure a named tunnel.

---

## Compose File

Location: `D:\lab\workflow-lab\agents\langfuse-compose.yml`

Key services:
- `langfuse-db` — Postgres 15 (internal, no external port)
- `langfuse` — App on port 3000, depends on langfuse-db

---

## Troubleshooting

| Problem | Fix |
|---|---|
| Port 3000 already in use | `netstat -ano \| findstr :3000`, kill the process |
| DB connection error | Ensure `langfuse-db` starts before `langfuse` (compose handles this) |
| Can't log in | DB may not be initialized yet — wait 60s, retry |
| Traces not appearing | Check OpenRouter Broadcast config, verify keys match |
| `LANGFUSE_ENABLED=true` but keys missing | Set `LANGFUSE_ENABLED=false` or add keys to `.env` |
