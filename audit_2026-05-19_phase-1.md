---
type: audit
domain: stack
source: session
status: active
tags: [audit, phase-1, workflow-indexation, n8n]
created: 2026-05-19
updated: 2026-05-19
---

# Phase 1 Audit Report — Recon & Inventory

**Date:** 2026-05-20
**Auditor:** Claude (Opus 4.7)
**Scope:** Workflow Lab repo, 5 custom containers, Dify stack, memory system, env-launch skill, CLAUDE.md, Obsidian vault + GitHub sync, .claude/ routing
**Approach:** Explore subagent + targeted verification. No fixes applied.

---

## 0. Headline finding (read this first)

**The project lives in TWO root directories**, and almost no documentation mentions this:

| Path | What's there | Mentioned in CLAUDE.md / docs? |
|---|---|---|
| `D:\workflow-lab\` | Workflow indexation scripts, agent routing MANIFESTs, docs, index/, .claude/ | YES — referenced everywhere |
| `D:\lab\workflow-lab\` | Actual running Python services, Dockerfile, live `.env`, langfuse-compose, memory_save.py | NO — only one memory file references it |

Every other section in this report is interpreted against this split. **This is the most consequential thing to clarify in Phase 2.**

---

## 1. Project topology (verified)

```
D:\workflow-lab\                          (documentation + indexation)
├── README.md, SETUP.md, DEPLOYMENT_REPORT.md, PHASE_COMPLETION_REPORT.md, FILE_GUIDE.txt
├── .claude\                              (routing.json, memory-agent/, audit logs, session export utils)
├── .sessions-archive\                    (2 archived sessions, May 18-19)
├── agents\                               (7 MANIFEST.md routing folders — NO running code here)
│   ├── docker-ops\                       (MANIFEST + build.ps1 + .env.example)
│   ├── git-ops\                          (MANIFEST only)
│   ├── infra-ops\                        (MANIFEST + check-status.ps1 + .env.example)
│   ├── memory-architect\                 (MANIFEST + check-triggers.ps1 + ingest.ps1 + project-context.json)
│   ├── n8n-workflow\                     (MANIFEST + create-workflow.py + .env.example)
│   ├── python-ops\                       (MANIFEST + run-compression.ps1 + .env.example)
│   └── research-agent\                   (MANIFEST + project-context.json + queue.json)
├── audit\                                (this report)
├── docs\                                 (only ARCHITECTURE.md — see §5)
├── index\                                (5 JSON indices, 4.4 MB total)
└── scripts\                              (extract_workflows.py, search.py, migrate.py, env-launch.ps1)

D:\lab\workflow-lab\                      (RUNNING SERVICES — not documented in CLAUDE.md)
├── .env.n8n                              (344 B, n8n token — live secrets)
├── .claude\
│   ├── memory_save.py                    (3.9 KB, snapshot updater referenced by MEMORY.md)
│   └── memory_save.bat
└── agents\
    ├── .env                              (351 B, LIVE secrets — not committed presumably)
    ├── Dockerfile                        (258 B, builds agent-llmlingua image)
    ├── requirements.txt                  (77 B)
    ├── api.py                            (4.9 KB, older Flask API)
    ├── api_with_langfuse.py              (8.5 KB, newer Flask API w/ dual tracing)
    ├── ask.py                            (1.8 KB, May 8 — wiki Q&A agent)
    ├── compress_api.py                   (1.5 KB, May 13 — source of stale compress-api image?)
    ├── langfuse-compose.yml              (1.7 KB — defines langfuse + langfuse-db services)
    ├── n8n_mcp_bridge.py                 (1.8 KB)
    ├── qdrant_embeddings.py              (6.7 KB)
    └── qdrant_wiki_embeddings.py         (5.3 KB)

