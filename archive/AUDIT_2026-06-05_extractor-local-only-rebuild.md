---
type: audit
date: 2026-06-05
session_id: extractor-local-only-rebuild
surface: claude-code
duration_min: 90
---

## What Was Built / Changed
- Docker: added volume mount `D:/aidirectory/knowledge:/data/knowledge:rw` + env `N8N_RESTRICT_FILE_ACCESS_TO=/data/knowledge` to `D:\Vms\Dockers\N8N\docker-compose.yml`; container recreated twice.
- n8n workflow `7l8aP0slan6tRkMy`: removed `GitHub Commit Extract`, `GitHub Commit Skill Candidates`, and `Has Skill or Agent Candidates` (IF). Added `Write Extract To Disk` (readWriteFile v1.1 → `/data/knowledge/extracts/{{ $json._extract_filename }}`). Published.
- `Decode Base64` node: truncation changed from first-100K to first-50K + last-50K with a `[...MIDDLE TRUNCATED...]` marker.
- `Build OpenRouter Body` node: prepends `Today's date is: <detected_at[:10]>` to the DeepSeek user message (kills date hallucination).
- `Build Extract Markdown` node: now emits binary `data` field (markdown) instead of base64 + `_has_candidates`.
- `scripts\session-watcher.ps1`: persistent dedup via `data\audits\sent-files.log` (load on start, append on success, skip-if-present in poll).
- `.claude\commands\wrap.md`: target path `queue\pending` → `knowledge\extracts` (Steps 1 & 3).
- New `scripts\reconcile-extracts.ps1`: compares `queue\done\*.jsonl` against `source_file:` in extracts.
- Moved 6 backlog EXTRACTs from `queue\pending` → `knowledge\extracts`.
- Registered `n8n-instance` MCP in `~\.claude.json` (http://localhost:5678/mcp-server/http, bearer token).

## Decisions Made
- Local write via `readWriteFile` node (not Write Binary File): `writeBinaryFile` is deprecated/not in this n8n's node registry; `readWriteFile` v1.1 is the supported equivalent.
- MCP over REST API for workflow edits: PLAN intended MCP, and it gives atomic `update_workflow` + `validate_workflow`. No `N8N_API_KEY` existed in `.env`/env, so REST was not viable anyway.
- MCP registered against `localhost`, not the Cloudflare tunnel: Claude Code runs on the same host; localhost is faster and survives tunnel-URL rotation.
- Single atomic `update_workflow` batch (8 ops) for Tasks 2/3/4: minimizes activate/deactivate cycles per PLAN.

## Errors Encountered
- `claude mcp add` CLI not on PATH in PowerShell: edited `~\.claude.json` directly via Python (PS `ConvertFrom-Json -Depth` unsupported on Win PS 5.1).
- `"The file ... is not writable"` at Write Extract To Disk (exec #382): n8n 2.23 restricts file access outside its data dir. Fixed by adding `N8N_RESTRICT_FILE_ACCESS_TO=/data/knowledge` → exec #383 success.
- Multiple PowerShell one-liner failures via Bash (`-f` operator, `extglob` mangling, backtick-n): switched to writing `.ps1` files and invoking with `-File`. Pattern confirmed: do not inline complex PS through bash on French Win locale.

## Token Usage (estimate)
- Input: ~80K | Output: ~12K | Compression: N

## What Worked
- n8n MCP `update_workflow` atomic batch — 8 operations applied cleanly, validated, published in one shot.
- Writing `.ps1` to disk then `-File` invocation — sidesteps all bash↔PS quoting/locale issues.
- Smoke test via synthetic JSONL → webhook → check vault + execution status: caught the writability bug immediately.

## What Didn't Work
- Inline PowerShell through Bash tool (quoting, `-f`, locale) — repeatedly mangled. Avoid.
- First assumption that the volume mount alone was sufficient — n8n's file-access guard is a separate gate.
- `writeBinaryFile` node lookup — not present; wasted one get_node_types call.

## Suggested Improvements
- Skill-candidate capture was dropped with the GitHub node — add a second local Write gated on `_has_candidates` if still wanted.
- Backfill the 32 historical `queue\done` sessions with no extract, or document them as accepted loss.
- Persist watcher SKIP/backoff counters to disk (only sent-files.log is persistent today).

## Files Modified
- `D:\Vms\Dockers\N8N\docker-compose.yml`: volume mount + file-access env (outside aidirectory — PLAN-authorized).
- `scripts\session-watcher.ps1`: sent-files.log dedup.
- `scripts\reconcile-extracts.ps1`: new.
- `.claude\commands\wrap.md`: path retarget.
- `~\.claude.json`: n8n-instance MCP entry (+ `.bak` backup).
- n8n workflow `7l8aP0slan6tRkMy`: 8-op rewrite (not a file).

## Next Session Should Know
- Extractor is now fully LOCAL — no GitHub. End-to-end verified (exec #383 success), output format matches existing extracts.
- n8n REQUIRES `N8N_RESTRICT_FILE_ACCESS_TO=/data/knowledge` or writes fail as "not writable". This is in the compose file at `D:\Vms\Dockers\N8N\`.
- Reconciliation baseline: 47 done / 21 extracts / 32 missing (historical pre-fix failures, not a regression).
- Skill-candidate output is gone (no local replacement for the removed GitHub skill node).
- n8n MCP token is a bearer JWT in `~\.claude.json`; localhost endpoint.
