---
type: extract
date: 2026-06-05
session_id: watcher-retry-storm-llmlingua-fix
surface: claude-code
environment: env1-claude-desktop
topics: [infra, python, docker, n8n, memory]
source_file: 2026-06-05-115802_dbd9ecf2-b5aa-45c7-b90d-9e7ca00ff517.jsonl
processed_at: 2026-06-05T10:19:50.407Z
---

# Session Extract: watcher-retry-storm-llmlingua-fix

## Decisions
- **Add per-file retry tracking with backoff and skip logic to session-watcher.ps1**: To prevent retry storm cascade where 6 files were hammered 144 times in minutes due to 2s polling loop re-detecting failed files
- **Add MAX_INPUT_CHARS=100,000 guard to LLMLingua Flask app compress endpoint**: To prevent OOM crash loop when large JSONL files (2509KB) with 1.4M tokens exceed LLMLingua's 512 token limit, causing 2300% CPU usage
- **Use PowerShell 7 (pwsh.exe) for Windows-side commands with ASCII strings in PS1 files**: French locale Windows 11 causes regex matching issues with .NET exception messages; PS7 avoids locale-specific string issues

## Problems Solved
- **Watcher retry storm: 6 backlog JSONL files hammered 144 times in minutes due to 2s polling loop re-detecting failed files**: Added per-file retry tracking hashtable, BACKOFF after 3 failures (5min wait), SKIP after backoff expires and fails again, PAUSE 30s before any retry attempt
- **LLMLingua container crash loop with 2300% CPU from oversized inputs (2509KB JSONL, 1.4M tokens)**: Added MAX_INPUT_CHARS=100,000 guard before LLMLingua call, truncate with warning log instead of rejecting, patched container via docker cp and restart
- **French locale Windows causes regex matching failures on English .NET exception messages**: Use SocketErrorCode (Win 10061) instead of string matching for 'connection refused', verified with mock-test agent

## Errors Encountered
- [resolved] LLMLingua 500 error on degenerate tokenless input ('x' * 250000) causing truncated raw text failure -> Added input size guard (MAX_INPUT_CHARS=100,000) before LLMLingua call
- [resolved] Watcher retry storm: 6 files retried 144 times, hammering n8n -> Implemented per-file retry counter, backoff, and skip logic
- [resolved] French locale regex bug: .NET exception message string matching failed due to localized stems -> Use SocketErrorCode Win 10061 instead of string matching

## Patterns Identified
- PS 5.1 polling loops need per-file retry tracking to avoid retry storms
- LLMLingua needs input size guards before compression calls
- Windows locale-specific error strings break English regex matching - use error codes instead
- Docker container patching via docker cp + restart is faster than rebuilding for single-file Python changes

## Files Modified
- modified: D:\aidirectory\scripts\session-watcher.ps1 -- Added per-file retry tracking, BACKOFF/SKIP logic, locale-proof connection detection, removed pre-drain startup retry path
- modified: D:\aidirectory\workflow-lab\agents\api_with_langfuse.py -- Added MAX_INPUT_CHARS=100,000 guard to compress endpoint, truncate with warning log
- modified: D:\aidirectory\HANDOVER.md -- Updated CODE section with watcher fixes, SHARED section with LLMLingua guard info

## Next Session Must Know
- Watcher retry logic: per-file counter, BACKOFF after 3 fails, SKIP after backoff expires, PAUSE 30s before retries
- LLMLingua compress endpoint now has MAX_INPUT_CHARS=100,000 guard - truncates with warning, doesn't reject
- French locale regex bug fixed: use SocketErrorCode Win 10061 not string matching for 'connection refused'
- Container patched via docker cp + restart, not rebuild - verify gunicorn healthy
- 6 backlog files drained, cascade resolved - monitor pipeline health via n8n Executions tab
- Scheduled task pwsh path pinned to 7.6.2.0 WindowsApps Store install - machine-scoped MSI needed for stable Program Files location
- Duplicate LLMLingua source: D:\aidirectory\projects\workflow-lab vs D:\aidirectory\workflow-lab\agents - pick one, delete other
- Watcher SKIP state is in-memory only - persists across restarts, consider disk persistence for tight restart loops

## Skill Candidates
- docker-patch-single-file: Copy single file into running container and restart service without full rebuild
- ps-retry-backoff-manager: Add per-item retry tracking with exponential backoff and skip logic to PowerShell polling scripts

## Token Waste Flags
- Repeated HANDOVER.md update process multiple times
- Detailed retry logic explanation repeated in different contexts

## Knowledge Base Updates
- [update] sop_python_flask-api-llmlingua.md: Add MAX_INPUT_CHARS guard pattern for LLMLingua compress endpoint to prevent OOM crashes
- [create] runbook_infra_watcher-retry-management.md: Document per-file retry tracking, backoff, skip logic patterns for PowerShell polling scripts
- [update] ref_security_agent-framework.md: Add locale-proof error handling pattern: use error codes not string matching for .NET exceptions

## Links
related:: [[_INDEX]]
tags: infra, python, docker, n8n, memory
