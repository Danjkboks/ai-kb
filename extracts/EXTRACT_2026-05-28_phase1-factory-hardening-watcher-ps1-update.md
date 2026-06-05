---
type: extract
date: 2026-05-28
session_id: phase1-factory-hardening-watcher-ps1-update
surface: claude-code
environment: env1-claude-desktop
topics: [n8n, infra, memory, python, docker]
source_file: 2026-06-05-095450_6ebf402b-954d-43db-b56b-f31117e1cb4b.jsonl
processed_at: 2026-06-05T10:19:04.834Z
---

# Session Extract: phase1-factory-hardening-watcher-ps1-update

## Decisions
- **Updated HANDOVER.md to use 3-section architecture (CHAT/COWORK/SHARED) with merge-overwrite rule enforced**: To maintain consistent project state tracking across different Claude surfaces and ensure Chat writes directly via Filesystem connector
- **Deferred n8n implementation of Integrity Agent in favor of Claude Code integrity command + PS7 script**: Simpler implementation path using existing tools rather than building new n8n workflow

## Problems Solved
- **Session watcher task not properly configured with correct PowerShell path**: Updated scheduled task to use version-pinned pwsh.exe path: 'C:\Program Files\WindowsApps\Microsoft.PowerShell_7.6.2.0_x64_8wekyb3d8bbwe\pwsh.exe' with NoProfile, WindowStyle Hidden, ExecutionPolicy Bypass flags
- **HANDOVER.md contained outdated information about session watcher status**: Updated HANDOVER.md to reflect current state: 26 backlog files drained, 3 synthetic test sessions PASS (~2s each), watcher verified working

## Errors Encountered
- [pending] GitHub commit node returns 422 on re-running session id (no idempotency guard) -> Proposed workflow session extractor instance-specific credential IDs reattach fresh n8n
- [pending] DeepSeek hallucinates extract date (model-driven not system clock) -> Need to inject system clock into extract prompt
- [workaround] Cloudflare tunnel URL changes on n8n restart requiring fresh URL API calls -> Documented as fragility - requires manual update when tunnel restarts

## Patterns Identified
- Phase 2 skill pipeline incomplete - knowledge skills not emitting EXTRACT files
- Scheduled task pwsh path version-pinned to WindowsApps Store install breaks on pwsh upgrade
- Session knowledge extractor webhook async pipeline failures have no surface watcher - only logs show 200s

## Files Modified
- modified: D:\aidirectory\HANDOVER.md -- Updated session watcher status, PowerShell path, and pipeline health monitoring details

## Next Session Must Know
- Session watcher task now uses: 'C:\Program Files\WindowsApps\Microsoft.PowerShell_7.6.2.0_x64_8wekyb3d8bbwe\pwsh.exe' -NoProfile -WindowStyle Hidden -ExecutionPolicy Bypass -File D:\aidirectory\scripts\session-watcher.ps1
- Watcher verified PASS: 26 backlog files drained + 3 synthetic test sessions
- Phase 2 skill pipeline incomplete - knowledge skills not emitting EXTRACT files to D:\aidirectory\knowledge\extracts\
- GitHub commit node returns 422 on re-running session id - needs idempotency fix
- DeepSeek hallucinates extract date - need to inject system clock into extract prompt
- Cloudflare tunnel URL changes on restart - requires manual update in n8n webhook calls
- Monitor pipeline health via n8n Executions tab: webhook watcher should show 200s, check downstream LLMLingua/DeepSeek/GitHub errors
- Helper scripts left in D:\aidirectory\scripts\: _register-watcher-task.ps1, _probe-webhook.ps1, _e2e-test.ps1, _inspect-task.ps1 (prefixed with _ for session tooling)

## Skill Candidates
- handover-md-maintainer: Updates HANDOVER.md with current project state, pipeline health, and tech debt tracking
- pipeline-health-check: Monitors n8n Executions tab for webhook watcher 200s and downstream failures

## Token Waste Flags
- Repeated reading of HANDOVER.md multiple times in same session
- Re-stating same pipeline fragilities multiple times

## Knowledge Base Updates
- [update] runbook_infra_session-watcher.md: Updated PowerShell path and scheduled task configuration for session-watcher.ps1
- [create] audit_n8n_pipeline-fragilities.md: Document known pipeline issues: GitHub 422 errors, DeepSeek date hallucination, Cloudflare URL changes, Phase 2 skill pipeline incomplete

## Links
related:: [[_INDEX]]
tags: n8n, infra, memory, python, docker
