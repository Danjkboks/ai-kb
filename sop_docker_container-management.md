---
type: sop
domain: docker
source: session
status: active
tags: [docker, containers, n8n, langfuse, qdrant, llmlingua, management, windows]
created: 2026-05-19
updated: 2026-05-21
---

# SOP — Docker Container Management

**Purpose:** Start, stop, rebuild, and audit all Docker containers in the Workflow Lab stack.

---

## Stack Containers

| Container | Image | Port | Purpose |
|---|---|---|---|
| `n8n` | `n8nio/n8n:latest` | 5678 | Workflow orchestrator |
| `agent-llmlingua` | custom (Dockerfile) | 5001 | Flask LLMLingua compression + Langfuse tracing |
| `qdrant` | `qdrant/qdrant` | 6333 | Vector DB (2,061 workflows + 71 wiki chunks) |
| `langfuse` | `langfuse/langfuse:2.95.11` | 3000 | LLM observability |
| `langfuse-db` | `postgres:15` | internal | Langfuse database |

---

## Quick Start — All Containers

Use the environment launcher script (preferred):

```powershell
powershell -File "D:\aidirectory\scripts\env-launch.ps1"
```

Or with Cloudflare tunnel skipped (local-only work):
```powershell
powershell -File "D:\aidirectory\scripts\env-launch.ps1" -SkipTunnel
```

The script:
1. Auto-starts Docker Desktop if not running
2. Starts all 5 containers with health checks
3. Waits up to 90s for services to be healthy
4. Starts Cloudflare tunnel (unless -SkipTunnel)

---

## Manual Container Commands

### Start individual container
```powershell
docker start n8n
docker start agent-llmlingua
docker start qdrant
docker start langfuse
docker start langfuse-db
```

### Stop individual container
```powershell
docker stop n8n
```

### Check status
```powershell
docker ps                          # running containers
docker ps -a                       # all containers (including stopped)
```

### View logs
```powershell
docker logs n8n --tail 50
docker logs agent-llmlingua --tail 50 -f   # -f = follow live
```

---

## Rebuild agent-llmlingua (after code changes)

Build directory must be `D:\lab\workflow-lab\agents\` (Dockerfile location):

```powershell
cd D:\lab\workflow-lab\agents
docker build -t agent-llmlingua:latest .
docker stop agent-llmlingua
docker rm agent-llmlingua
docker run -d --name agent-llmlingua -p 5001:5001 --env-file .env agent-llmlingua:latest
```

⚠️ The container takes **2-3 minutes** to be ready after start — LLMLingua model downloads on first run from HuggingFace. Poll `/health` before testing:
```powershell
curl http://localhost:5001/health
```

---

## Langfuse (Docker Compose)

Langfuse uses a compose file with its Postgres DB:

```powershell
cd D:\lab\workflow-lab\agents
docker compose -f langfuse-compose.yml up -d       # start
docker compose -f langfuse-compose.yml down        # stop
docker compose -f langfuse-compose.yml logs -f     # logs
```

---

## Pre-build Audit Protocol

Before deploying new containers, always audit first:

```powershell
docker ps -a
```

Map each container: keep / stop / delete / rebuild. Confirm with user before deleting.

**Lessons learned:**
- The old `compress-api` container had wrong filename (`compress_api.py` vs `api.py`) and wrong port (5011 vs 5001) — always verify Dockerfile before building
- Deleting containers doesn't remove images — use `docker images` to check

---

## Qdrant Storage

Qdrant data is stored at `D:\lab\qdrant-storage\`. **Do not move this directory** — Docker volume mounts it. Moving it breaks the Qdrant container.

---

## Environment Variables

All secrets in `D:\aidirectory\.env`. The agent-llmlingua container reads `.env` at runtime via `--env-file`.

Key variables used by containers:
- `OPENROUTER_API_KEY` — LLM gateway
- `LANGFUSE_PUBLIC_KEY`, `LANGFUSE_SECRET_KEY`, `LANGFUSE_HOST`
- `NEXTAUTH_SECRET`, `SALT`, `POSTGRES_PASSWORD` — Langfuse auth

---

## Troubleshooting

| Problem | Fix |
|---|---|
| Container won't start | `docker logs {name}` to see error |
| Port already in use | `netstat -ano \| findstr :{port}` to find the process |
| agent-llmlingua not ready | Wait 2-3 min, poll `/health` endpoint |
| Qdrant data lost | Check `D:\lab\qdrant-storage\` exists, check volume mount in compose |
| Docker Desktop not starting | Manual launch: `C:\Program Files\Docker\Docker\Docker Desktop.exe` |
