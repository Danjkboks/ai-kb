---
type: extract
date: 2026-06-05
slug: extractor-local-rebuild-watcher-scheduled
surface: claude-code
topics: [n8n, docker, infra, claude-code, knowledge-base]
skill_candidates: []
agent_candidates: [audit-agent]
duration_min: 120
---

## What Happened
- Executed full PLAN.md: rerouted n8n session-knowledge-extractor from GitHub push → local vault write
- Added Docker volume mount `D:/aidirectory/knowledge:/data/knowledge:rw` to n8n compose file
- Discovered and fixed n8n 2.23 file-write restriction: added `N8N_RESTRICT_FILE_ACCESS_TO=/data/knowledge`
- Removed GitHub Commit Extract + Skill Candidates nodes from workflow; added Write Extract To Disk (readWriteFile v1.1)
- Fixed 3 bugs: truncation (first-50K + last-50K), DeepSeek date hallucination (prepend detected_at), watcher dedup (sent-files.log)
- Updated /wrap command target path: queue\pending → knowledge\extracts
- Created reconcile-extracts.ps1; moved 6 backlog EXTRACTs from queue\pending to vault
- Registered n8n-instance MCP server in ~/.claude.json (localhost:5678, bearer JWT)
- Ran /audit and /handover to document session
- Deployed independent audit agent: verified all 8 tasks, mock-tested 3 historical sessions → 3/3 extracts produced, date-injection fix confirmed working
- Converted session-watcher from always-on 2s-poll loop to twice-daily scheduled one-shot with Yes/No popup
- Scheduled task re-registered: daily 09:00 + 18:00, -Once -Prompt flags, 1h execution limit

## Decisions Made
- `readWriteFile` v1.1 over `writeBinaryFile`: latter not in n8n 2.23 registry
- MCP over REST for workflow edits: no N8N_API_KEY existed; MCP gives atomic update_workflow
- MCP registered on localhost not Cloudflare tunnel: stable, no rotation risk
- Popup auto-dismiss → SKIP (not run): respects "don't drain CPU during important work" intent
- No fix for extract slug-collision silent overwrite: by-design idempotency (deliberate choice)

## Errors Encountered
- `claude mcp add` not on PATH: edited ~/.claude.json directly via Python | resolved
- "file is not writable" on Write Extract To Disk (exec #382): n8n 2.23 file-access guard. Fixed with `N8N_RESTRICT_FILE_ACCESS_TO=/data/knowledge` | resolved
- Inline PowerShell through bash mangled by French locale (`-f` operator, extglob): write .ps1 to disk + `-File` invocation | resolved (pattern now documented)
- Task re-registration required elevation (UAC): used Start-Process -Verb RunAs | resolved

## What Worked
- n8n MCP `update_workflow` atomic 8-op batch: clean, no mid-workflow broken state
- .ps1-to-disk pattern for all complex PS: sidestepped every bash↔PS quoting failure
- Smoke test pattern (synthetic JSONL → webhook → poll vault): caught writability bug immediately
- Independent audit agent: caught 4 issues including real contamination bug (watcher racing tests)

## What Didn't Work
- Inline PowerShell in Bash (quoting, locale, `-f`): broken every time. Never again.
- Assuming volume mount alone sufficient for n8n file writes: wrong, second gate exists

## Suggested Improvements
- Add skill-candidate local Write node gated on `_has_candidates` (dropped when GitHub node removed)
- Backfill 32 historical queue/done sessions with no extract (or accept as loss)
- Live popup test with synthetic session to confirm dialog appearance for user

## Files Modified
- `D:\Vms\Dockers\N8N\docker-compose.yml`: volume mount + N8N_RESTRICT_FILE_ACCESS_TO (PLAN-authorized, outside aidirectory)
- `scripts\session-watcher.ps1`: sent-files.log dedup + -Once/-Prompt/-PromptTimeoutSec flags + do/while loop
- `scripts\_register-watcher-task.ps1`: twice-daily triggers, -Once -Prompt args, 1h limit
- `scripts\reconcile-extracts.ps1`: new
- `.claude\commands\wrap.md`: queue\pending → knowledge\extracts
- `~\.claude.json`: n8n-instance MCP entry
- `knowledge\audits\AUDIT_2026-06-05_extractor-local-only-rebuild.md`: new
- n8n workflow `7l8aP0slan6tRkMy`: 8-op rewrite (not a file)

## Next Session Should Know
- Extractor is LOCAL only — no GitHub. End-to-end verified (exec #383 success).
- n8n 2.23 REQUIRES `N8N_RESTRICT_FILE_ACCESS_TO=/data/knowledge` in compose or writes fail "not writable". File: `D:\Vms\Dockers\N8N\docker-compose.yml`.
- Watcher now twice-daily scheduled (09:00 + 18:00) with Yes/No popup. Not a loop. Re-register with `_register-watcher-task.ps1` elevated if task breaks.
- Reconciliation baseline: 47 done / 21 extracts / 32 missing (pre-fix historical failures).
- Skill-candidate output dropped — no local replacement for removed GitHub skill node.
- n8n MCP registered: `n8n-instance` in `~\.claude.json`, localhost:5678, bearer JWT.
- sent-files.log at `data\audits\sent-files.log` (5 entries as of session end).

## Knowledge Candidates
- SOP: n8n file write restrictions (N8N_RESTRICT_FILE_ACCESS_TO) — worth adding to sop_n8n_mcp-setup.md
- Pattern: always use .ps1-to-disk + -File for PS on French Win locale — add to CLAUDE.md operational notes
- Runbook: watcher task registration (elevated, two triggers, -Once -Prompt) — update runbook_stack_master-build.md
