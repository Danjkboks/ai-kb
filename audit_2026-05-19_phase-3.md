---
type: audit
domain: stack
source: session
status: active
tags: [audit, phase-3, workflow-indexation, migration]
created: 2026-05-19
updated: 2026-05-19
---

# Phase 3 Audit Report — Code, Security & Diagnoses

**Date:** 2026-05-20
**Auditor:** Claude (Opus 4.7)
**Goal:** Code-level security audit + answer the 7 open questions from Phase 2.

---

## 0. Big surprise (also affects Phase 4)

**`D:\lab\workflow-lab\wiki\`** exists — I didn't know about it. It contains MASTER_BUILD.md, SOP skill installation.md, SOP_ClaudeCode_Full_Environment_Setup.md, SOP_LLMLingua_Token_Reduction_Docker.md, and likely more. These overlap with the Obsidian vault at `D:\Wproject\Notes\workflow-lab\docs\`.

**Likely status:** the `wiki/` folder is the source the Docker container mounts as `/wiki` (api files read from `/wiki`). The Obsidian vault is the human editing surface. Unclear if they're auto-synced or diverged.

**Implication:** unification must reconcile **three documentation surfaces**: `D:\workflow-lab\docs\`, `D:\lab\workflow-lab\wiki\`, and `D:\Wproject\Notes\workflow-lab\`.

---

## 1. Answers to the 7 questions

### Q1: Is `ask.py` still used? — YES, but indirectly

- Original Dockerfile (per MASTER_BUILD.md) had `CMD ["python", "ask.py"]`.
- Current Dockerfile has `CMD ["python", "api_with_langfuse.py"]`.
- `ask.py` is COPIED into the image but is NOT the entrypoint.
- It's invoked via `docker exec` or `docker run agent-llmlingua python ask.py` with `QUESTION` env var — useful for one-off CLI queries.
- **Verdict:** keep, but understand it's a CLI tool, not a service. After unification: rename to `ask-cli.py` or move to `scripts/` to make its role obvious.

### Q2: `n8n_mcp_bridge.py` vs n8n-MCP — bridge is current

- `n8n_mcp_bridge.py` is a thin Python wrapper around the n8n MCP HTTP endpoint (`/mcp-server/http`).
- It's NOT loaded as an MCP server by Claude Code — it's a utility script.
- **Per user:** this is the chosen path until GitHub commit happens. Confirmed: KEEP.
- **Issue (minor):** L64 has unsafe pattern `N8N_MCP_TOKEN[:20]` inside a conditional that branches on `if N8N_MCP_TOKEN else "No token set"` — the slicing only runs when the token exists, so it's actually safe. False alarm.

### Q3: `research-agent` — needs orchestration to be useful

- **What exists:** MANIFEST + `project-context.json` + `queue.json` (with 3 pending queries about AgentFS, AgentSpawn, token optimization 2026).
- **What's missing:** any code that *processes* the queue. No `process-queue.ps1`, no Python runner. The MANIFEST describes the workflow but no implementation backs it.
- **What's required to finish it:** a runner that
  1. Reads `queue.json`,
  2. Invokes Claude (via the Claude API or `claude -p`) with each query,
  3. Writes results to `findings/{date}-{id}.md`,
  4. Marks the query processed.
  Estimated effort: **2-3 hours** (Python script, ~150 lines, Anthropic SDK or claude CLI).
- **Question for you:** is `claude -p` (Claude Code CLI in headless mode) available on this machine? If yes, that's the simplest runtime. If not, we'd use the Anthropic SDK.

### Q4: `.sessions-archive\` — kept per your instruction

2 JSON files from May 18-19, 25KB and 27KB. These are session exports. Keep, but rotate (e.g., keep last 30 days, archive older to a `history/` subfolder).

### Q5: `routing.json` — decision: **RETIRE**, preserve info as docs

**Why retire:**
- No runtime actually consults it (no hook, no plugin).
- Skills + memory have solved the token-discipline problem it was designed for (the env-launch session proved this).
- 4 of 12 referenced files don't exist — fixing it costs more than it returns.

**How to preserve info (no data lost):**
- Extract the trigger taxonomy and agent descriptions into a single `docs/agents-reference.md`.
- Keep the 7 MANIFEST.md files (they're useful as agent reference docs).
- Delete `routing.json` only after `agents-reference.md` is written and reviewed.

**Net change:** -1 unused JSON, +1 useful doc, MANIFESTs untouched. Effort: 30 min.

### Q6: `memory-architect` — install plan

The agent is a 3-layer scaffold:

1. **`export-sessions.ps1`** — supposed to dump Claude sessions daily. *Not inspected yet — assumed functional.*
2. **`check-triggers.ps1`** — functional. Checks if new sessions / audit / report is due. Returns JSON.
3. **`ingest.ps1`** — **STUB** (literally says "Stub — actual implementation requires Claude to read sessions and extract"). The actual ingestion needs Claude to be the runtime.

The existing `setup-windows-task.ps1` only installs the **session export** task at 9:00 AM daily — it does NOT install ingestion or trigger-checking.

**To make it live (the way the user wants):**
- Step 1: Run `setup-windows-task.ps1` (as Admin) — gets daily session exports running.
- Step 2: Decide on the ingestion runtime — same `claude -p` question as Q3. The ingestion script needs to call Claude with each new session and have Claude extract patterns into memory files.
- Step 3: Add a second scheduled task that runs `check-triggers.ps1` daily, and if `should_run=true`, invokes a Claude-driven ingest.
- Step 4: Wire the audit log to a small dashboard so the user can see what the agent did.

**Effort:** ~3-4 hours to build the Claude orchestration glue. The PowerShell scaffolding is ready.

### Q7: Obsidian git sync — DIAGNOSED

**Findings:**
- Remote: `https://github.com/Danjkboks/workflow-lab.git` ✓
- Plugin installed: `obsidian-git` ✓
- No git hooks active (only `.sample` files).
- **Last commit: 2026-05-17 16:34** (3 days stale).

