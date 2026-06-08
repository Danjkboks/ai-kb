---
type: extract
date: 2026-06-05
session_id: extractor-local-rebuild-watcher-scheduled
surface: chat
environment: both
topics: [n8n, docker, memory, infra, python]
source_file: 2026-06-05-124742_12fe7480-aeda-4cc8-be44-a28788ac71cd.jsonl
processed_at: 2026-06-07T16:02:47.300Z
---

# Session Extract: extractor-local-rebuild-watcher-scheduled

## Decisions
- **Remove GitHub dependency from extractor pipeline and write extracts directly to local vault**: GitHub commit node was causing 422 errors and pipeline failures; local file write is more reliable and eliminates external dependency
- **Convert session-watcher from AtLogOn trigger to twice-daily scheduled task with user confirmation popup**: AtLogOn continuous loop was causing duplicate file processing; scheduled task with user gate prevents unwanted execution and reduces system load
- **Accept historical loss of 32 missing extracts rather than attempting complex backfill**: Pre-fix pipeline failures resulted in missing extracts; re-POSTing historical sessions would be complex and error-prone; accept loss as known data gap
- **Use Docker volume mount for n8n container to access knowledge extracts directory**: N8N RESTRICT_FILE_ACCESS prevents writing outside /data; volume mount allows n8n workflow to write to D:\aidirectory\knowledge\extracts via container path

## Problems Solved
- **GitHub commit node returning 422 errors causing pipeline failures**: Removed GitHub Commit node and replaced with Write Binary File node writing directly to local vault via Docker volume mount
- **Truncation strategy discarding important context (only first 100K chars)**: Modified truncation to keep first 50K + last 50K characters to preserve both decisions and resolutions
- **DeepSeek date hallucination in extracted JSON**: Prepend session date from webhook payload to user message in OpenRouter request body
- **Session watcher processing same files multiple times due to in-memory deduplication**: Added persistent sent-files.log to D:\aidirectory\data\audits\ loaded on startup, appended on success, skip if present in poll loop
- **Pipeline failures not visible in watcher logs (only shows 200 OK to webhook)**: Created reconciliation script (reconcile-extracts.ps1) to compare done-files vs source_file extracts and report missing extracts

## Errors Encountered
- [resolved] GitHub commit node returning 422 errors -> Removed GitHub dependency entirely, writing extracts locally
- [resolved] N8N RESTRICT_FILE_ACCESS preventing writes outside /data directory -> Added Docker volume mount from D:\aidirectory\knowledge to container /data/knowledge
- [resolved] 32 historical extracts missing due to pre-fix pipeline failures -> Accept historical loss, sync delete stale workflow-lab agents, no backfill attempted

## Patterns Identified
- Windows PowerShell 7.6.2.0 Store install path breaks on upgrade - pin to stable MSI install in Program Files
- N8N 2.23 file access restrictions require Docker volume mounts for external file writes
- Session watcher needs persistent deduplication across restarts - file-based log vs in-memory
- Truncation should preserve both beginning and end of long sessions (first 50K + last 50K)
- LLM date hallucination can be fixed by injecting known date into user message rather than system prompt

## Files Modified
- modified: D:\aidirectory\HANDOVER.md -- Updated CODE and COWORK sections with new extractor pipeline architecture, bug fixes, and watcher scheduling changes
- modified: D:\aidirectory\scripts\session-watcher.ps1 -- Added persistent sent-files.log deduplication, changed from AtLogOn to scheduled task pattern
- created: D:\aidirectory\scripts\reconcile-extracts.ps1 -- Created reconciliation script to compare done-files vs extracts and report pipeline failures
- modified: n8n workflow 7l8aP0slan6tRkMy -- Removed GitHub Commit node, added Write Binary File node, fixed truncation (first 50K+last 50K), added date injection to DeepSeek prompt
- created: D:\aidirectory\data\audits\sent-files.log -- Created persistent log of processed files for session-watcher deduplication

## Next Session Must Know
- Extractor pipeline now writes locally via Docker volume mount at /data/knowledge/extracts
- Session watcher runs twice daily (09:00 & 18:00) with user confirmation popup, not AtLogOn
- 32 historical extracts are missing and accepted as loss - no backfill planned
- Reconciliation script at D:\aidirectory\scripts\reconcile-extracts.ps1 reports pipeline failures
- Three bugs fixed: truncation (first 50K+last 50K), date injection, persistent deduplication
- GitHub dependency removed - all extracts go to local vault only
- N8N workflow ID 7l8aP0slan6tRkMy modified with Write Binary File node
- Docker volume mount connects D:\aidirectory\knowledge to container /data/knowledge

## Skill Candidates
- write-extract-disk: Writes session extract JSON as markdown to local knowledge vault via Docker volume mount
- reconcile-extracts: Compares done-files directory with knowledge extracts to identify pipeline failures
- session-watcher-dedupe: Manages persistent file deduplication log for session watcher across restarts

## Token Waste Flags
- Repeated description of same three bugs multiple times in handover updates
- Detailed re-listing of MCP tools that weren't used in session
- Multiple iterations of HANDOVER.md edits with similar content

## Knowledge Base Updates
- [update] sop_n8n_mcp-setup.md: Add pattern for local file writes via Docker volume mounts due to N8N RESTRICT_FILE_ACCESS
- [update] runbook_stack_master-build.md: Include session extractor pipeline architecture changes (GitHub removal, local writes, volume mounts)
- [create] sop_memory_extractor-pipeline.md: Document new extractor pipeline architecture, bug fixes, and reconciliation procedures
- [create] ref_infra_docker-volume-mounts.md: Document Docker volume mount patterns for n8n file access restrictions

## Links
related:: [[_INDEX]]
tags: n8n, docker, memory, infra, python
