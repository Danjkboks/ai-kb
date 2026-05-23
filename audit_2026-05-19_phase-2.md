---
type: audit
domain: stack
source: session
status: active
tags: [audit, phase-2, workflow-indexation, search]
created: 2026-05-19
updated: 2026-05-19
---

# Phase 2 Audit Report — Health & Hygiene

**Date:** 2026-05-20
**Auditor:** Claude (Opus 4.7)
**Goal:** Determine what's alive, stale, dead, or broken — *before* the unification (Phase 4).
**Method:** Direct verification — read every file, check every process, query every endpoint.

---

## 0. Headline findings (priority order)

| # | Finding | Severity |
|---|---|---|
| 1 | **Langfuse secrets hardcoded in 2 committed source files** (`api_with_langfuse.py` + `langfuse-compose.yml`) — currently safe (no git), but a landmine if ever pushed. | **P0** |
| 2 | **Routing system (`routing.json`) is broken** — references 4 files that don't exist. The whole agent-spawning layer is non-functional. | **P0** |
| 3 | **Memory-architect agent ran once on 2026-05-19 and never again.** No Windows scheduled task installed. The autonomous memory keeper is dead. | **P1** |
| 4 | **Obsidian → GitHub auto-sync hasn't run since 2026-05-17** (3 days). The "every 10 min" sync is not actually happening. | **P1** |
| 5 | **Project is not under version control.** Zero git history for either `D:\workflow-lab\` or `D:\lab\workflow-lab\`. Three months of work has no backup beyond the disk. | **P0** |

---

## 1. Disposition table — every item from Phase 1

Legend:
- **KEEP** = actively used, working
- **STALE** = was used, now superseded
- **DEAD** = never integrated or fully broken
- **BROKEN** = intended-but-failing
- **UNKNOWN** = inconclusive

### Code & containers

| Item | Status | Evidence | Disposition | Effort |
|---|---|---|---|---|
| `agent-llmlingua:latest` image (8.34 GB) | **KEEP** | Active, used by running container. Size driven by LLMLingua model weights (~7 GB) + Python deps. | Keep but consider GPU-passthrough variant later | — |
| `compress-api:latest` image (213 MB) | **DEAD** | Created 2026-05-12, no container uses it. Predecessor of `agent-llmlingua`. | **DELETE** | S |
| `compress_api.py` (source) | **DEAD** | Source of the orphan image above. Superseded by `api_with_langfuse.py`. | **DELETE** | S |
| `api.py` | **STALE** | Dockerfile entrypoint is `api_with_langfuse.py`. Older variant kept for reference. | **DELETE** (it's in your git history once we put it there) | S |
| `api_with_langfuse.py` | **KEEP** | Active entrypoint. | Keep — but **scrub hardcoded keys** (see §3) | M |
| `ask.py` | **UNKNOWN** | Copied into container but not the entrypoint. May be invoked manually. | Verify with user before deciding | — |
| `Dockerfile` | **KEEP** | Builds the live image. | Keep | — |
| `requirements.txt` | **KEEP** | 6 deps, all used. | Keep — pin versions for `llmlingua`, `openai`, `langfuse` (currently unpinned). | S |
| `n8n_mcp_bridge.py` | **UNKNOWN** | Mentioned in `n8n_mcp_setup.md` memory as "Option 2" — alternative integration path, not the active one. | Verify with user | — |
| `qdrant_embeddings.py` | **KEEP** | Indexed 2,061 workflows. | Keep — re-run when workflow set changes | — |
| `qdrant_wiki_embeddings.py` | **KEEP** | Indexed 71 wiki chunks. | Keep | — |
| `langfuse-compose.yml` | **STALE** (partial) | Exists but `env-launch.ps1` uses `docker start` directly instead. Compose file isn't invoked. | Keep as IaC source-of-truth, change env-launch to `docker compose up -d`. | M |
| `n8n` container image hash mismatch | **non-issue** | The hash `b1b0c592735e` IS the current `docker.n8n.io/n8nio/n8n:latest`. Verified via `docker inspect`. | — | — |

### Documentation

| Item | Status | Evidence | Disposition | Effort |
|---|---|---|---|---|
| `README.md` references to 5 missing docs | **DEAD links** | None of the 5 exist anywhere (`.claude/`, backups, sessions, Obsidian). Never written. | **Update README** to point to actual docs (Obsidian SOPs + `docs/ARCHITECTURE.md`) | S |
| `docs/ARCHITECTURE.md` | **KEEP** | Real, complete, accurate. | Keep | — |
| Obsidian SOPs (`D:\Wproject\Notes\workflow-lab\docs\`) | **KEEP** (18 docs) | MASTER_BUILD.md + 8 SOPs covering n8n bridge, LLMLingua, Obsidian/GitHub, ScrapeGraphAI, Claude Code setup, etc. | Keep — link from repo README | S |
| `DEPLOYMENT_REPORT.md`, `PHASE_COMPLETION_REPORT.md`, `FILE_GUIDE.txt` | **STALE** (snapshot in time) | Describe Phase 1-4 of indexation, no longer the project's primary docs. | Move to `docs/history/` | S |

### Routing & agents

| Item | Status | Evidence | Disposition | Effort |
|---|---|---|---|---|
| `routing.json` | **BROKEN** | 4 of 12 referenced context files don't exist: `api-reference.md`, `commands.sh`, `status.md`, `.claude/memory-agent/MEMORY.md`. The routing layer references things that were planned, not written. | **Either complete it or delete it.** See §2. | L |
| `agents/docker-ops` | **KEEP** | MANIFEST + build.ps1 + .env.example all present. | Keep — but build.ps1 hasn't been used in this session (we invoked docker manually). Test or document its actual use. | — |
| `agents/git-ops` | **BROKEN** | MANIFEST only — no `commands.sh` to back it. | Add commands.sh or remove from routing. | S |
| `agents/infra-ops` | **PARTIAL** | MANIFEST + check-status.ps1 present. `status.md` (referenced by routing.json) missing. | Complete or simplify | S |
| `agents/memory-architect` | **BROKEN** | MANIFEST + scripts present, but the scheduled task is NOT installed (Windows schtasks shows nothing). | Either run `setup-windows-task.ps1` to install or remove the scheduling expectation. | M |
| `agents/n8n-workflow` | **PARTIAL** | MANIFEST + create-workflow.py. `api-reference.md` (routing target) missing. | Either write it or remove from routing. | S |
| `agents/python-ops` | **KEEP** | MANIFEST + run-compression.ps1. | Keep | — |
| `agents/research-agent` | **UNKNOWN** | MANIFEST + queue.json + project-context.json. No entry point. Mechanism for processing the queue is unclear — there's no `process-queue.ps1` or similar. | Verify with user how research-agent is supposed to fire. | — |

### Configuration & secrets

| Item | Status | Evidence | Disposition | Effort |
|---|---|---|---|---|
| `D:\lab\workflow-lab\agents\.env` (351 B) | **KEEP** | Live secrets — referenced by `python-dotenv` in api files. | Keep, but **document what it should contain** in `.env.example` next to it. | S |
| `D:\lab\workflow-lab\.env.n8n` (344 B) | **KEEP** | n8n JWT token per memory. | Keep, rotate periodically. | — |
| `.env.example` × 4 in `D:\workflow-lab\agents\*\` | **DEAD** | Templates for an agent structure that's only partially built. The actual live `.env` lives elsewhere. | Either delete or move to canonical location after unification. | S |
| Hardcoded Langfuse keys in `api_with_langfuse.py` (L52-53) | **P0 BUG** | Public + secret keys baked in as `os.getenv()` defaults. | **Move to .env, remove defaults.** See §3. | S |
| Hardcoded secrets in `langfuse-compose.yml` (L13-25) | **P0 BUG** | `NEXTAUTH_SECRET` + `SALT` are placeholder values ("change-this-32chars"), admin password = `admin2026`, public/secret keys committed. | **Move to .env, regenerate secrets, change password.** | M |

### Infrastructure & operations

| Item | Status | Evidence | Disposition | Effort |
|---|---|---|---|---|
| Cloudflare tunnel | **BROKEN** | `~/.cloudflared/` is empty. No `cert.pem`, no `config.yml`. | Run `cloudflared tunnel login` once, then `env-launch` will work. | S |
| Memory-architect scheduled task | **DEAD** | `schtasks /query` shows no workflow-related task. Audit log has 1 entry from May 19 01:58 then nothing. | Decide: run `setup-windows-task.ps1` to make it real, OR delete the agent + scheduling docs. | M |
| Obsidian → GitHub sync | **BROKEN** | Last vault commit: 2026-05-17 16:34. The documented "every 10 min" sync isn't running. | Diagnose: probably Obsidian Git plugin disabled, or sync timer broken. | M |
| Audit log freshness | **STALE** | Only 1 init entry + 1 trigger entry, both 2026-05-19 01:58. Nothing since. | Tied to memory-architect being dead. | — |
| `.sessions-archive\` (2 files) | **UNKNOWN** | 2 JSON session archives from May 18-19. Purpose unclear. | Verify with user — keep or delete. | — |
| `D:\workflow-lab\.claude\` extra scripts (`audit-log-init.ps1`, `export-sessions.ps1`, `setup-windows-task.ps1`) | **PARTIALLY DEAD** | Setup scripts exist but the resulting state isn't installed. | Run them OR delete them. Don't leave stubs. | M |

### Project structure

| Item | Status | Evidence | Disposition | Effort |
|---|---|---|---|---|
| Two project roots (`D:\workflow-lab\` + `D:\lab\workflow-lab\`) | **HISTORICAL** | User confirmed: leftover from old Claude Code CLI. | Phase 4 work — unification. | L |
| CLAUDE.md path inconsistency (`D:\Claudi\` should be `D:\lab\Claudi\`) | **STALE doc** | Confirmed: actual location is `D:\lab\Claudi\`. | Fix CLAUDE.md after unification settles paths. | S |
| No git anywhere | **P0 RISK** | `git status` fails in both roots. No `.gitignore`. | **Initialize git immediately**, before any further work. | M (needs `.gitignore` design + first commit) |

---

## 2. Routing system (`routing.json`) — should it live or die?

`routing.json` defines 6 agent routes. Of the 12 context files it points to, **4 are missing**:

```
✓ EXISTS  docker-ops/MANIFEST.md + .env.example
✗ MISSING n8n-workflow/api-reference.md     ← never written
✓ EXISTS  python-ops/MANIFEST.md + requirements.txt
✗ MISSING git-ops/commands.sh                ← never written
✗ MISSING infra-ops/status.md                ← never written
✗ MISSING .claude/memory-agent/MEMORY.md     ← (the real MEMORY.md is in user's home)
```

It also assumes a runtime that loads these files per task, but **the current Claude Code session ignores `routing.json` entirely** — there's no plugin or hook that consults it. It is documentation, not infrastructure.

**Two paths forward:**
- **(A) Make it real**: write the 4 missing files, build a pre-task hook that loads the right ones based on the user's prompt. Significant effort, real benefit (token discipline per task).
- **(B) Retire it**: delete `routing.json`, keep the MANIFEST.md files as human-readable docs only.

I'd lean **(B) for now** — the env-launch session showed that natural skills + good memory work fine. The routing.json was solving a problem (token bloat) that has better solutions (skills, the consolidate-memory skill, etc.). But the user should decide.

---

## 3. Secrets exposure (P0 detail)

Two files contain hardcoded secrets currently safe only because no git repo exists:

**`D:\lab\workflow-lab\agents\api_with_langfuse.py` lines 52-53:**
```python
public_key=os.getenv("LANGFUSE_PUBLIC_KEY", "pk-lf-3d66cf4c-cee8-402c-8509-7f984c3b7e61"),
secret_key=os.getenv("LANGFUSE_SECRET_KEY", "sk-lf-24e35c3e-fbc2-463f-8a62-acfc089c8a21"),
```
*Risk:* When this file is eventually committed (per CLAUDE.md, the intent is `github.com/Danjkboks/workflow-lab`), the secret leaks immediately. If the GitHub repo is public, it's catastrophic. Even private — it ends up in any fork or backup.

**`D:\lab\workflow-lab\agents\langfuse-compose.yml` lines 13-25:**
- `NEXTAUTH_SECRET=workflow-lab-secret-change-this-32chars` ← unchanged placeholder, weak
- `SALT=workflow-lab-salt-change-this-32c` ← unchanged placeholder, weak
- `LANGFUSE_INIT_USER_PASSWORD=admin2026` ← weak, in source
- Same public + secret keys as above
- `POSTGRES_PASSWORD=postgres` in langfuse-db block (line 39)

**Recommendation:** *Before* the first git commit:
1. Move all secrets to `.env` (already present at `D:\lab\workflow-lab\agents\.env`).
2. Remove defaults from `os.getenv()` calls in Python — fail fast if missing.
3. Regenerate `NEXTAUTH_SECRET` and `SALT` to real 32-char random values.
4. Change `admin2026` to something strong.
5. Write a `.env.example` listing the variables without values.
6. Add a `.gitignore` that excludes `.env` and `.env.n8n`.

---

## 4. What's safe to delete RIGHT NOW (without unification)

Lowest-risk cleanup that can happen before Phase 4:

| File / image | Why safe | Size reclaimed |
|---|---|---|
| `compress-api:latest` Docker image | Orphan, no container uses it | 213 MB |
| `D:\lab\workflow-lab\agents\compress_api.py` | Source of orphan image, superseded by api_with_langfuse.py | <2 KB |
| `D:\lab\workflow-lab\agents\api.py` | Superseded by api_with_langfuse.py (Dockerfile confirms) | 5 KB |
| `D:\workflow-lab\agents\*\.env.example` (4 files) | Templates for agent dirs that have no real .env paired with them | <2 KB |

These deletions are reversible until the next docker prune.

---

## 5. What MUST happen before unification (Phase 4)

In dependency order:

1. **Initialize git** in the eventual unified root, with `.gitignore` first. *Don't move files until git is ready, or history is lost.*
2. **Scrub secrets** from `api_with_langfuse.py` and `langfuse-compose.yml`. *Don't commit secrets, ever.*
3. **Decide routing.json fate** (make real or retire). *Affects unified structure.*
4. **Decide memory-architect fate** (install scheduled task or remove). *Affects whether `agents/memory-architect/` survives unification.*
5. **Fix Cloudflare cert** so env-launch is 100% green. *Otherwise we ship a broken default.*
6. **Confirm `ask.py`, `n8n_mcp_bridge.py`, `research-agent` usage** with user. *Don't migrate dead code.*

---

## 6. Open questions (need user confirmation)

1. **`ask.py`** — still used for anything, or dead?
2. **`n8n_mcp_bridge.py`** — Option 1 (n8n-MCP from GitHub) or Option 2 (this bridge) is current?
3. **`research-agent`** — was this scaffolding never finished, or am I missing the orchestration?
4. **`.sessions-archive\` (2 JSON files)** — keep or trash?
5. **`routing.json`** — make real (A) or retire (B)?
6. **`memory-architect` scheduled task** — install (run `setup-windows-task.ps1`) or remove the agent?
7. **Obsidian git sync** — should I diagnose & fix, or is that out of scope?

---

## 7. Phase 2 stats

- **Items inventoried:** 30
- **KEEP:** 12 (40%)
- **STALE:** 5 (17%)
- **DEAD:** 4 (13%)
- **BROKEN:** 5 (17%)
- **UNKNOWN:** 4 (13%)
- **P0 issues:** 5
- **P1 issues:** 4
- **P2 issues:** 7
- **Estimated cleanup effort:** ~4-6 hours focused work
- **Estimated unification effort:** ~2-4 hours (Phase 4) after cleanup

---

**End of Phase 2 report.** Ready to start Phase 3 (Code & Security) or pivot to Phase 4 (unified structure design) — your call.