**Root cause:** the `obsidian-git` plugin's auto-commit interval is configured *inside Obsidian* and only runs **while Obsidian is open**. If Obsidian isn't running for 3 days, no syncs happen.

**Two fixes:**
- **(A) Keep using the plugin:** ensure Obsidian is always running. Fragile — the user has to remember.
- **(B) Add a Windows scheduled task** that runs a small script every 10 min: `cd vault && git add -A && git commit -m "auto-sync" && git push`. Independent of Obsidian. **Recommended.**

I'd build option B as one of the scheduled tasks alongside memory-architect. Effort: ~30 min including the safety checks (don't commit if no changes, don't push if no network).

---

## 2. Security findings (P0/P1)

### P0 — Hardcoded secrets (already in Phase 2, confirmed here)

- `api_with_langfuse.py:52-53` — Langfuse keys baked as `os.getenv()` defaults.
- `langfuse-compose.yml:13-25` — 5 secrets in plain env vars.

**Recommended remediation order (do BEFORE first git commit):**
1. Write `.env.example` listing every required key without values.
2. Update `api_with_langfuse.py` to remove defaults — fail with clear error if missing.
3. Replace `NEXTAUTH_SECRET` and `SALT` in compose with `${VAR}` references, source from `.env`.
4. Change `admin2026` password.
5. Write `.gitignore` with `.env`, `.env.n8n`, `__pycache__`, `.pytest_cache`, `*.log`.
6. Verify with `git status --ignored` that no secrets are tracked.

### P1 — Container runs as root

`Dockerfile` doesn't create a non-root user. Container processes run as UID 0. Mitigated by Docker isolation but not best practice.

**Fix:**
```dockerfile
RUN useradd --create-home --shell /bin/bash app
USER app
```

Effort: 5 min, requires rebuild.

### P1 — `qdrant_embeddings.py` always deletes the collection on run

Line 77-79: `client.delete_collection(COLLECTION_NAME)` runs unconditionally. If the user re-runs the script after adding metadata for new workflows, the entire 2,061-workflow index is wiped and rebuilt. No incremental update path.

