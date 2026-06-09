---
type: extract
date: 2026-05-28
session_id: phase1-factory-hardening-watcher-verification
surface: claude-code
environment: env1-claude-desktop
topics: [n8n, infra, memory, python]
source_file: 2026-06-05-124736_6ebf402b-954d-43db-b56b-f31117e1cb4b.jsonl
processed_at: 2026-06-09T07:47:06.246Z
---

# Session Extract: phase1-factory-hardening-watcher-verification

## Decisions
- **Updated HANDOVER.md to use 3-section architecture (CHAT/COWORK/CODE + SHARED) with merge-overwrite rule enforced**: To provide clear separation of concerns and ensure Chat surface writes directly via Filesystem connector
- **Kept session-watcher.ps1 scheduled task with version-pinned pwsh path (7.6.2.0)**: WindowsApps Store install can break on pwsh upgrade, so version pinning ensures stability

## Problems Solved
- **HANDOVER.md was outdated with 2026-05-24 information**: Updated HANDOVER.md with current state including session-watcher verification results and pipeline status
- **Session watcher task needed verification after setup**: Verified 26 backlog files drained and 3/3 synthetic test sessions PASS (~2s each)

## Errors Encountered
- [pending] GitHub commit node returns 422 when re-running session -> Need idempotency fix - currently no guard against re-running same session
- [pending] DeepSeek hallucinates extract date (model-driven not system clock) -> Need to inject system clock into extract prompt
- [pending] Cloudflare tunnel URL changes on restart, breaking n8n API calls -> n8n needs to restart with fresh URL

## Patterns Identified
- WindowsApps Store pwsh install can break on upgrade - version pinning required
- Session knowledge extractor webhook async pipeline failures have no surface watcher - need logs monitoring
- Phase 2 skill pipeline incomplete - knowledge skills not emitting files

## Files Modified
- modified: aidirectory/HANDOVER.md -- Updated with current session-watcher status, pipeline health, and fragilities section

## Next Session Must Know
- Session-watcher.ps1 verified PASS: 26 backlog drained, 3 test replays
- Knowledge extractor webhook responds 200 but async pipeline failures have no surface watcher
- GitHub commit node returns 422 on re-run - needs idempotency fix
- DeepSeek hallucinates extract date - need system clock injection
- Cloudflare tunnel URL changes break n8n API - needs restart with fresh URL
- Phase 2 skill pipeline incomplete - knowledge skills not emitting files
- Scheduled task pwsh path pinned to 7.6.2.0 in WindowsApps to prevent upgrade breaks
- Helper scripts left in scripts/: _register-watcher-task.ps1, _probe-webhook.ps1, _e2e-test.ps1, _inspect-task.ps1 - identify as session tooling

## Skill Candidates
- handover-md-updater: Updates HANDOVER.md with current pipeline state, fragilities, and next tasks
- n8n-pipeline-health-check: Checks n8n Executions tab, webhook watcher 200s, downstream failures via REST API

## Token Waste Flags
- Repeated reading of HANDOVER.md multiple times
- Detailed listing of all MCP tools and skills at session start

## Knowledge Base Updates
- [update] sop_infra_session-watcher.md: Add verification results: 26 backlog drained, 3 test sessions PASS, pwsh version pinning requirement
- [update] runbook_n8n_pipeline-health.md: Add monitoring steps: check n8n Executions tab, webhook watcher 200s, downstream failures audit script
- [create] ref_infra_fragilities.md: Document known fragilities: GitHub 422 idempotency, DeepSeek date hallucination, Cloudflare URL changes, Phase 2 pipeline incomplete

## Links
related:: [[_INDEX]]
tags: n8n, infra, memory, python
