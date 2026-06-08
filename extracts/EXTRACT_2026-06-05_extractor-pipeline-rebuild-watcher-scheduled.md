---
type: extract
date: 2026-06-05
session_id: extractor-pipeline-rebuild-watcher-scheduled
surface: chat
environment: both
topics: [n8n, docker, memory, infra, python]
source_file: 2026-06-05-130303_12fe7480-aeda-4cc8-be44-a28788ac71cd.jsonl
processed_at: 2026-06-07T16:02:55.563Z
---

# Session Extract: extractor-pipeline-rebuild-watcher-scheduled

## Decisions
- **Route extractor pipeline from GitHub to local vault, removing GitHub dependency**: To eliminate GitHub commit node 422 errors and make pipeline fully local, writing extracts directly to knowledge vault via Docker volume mount
- **Convert session-watcher from AtLogOn trigger to twice-daily scheduled task (09:00 + 18:00) with Yes/No popup**: To prevent duplicate file processing and give user control over when watcher runs, replacing continuous loop with one-shot drain
- **Accept historical loss of 32 missing extracts from pre-fix pipeline failures, not attempting backfill**: Historical sessions before bug fixes cannot be recovered; focus on fixing current pipeline rather than complex backfill

## Problems Solved
- **GitHub commit node returning 422 errors causing pipeline failures**: Removed GitHub node entirely, replaced with Write Binary File node writing to Docker-mounted volume at D:\aidirectory\knowledge\extracts\
- **Truncation strategy discarding important content (only first 100K chars)**: Modified truncation to take first 50K + last 50K with separator to preserve decisions and resolutions at both ends
- **DeepSeek date hallucination in extracted JSON**: Inject session date from webhook payload into user message prepended to DeepSeek prompt: 'date: {sessionDate}\n'
- **Session watcher processing same files multiple times due to duplicate processes**: Added persistent sent-files.log loaded at startup, appending successfully processed filenames, skipping if already in log

## Errors Encountered
- [resolved] GitHub commit node returns 422 error on re-run -> Removed GitHub node entirely, routing to local file write instead

## Patterns Identified
- Windows PowerShell 5.1 event action blocks lose variable scope - use polling instead
- N8N RESTRICT FILE ACCESS prevents direct file writes from container - must use Docker volume mounts
- Session watcher duplicate processing requires persistent disk-based deduplication, not in-memory only

## Files Modified
- modified: D:\aidirectory\HANDOVER.md -- Updated COWORK and SHARED sections with new pipeline state, watcher schedule, and bug fixes
- modified: D:\aidirectory\scripts\session-watcher.ps1 -- Added sent-files.log persistent deduplication, changed from AtLogOn to scheduled trigger logic
- created: D:\aidirectory\scripts\reconcile-extracts.ps1 -- Created reconciliation script to compare done-files vs source_file extracts, identifying pipeline failures

## Next Session Must Know
- Docker volume mount added: D:\aidirectory\knowledge mapped to /data/knowledge in n8n container
- Workflow 7l8aP0slan6tRkMy modified: GitHub node removed, Write Binary File node added for local extracts
- Session watcher now scheduled twice daily (09:00 + 18:00) with Yes/No popup, not AtLogOn
- Reconciliation baseline: 47 done files, 27 extracts, 26 missing (historical pre-fix losses accepted)
- Three bugs fixed: truncation (50K+50K), date injection, watcher deduplication via sent-files.log
- LLMLingua compress endpoint at localhost:5001/compress, workflow uses OpenRouter with DeepSeek v3.2
- Cloudflare tunnel URL changes on restart - n8n at localhost:5678, tunnel exposes externally
- Stale duplicate file: D:\aidirectory\projects\workflow-lab\agents\api_with_langfuse.py needs deletion decision

## Skill Candidates
- write-extract-to-disk: Formats JSON extract to markdown and writes to knowledge vault via Docker volume mount
- session-watcher-dedupe: Implements persistent file deduplication using sent-files.log for PowerShell watcher scripts

## Token Waste Flags
- Repeated full context of HANDOVER.md structure in multiple edits
- Detailed re-explanation of already-implemented bug fixes in handover updates

## Knowledge Base Updates
- [update] sop_n8n_mcp-setup.md: Add details about local extract writing via Docker volume mounts, removing GitHub dependency
- [update] runbook_stack_master-build.md: Include session watcher scheduled task configuration and persistent deduplication pattern
- [create] sop_infra_extractor-pipeline-local.md: Document complete local extractor pipeline architecture, volume mounts, and bug fixes

## Links
related:: [[_INDEX]]
tags: n8n, docker, memory, infra, python