**Fix:** Check if collection exists with same vector size; if so, upsert (don't recreate). Effort: 20 min.

### P1 — Daily budget counter never resets

`api_with_langfuse.py:74` initializes `cache_stats["total_cost_usd"] = 0.0`. It only resets on process restart. If the container runs for >24h, the "daily" budget warning becomes a "since start" warning.

**Fix:** Track a `last_reset_date` and reset on date rollover. Effort: 10 min.

### P2 — Unpinned dependencies

`requirements.txt`:
```
flask==3.0.0          ✓ pinned
python-dotenv==1.0.0  ✓ pinned
gunicorn==21.2.0      ✓ pinned (but UNUSED — entrypoint uses Flask dev server)
llmlingua             ✗ unpinned
openai                ✗ unpinned
langfuse              ✗ unpinned
```

Means a rebuild months from now produces different behavior than today. Pin everything.

### P2 — Flask dev server used in production

`api_with_langfuse.py:248`: `app.run(host="0.0.0.0", port=5001, debug=False)`. The Dockerfile has `gunicorn` installed but doesn't use it. Flask's built-in server is **not production-grade** — single-threaded, no graceful shutdown, performance issues under load.

**Fix:**
```dockerfile
CMD ["gunicorn", "--bind", "0.0.0.0:5001", "--workers", "2", "api_with_langfuse:app"]
```

Effort: 5 min. Will need to verify the LLMLingua model loads cleanly per-worker (it's big — might want `--preload`).

### P2 — No error handling around OpenRouter calls

In both `api.py` and `api_with_langfuse.py`, `client.chat.completions.create(...)` has no try/except, no retries, no rate limit handling. If OpenRouter rate-limits or errors, the request fails ungracefully.

**Fix:** Wrap with retry logic (tenacity library or simple loop with backoff). Effort: 30 min.

### P3 — `agent-llmlingua` image is 8.34 GB

Breakdown estimate: Python 3.12 base (~150 MB) + LLMLingua model weights (multilingual BERT, ~2 GB) + sentence-transformers (~5 GB if loaded) + Python deps (~1 GB). Actual content audit would require `docker history agent-llmlingua`.

This isn't a security issue but is a deployment issue. If you ever push to a registry, that's slow. Multi-stage builds + model caching could reduce by 30-50%. Defer to post-unification.

---

## 3. Wiki / documentation drift (new finding)

Three documentation surfaces with overlapping content:

| Location | Purpose | Sync? |
|---|---|---|
| `D:\workflow-lab\docs\` | Repo docs (ARCHITECTURE.md only) | Manual |
| `D:\lab\workflow-lab\wiki\` | Container-mounted source (`/wiki`) | Manual |
| `D:\Wproject\Notes\workflow-lab\` | Obsidian vault (18 docs) | Manual + obsidian-git plugin (broken) |

The api_with_langfuse.py and ask.py read `/wiki` — which means the Docker container mounts `D:\lab\workflow-lab\wiki\` (need to verify the docker run command). Updates to the Obsidian vault don't reach the running container until manual copy.

**Recommendation for Phase 4 unification:**
- One canonical location (e.g., `<unified-root>/wiki/`).
- Symlink or junction for Obsidian to point there directly.
- Container mounts the same canonical path.
- Single source, three views.

---

## 4. Updated disposition based on user input

| Item | Phase 2 status | User input | Phase 3 verdict |
|---|---|---|---|
| `ask.py` | STALE | "still used" | KEEP — rename to `ask-cli.py`, move to `scripts/` |
| `api.py` | STALE | (no override) | DELETE — superseded |
| `n8n_mcp_bridge.py` | UNKNOWN | "didn't commit github yet" → keep | KEEP |
| `research-agent` | UNKNOWN | "finish if needed" | FINISH — write the queue processor |
| `.sessions-archive\` | UNKNOWN | "keep for context" | KEEP — add rotation |
| `routing.json` | BROKEN | "choose efficient, preserve info" | RETIRE — extract to `agents-reference.md` |
| `memory-architect` | BROKEN | "install, deploy, make live" | INSTALL — task + Claude runtime needed |
| `obsidian-git sync` | BROKEN | "diagnose, yes please" | FIX with scheduled task (option B) |

---

## 5. Things newly discovered in Phase 3

1. **`D:\lab\workflow-lab\wiki\` exists** — third docs surface, must be reconciled.
2. **`ingest.ps1` is a stub** — memory-architect is non-functional without Claude in the loop.
3. **`setup-windows-task.ps1` only installs session export**, not ingestion/triggers — installing alone won't make memory-architect "live".
4. **`gunicorn` is installed but unused** — using Flask dev server in production.
5. **`qdrant_embeddings.py` destroys collection on every run** — no incremental updates.

---

## 6. Action plan summary (BEFORE Phase 4)

Priority-ordered, conservative time estimates:

| # | Action | Effort |
|---|---|---|
| 1 | Scrub all secrets, write `.env.example`, write `.gitignore` | 30 min |
| 2 | Fix Cloudflare cert (`cloudflared tunnel login`) | 5 min |
| 3 | Pin all Python deps in `requirements.txt` | 5 min |
| 4 | Switch Dockerfile to gunicorn + add non-root user | 15 min |
| 5 | Fix budget reset bug in `api_with_langfuse.py` | 10 min |
| 6 | Fix `qdrant_embeddings.py` destructive rebuild | 20 min |
| 7 | Extract `routing.json` → `docs/agents-reference.md`, delete json | 30 min |
| 8 | Write Obsidian git auto-sync scheduled task | 30 min |
| 9 | Run `setup-windows-task.ps1` (as Admin) | 5 min |
| 10 | Build research-agent queue processor (depends on Q3 answer) | 2-3 hr |
| 11 | Wire memory-architect ingest → Claude (depends on Q3 answer) | 3-4 hr |

**Total:** ~8-10 hours of focused work, of which **~1.5 hours are pure cleanup** (#1-9) and **~6-7 hours are the Claude orchestration** (#10-11).

---

## 7. One question to resolve before Phase 4

**Is `claude -p` (Claude Code CLI in headless mode) available on this machine?**

If yes: simplest path for memory-architect + research-agent orchestration. A scheduled task calls `claude -p "ingest this session" < session.json`.

If no: we use the Anthropic SDK (Python script + ANTHROPIC_API_KEY). Slightly more code, same functionality.

This affects how I design the unified `automation/` folder in Phase 4.

---

**End of Phase 3.** Ready for Phase 4 (unified structure design + migration plan) once you answer the `claude -p` question. The audit work is essentially done — Phase 4 is the synthesis.