D:\lab\Claudi\                            (Claude's primary working directory)
└── CLAUDE.md                             (Project instructions — references D:\workflow-lab\ only)
```

---

## 2. Running services (verified via `docker ps`)

### Workflow Lab stack (5 containers)
| Container | Image | Port | Source location |
|---|---|---|---|
| n8n | `docker.n8n.io/n8nio/n8n:latest` | 5678 | Public image |
| agent-llmlingua | `agent-llmlingua:latest` (custom, 8.34 GB) | 5001 | Built from `D:\lab\workflow-lab\agents\Dockerfile` |
| qdrant | `qdrant/qdrant:latest` | 6333 | Public image |
| langfuse | `ghcr.io/langfuse/langfuse:2` | 3000 | Public image, defined in `D:\lab\workflow-lab\agents\langfuse-compose.yml` |
| langfuse-db | `postgres:16-alpine` | — | Defined in same compose file |

### Dify stack (10 containers — separate project)
| Container | Image |
|---|---|
| docker-nginx-1 | nginx:latest |
| docker-api-1, docker-worker-1, docker-worker_beat-1 | langgenius/dify-api:1.14.0 |
| docker-web-1 | langgenius/dify-web:1.14.0 |
| docker-sandbox-1 | langgenius/dify-sandbox:0.2.15 |
| docker-plugin_daemon-1 | langgenius/dify-plugin-daemon:0.6.0-local |
| docker-db_postgres-1 | postgres:15-alpine |
| docker-redis-1 | redis:6-alpine |
| docker-ssrf_proxy-1 | ubuntu/squid:latest |

Defined in `D:\dify\dify\docker\docker-compose.yaml`. Shares the Docker daemon but no integration with the workflow lab is documented.

### Cloudflare tunnel
- Configured as named tunnel `workflow-lab`.
- Currently broken: missing `cert.pem` (needs `cloudflared tunnel login`).
- Last working session: per `project_progress.md`, mentioned as "active" on 2026-05-19.

---

## 3. Custom Docker images (verified)

| Image | Size | Status | Notes |
|---|---|---|---|
| `agent-llmlingua:latest` | **8.34 GB** | Running | Built from `D:\lab\workflow-lab\agents\Dockerfile` |
| `compress-api:latest` | 213 MB | Created 2026-05-12, **no container uses it** | Likely orphaned — `compress_api.py` is its probable source |

---

## 4. Configuration & secrets (paths only, no content read)

### Live secrets (NOT in `D:\workflow-lab\`)
- `D:\lab\workflow-lab\.env.n8n` (344 B)
- `D:\lab\workflow-lab\agents\.env` (351 B)

### Template-only (`.env.example`, no real secrets)
- `D:\workflow-lab\agents\docker-ops\.env.example`
- `D:\workflow-lab\agents\infra-ops\.env.example`
- `D:\workflow-lab\agents\n8n-workflow\.env.example`
- `D:\workflow-lab\agents\python-ops\.env.example`

**Observation (not judgment):** the 4 `.env.example` templates in `D:\workflow-lab\agents\*\` are paired with MANIFEST routing folders but the actual services run from `D:\lab\workflow-lab\agents\.env` — a single file.

---

## 5. Documentation inventory

### Files actually present in `D:\workflow-lab\`
- `README.md` (project navigation hub)
- `SETUP.md` (5-minute quick start for indexation)
- `DEPLOYMENT_REPORT.md` (Phase 1-4 indexation deployment)
- `PHASE_COMPLETION_REPORT.md` (Phase 1-4 completion summary)
- `FILE_GUIDE.txt` (file organization reference)
- `docs\ARCHITECTURE.md` (indexation system architecture)

### Files referenced by README.md but NOT FOUND
- `docs\PHASE_COMPLETION_REPORT.md` (actually at root, not under `docs\`)
- `docs\IMPLEMENTATION_SUMMARY.md` (**missing**)
- `docs\WORKFLOW_ANALYSIS.md` (**missing**)
- `docs\WORKFLOW_TAXONOMY.md` (**missing**)
- `docs\WORKFLOW_IMPLEMENTATION_GUIDE.md` (**missing**)

### Other docs
- `D:\lab\Claudi\CLAUDE.md` (project root instructions — last reviewed 2026-05-19)

---

## 6. Memory system inventory

Location: `C:\Users\GnReN-PC\.claude\projects\D--lab-Claudi\memory\`

| File | Type | Last modified | Purpose |
|---|---|---|---|
| `MEMORY.md` | index | May 20 | Index of all memory files |
| `SESSION_SNAPSHOT.md` | project | May 20 13:26 | Live state (auto-written by env-launch.ps1) |
| `project_progress.md` | project | May 19 | Completed milestones, what's running |
| `project_stack_status.md` | project | May 19 | Live components summary |
| `workflow_indexation_system.md` | project | May 19 | Indexation system spec |
| `n8n_mcp_setup.md` | project | May 18 | n8n MCP server config |
| `feedback_quality_over_savings.md` | feedback | May 19 | Token efficiency principles |
| `feedback_read_memory_first.md` | feedback | May 20 | Lesson from 88K-token session |
| `reference_agent_security.md` | reference | May 19 | Agent security framework |
| `reference_model_selection.md` | reference | May 19 | Model selection framework |

External references the memory points to (not verified):
- `C:\Users\GnReN-PC\.claude\shared-knowledge\agent-security-framework.md`
- `C:\Users\GnReN-PC\.claude\shared-knowledge\model-selection-framework.md`

---

## 7. Skills inventory

Location: `C:\Users\GnReN-PC\AppData\Local\Packages\Claude_pzs8sxrjxfjjc\LocalCache\Roaming\Claude\local-agent-mode-sessions\skills-plugin\<id>\skills\`

| Skill | Custom? | Purpose |
|---|---|---|
| **env-launch** | YES (we built it) | Boot the workflow lab stack |
| consolidate-memory | bundled (anthropic) | Memory housekeeping |
| docx, pdf, pptx, xlsx | bundled (anthropic) | File format handlers |
| schedule | bundled (anthropic) | Scheduled tasks |
| setup-cowork | bundled (anthropic) | Cowork setup |
| skill-creator | bundled (anthropic) | Build new skills |

---

## 8. Agent routing system (`D:\workflow-lab\.claude\`)

Files present:
- `routing.json` — version 1.0, routes task types (docker, n8n, python, git, infrastructure, memory) to agent folders. Per-agent max context: 1000–2500 tokens. Spawn strategy: `isolated`. Fallback: `general`.
- `memory-agent\MANIFEST.md` — manages CLAUDE.md routing & memory index
- `audit-log-init.ps1` — audit logging setup script
- `audit-memory-architect.log` — actual audit log file
- `export-sessions.ps1` — session export utility
- `setup-windows-task.ps1` — Windows Task Scheduler setup
- `IMPLEMENTATION-READY.md`, `MEMORY-ARCHITECT-SETUP.md`, `SCHEDULING-SETUP-C.md` — internal docs

**Not yet inspected** (for Phase 2/3): the actual routing logic, whether the Windows Scheduled Task is installed, whether the audit log shows recent activity.

---

## 9. Obsidian vault + GitHub sync (verified)

- **Path:** `D:\Wproject\Notes\workflow-lab\`
- **Has `.obsidian\`** (verified vault)
- **Has `.git\`** (verified repo — auto-sync target)
- Top-level structure: `agents/`, `connectors/`, `docs/`, `experiments/`, `specs/`, `wiki/`, `workflows/`, plus `README.md`, `TestNote.md`

GitHub remote URL: NOT inspected (would need to read .git/config). Sync mechanism (cron? Obsidian Git plugin? auto-commit hook?): NOT inspected — `project_progress.md` says "auto-commit every 10 min" but the mechanism wasn't verified.

---

## 10. Other docker-compose files found (potentially abandoned)

| Path | Purpose | Status |
|---|---|---|
| `D:\dify\dify\docker\docker-compose.yaml` | Dify stack | **In use** (10 containers running) |
| `D:\dify\dify\docker\docker-compose.middleware.yaml` | Dify variant | Unknown |
| `D:\n8n-workflows\docker-compose.yml` | n8n base | Unknown — n8n is running from a different image |
| `D:\n8n-workflows\docker-compose.dev.yml` | n8n dev variant | Unknown |
| `D:\n8n-workflows\docker-compose.prod.yml` | n8n prod variant | Unknown |
| `D:\VMs\Dockers\N8N\docker-compose.yml` | n8n alt | Unknown |

The n8n container that env-launch starts is the `docker.n8n.io/n8nio/n8n:latest` standalone image, not from any of these compose files. **Four n8n compose definitions exist outside the project — relationship unclear.**

---

## 11. Python source files (in `D:\lab\workflow-lab\agents\`)

| File | Size | Modified | Purpose (from filename + memory) |
|---|---|---|---|
| `api.py` | 4.9 KB | May 19 21:55 | Older Flask API (LLMLingua + caching) |
| `api_with_langfuse.py` | 8.5 KB | May 19 22:16 | Newer — adds Langfuse dual tracing |
| `ask.py` | 1.8 KB | May 8 | Wiki Q&A agent — oldest file in folder |
| `compress_api.py` | 1.5 KB | May 13 | Likely source of orphan `compress-api` image |
| `n8n_mcp_bridge.py` | 1.8 KB | May 18 | Bridge between n8n and Claude via MCP |
| `qdrant_embeddings.py` | 6.7 KB | May 19 | Indexes 2,061 n8n workflows into Qdrant |
| `qdrant_wiki_embeddings.py` | 5.3 KB | May 19 | Indexes wiki (71 chunks) into Qdrant |

---

## 12. Items flagged for Phase 2 (Health & Hygiene)

These are observations, not judgments. Each will be evaluated in Phase 2.

1. **Two project roots** (`D:\workflow-lab\` and `D:\lab\workflow-lab\`) with no documented relationship.
2. **CLAUDE.md path inconsistencies**: says `D:\Claudi\` but actual location is `D:\lab\Claudi\`; never mentions `D:\lab\workflow-lab\`.
3. **5 docs referenced by `README.md` are missing** from `docs/`.
4. **`compress-api:latest` Docker image (213 MB)** orphaned — no container uses it.
5. **`agent-llmlingua:latest` is 8.34 GB** — investigate why; LLMLingua models can be large but worth profiling.
6. **Two API files** (`api.py` and `api_with_langfuse.py`) — per memory, the dual-tracing version is current, so `api.py` may be stale.
7. **Two old session archives** in `.sessions-archive\` — purpose unclear.
8. **`langfuse-compose.yml` exists but `env-launch.ps1` uses `docker start` directly** — compose file isn't invoked. May be out of sync.
9. **`.env.n8n` and `agents\.env`** are the only live secrets, both in `D:\lab\workflow-lab\` — no backup or rotation evidence.
10. **Four unrelated n8n docker-compose files** in other folders (`D:\n8n-workflows\`, `D:\VMs\Dockers\N8N\`) — relationship to the running n8n container unknown.
11. **Cloudflare tunnel is broken** (cert missing) — last successful run on or before 2026-05-19.
12. **Audit log file** `audit-memory-architect.log` exists but freshness not yet checked.
13. **No `.gitignore` observed** in `D:\workflow-lab\` (Explore agent didn't list one) — verify before any commit.
14. **`research-agent\queue.json` and `project-context.json`** exist but no entry point — orchestration mechanism not verified.
15. **n8n image hash** `b1b0c592735e` in earlier `docker ps -a` vs `docker.n8n.io/n8nio/n8n:latest` per `docker inspect` — verify the running container actually matches the latest tag.

---

## What's NOT yet known (questions for Phase 2 verification)

- Is the Windows Scheduled Task installed and firing? (`setup-windows-task.ps1`)
- Does `audit-memory-architect.log` show recent activity?
- Does `routing.json` actually get consulted by anything?
- Is the Obsidian → GitHub sync running? When was the last commit?
- Are the 5 "missing" docs deleted, never written, or living somewhere else?
- What's in the live `.env` files? (Not for the report — for the user to confirm secrets are correct.)

---

**End of Phase 1 report.** Ready to start Phase 2 (Health & Hygiene) on your signal.
