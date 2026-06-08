---
type: extract
date: 2026-05-28
session_id: phase1-factory-hardening-watcher-verification
surface: claude-code
environment: env1-claude-desktop
topics: [n8n, infra, memory, security, stack]
source_file: 2026-06-05-215921_6ebf402b-954d-43db-b56b-f31117e1cb4b.jsonl
processed_at: 2026-06-07T16:04:21.925Z
---

# Session Extract: phase1-factory-hardening-watcher-verification

## Decisions
- **Use version-pinned PowerShell 7.6.2.0 path for scheduled task to avoid breakage on upgrades**: Windows Store installs of pwsh can break scheduled tasks when upgraded; pinning to specific version ensures stability
- **Keep session-watcher.ps1 as primary integrity command (not n8n)**: PS7 script scheduled at logon provides reliable, surface-independent monitoring vs n8n workflow dependencies

## Problems Solved
- **Session watcher scheduled task needed verification and re-registration**: Verified task executes Program Files\WindowsApps\Microsoft.PowerShell_7.6.2.0_x64_8wekyb3d8bbwe\pwsh.exe with NoProfile, WindowStyle Hidden, ExecutionPolicy Bypass, triggered AtLogOn for user GnReN-PC, Limited privileges, Hidden, no time limit, runs on battery
- **Backlog of 26 session files needed draining through n8n pipeline**: Ran session-watcher.ps1 which processed 26 backlog files + 3 synthetic test sessions (all PASS, ~2s each)

## Errors Encountered
- [pending] GitHub commit node returns 422 when re-running same session ID (no idempotency guard) -> Pending idempotency fix needed; currently fails on re-run
- [pending] DeepSeek hallucinates extract date (model-driven not system clock) -> Need to inject system clock into extract prompt
- [workaround] Cloudflare tunnel URL changes on every restart, breaking n8n API calls -> n8n needs restart with fresh URL; knowledge/skills folder data queue pending EXTRACT files

## Patterns Identified
- Windows Store pwsh installs break scheduled tasks on upgrade - must pin version
- Session watcher logs show 200 responses but downstream failures (LLMLingua, DeepSeek, GitHub) need monitoring
- Phase 2 skill pipeline incomplete - knowledge/skills not emitting files
- n8n instance-specific credential IDs need reattachment after fresh n8n starts

## Files Modified
- modified: D:\aidirectory\HANDOVER.md -- Updated to reflect session-watcher verification, task configuration, backlog drain results, and current fragilities

## Next Session Must Know
- Session watcher verified PASS: 26 backlog + 3 test sessions drained
- Scheduled task path: Program Files\WindowsApps\Microsoft.PowerShell_7.6.2.0_x64_8wekyb3d8bbwe\pwsh.exe -NoProfile -WindowStyle Hidden -ExecutionPolicy Bypass -File D:\aidirectory\scripts\session-watcher.ps1
- Critical fragility: Cloudflare tunnel URL changes on restart - n8n API calls break until restart with fresh URL
- Phase 2 skill pipeline incomplete: knowledge/skills folder not emitting EXTRACT files
- GitHub commit node 422 error on re-run needs idempotency fix
- DeepSeek date hallucination: extract date is model-driven, not system clock
- Monitor n8n Executions tab for pipeline health: webhook 200s but downstream failures (LLMLingua, DeepSeek, GitHub)
- Helper scripts in scripts\: _register-watcher-task.ps1, _probe-webhook.ps1, _e2e-test.ps1, _inspect-task.ps1 (identify as session tooling, can delete)

## Skill Candidates
- n8n-pipeline-health-poller: Poll n8n Executions tab via REST API to monitor webhook 200s and downstream failures, alert on errors
- scheduled-task-verifier: Verify Windows scheduled task configuration, execution path, triggers, and test run

## Token Waste Flags
- Repeated reading of HANDOVER.md multiple times in same session
- Tool use for simple text replacements that could be batched

## Knowledge Base Updates
- [update] runbook_stack_master-build.md: Add session watcher verification procedure, scheduled task configuration, and pipeline health monitoring steps
- [update] sop_n8n_mcp-setup.md: Document Cloudflare tunnel URL change impact and n8n restart procedure
- [create] runbook_infra_pipeline-health.md: Document monitoring n8n executions, checking downstream services, and alerting on session processing failures

## Links
related:: [[_INDEX]]
tags: n8n, infra, memory, security, stack
