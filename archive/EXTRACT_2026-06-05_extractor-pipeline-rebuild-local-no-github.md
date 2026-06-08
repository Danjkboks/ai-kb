---
type: extract
date: 2026-06-05
session_id: extractor-pipeline-rebuild-local-no-github
surface: claude-code
environment: both
topics: [n8n, docker, memory, infra, python]
source_file: 2026-06-05-121554_12fe7480-aeda-4cc8-be44-a28788ac71cd.jsonl
processed_at: 2026-06-05T10:20:07.153Z
---

# Session Extract: extractor-pipeline-rebuild-local-no-github

## Decisions
- **Replace GitHub node with local Write Binary File node in n8n workflow 7l8aP0slan6tRkMy**: To eliminate GitHub dependency and enable local file writes to the knowledge vault via Docker volume mount
- **Use localhost:5678 for n8n MCP instead of Cloudflare tunnel URL**: Localhost is faster and survives tunnel URL rotation on restart
- **Add persistent deduplication to session-watcher.ps1 using sent-files.log**: Prevent duplicate processing of the same session files across multiple watcher processes
- **Set N8N_RESTRICT_FILE_ACCESS_TO=/data in Docker compose**: Required for n8n 2.23+ to allow file writes to mounted volume at /data/knowledge/extracts

## Problems Solved
- **GitHub 422 error when pushing extracts**: Removed GitHub Commit node and replaced with Write Binary File node writing to Docker volume mount
- **Date hallucination in DeepSeek extracts**: Modified OpenRouter HTTP Request node to prepend session date from webhook payload before sending to DeepSeek
- **Truncation discarding important decisions/resolutions at end of sessions**: Changed truncation strategy from first 100K to first 50K + last 50K (head+tail) to preserve end content
- **Duplicate file processing by multiple session-watcher instances**: Added sent-files.log persistence that loads on startup and appends after successful sends
- **PowerShell one-liner failures when called from Bash due to French locale**: Write .ps1 files to disk and invoke with -File instead of inline PowerShell via Bash

## Errors Encountered
- [resolved] n8n 2.23 restricts file access outside /data (Write Extract Disk exec #382) -> Added N8N_RESTRICT_FILE_ACCESS_TO=/data to Docker compose and mounted volume at /data/knowledge/extracts
- [resolved] PowerShell syntax errors when called from Bash due to locale mangling of operators -> Write PowerShell scripts to disk and invoke with -File parameter instead of inline execution
- [resolved] 32 historical queue sessions had no matching extracts (pre-fix failures) -> Created reconciliation script to identify missing extracts and moved backlog from queue to vault

## Patterns Identified
- PowerShell 5.1 with French locale breaks inline Bash execution - write to .ps1 files and invoke with -File
- n8n 2.23+ requires N8N_RESTRICT_FILE_ACCESS_TO environment variable for file system access
- Session watcher needs persistent deduplication across process restarts using disk-based log
- Localhost MCP connection survives Cloudflare tunnel URL rotation

## Files Modified
- modified: Vms/Dockers/N8N/docker-compose.yml -- Added volume mount for aidirectory/knowledge:/data/knowledge and N8N_RESTRICT_FILE_ACCESS_TO=/data environment variable
- modified: scripts/session-watcher.ps1 -- Added persistent deduplication using sent-files.log that loads on startup and appends after successful sends
- created: scripts/reconcile-extracts.ps1 -- New script to compare queue source files against knowledge extracts to identify missing extracts
- modified: claude/commands/wrap -- Updated target path from queue/pending to knowledge/extracts (Steps 1 & 3)
- modified: claude.json -- Registered n8n-instance MCP with localhost:5678 endpoint
- created: aidirectory/audits/AUDIT_2026-06-05_extractor-local-rebuild.md -- Session audit documenting the pipeline rebuild changes and verification

## Next Session Must Know
- Extractor pipeline is now fully local with no GitHub dependency - writes to /data/knowledge/extracts via Docker volume
- N8N_RESTRICT_FILE_ACCESS_TO=/data must be set in Docker compose for n8n 2.23+ file access
- PowerShell scripts must be written to disk and invoked with -File due to French locale mangling of operators in Bash
- 32 historical sessions were missing extracts (pre-fix failures) - backlog moved to vault, not a regression
- Session watcher uses sent-files.log for deduplication - loads on startup, appends after successful sends
- Truncation strategy changed to head+tail (first 50K + last 50K) to preserve decisions/resolutions at session end
- Date injection fix: OpenRouter node prepends session date from webhook payload to prevent DeepSeek hallucination
- n8n MCP registered against localhost:5678 (not Cloudflare tunnel) for reliability across tunnel rotations

## Skill Candidates
- n8n-workflow-update-batch: Atomic batch update of n8n workflows via MCP REST API with validation
- session-watcher-dedup: Persistent file deduplication for session watcher using disk-based sent log
- extract-reconciliation: Compare source queue files against generated extracts to identify pipeline failures

## Token Waste Flags
- Repeated PowerShell syntax debugging due to locale issues
- Multiple attempts to inspect workflow via MCP when REST API wasn't available
- Generated temporary .ps1 files for testing that were immediately cleaned up

## Knowledge Base Updates
- [update] sop_n8n_mcp-setup.md: Add localhost MCP configuration and workflow update patterns for atomic batch operations
- [update] sop_docker_container-management.md: Document N8N_RESTRICT_FILE_ACCESS_TO requirement and volume mount configuration for n8n 2.23+
- [create] runbook_n8n_extractor-pipeline-local.md: Complete runbook for local extractor pipeline with volume mounts, deduplication, and reconciliation
- [create] ref_powershell_locale-issues.md: Document PowerShell 5.1 French locale issues with Bash and workaround using -File parameter

## Links
related:: [[_INDEX]]
tags: n8n, docker, memory, infra, python
