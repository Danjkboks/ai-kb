---
type: extract
date: 2026-05-28
session_id: container-audit-dify-stack-cpu-runaway
surface: claude-code
environment: env2-workflow-lab
topics: [docker, infra, llm]
source_file: 2026-05-28-195838_d24090f6-5ba2-48d0-a51e-6975950d2a20.jsonl
processed_at: 2026-06-05T10:17:20.111Z
---

# Session Extract: container-audit-dify-stack-cpu-runaway

## Decisions
- **Audit and consider stopping the Dify stack due to resource conflict and overlap**: Dify stack consumes 1.5GB RAM, holds ports 80/443, and overlaps with n8n + OpenRouter + LLMLingua core stack; it's unused and blocks Cloudflare tunnel origin for future services.

## Problems Solved
- **Identified agent-llmlingua container at 1260% CPU (runaway process)**: Plan to check docker logs for agent-llmlingua (tail -100) to diagnose CPU spike.

## Errors Encountered
- [resolved] docker stats command interrupted with '760 error' -> Command was interrupted; no fix applied as it was a monitoring command.

## Patterns Identified
- Container resource monitoring via docker ps and docker stats reveals runaway processes.

## Files Modified
none

## Next Session Must Know
- Dify stack (10 containers, 1.5GB RAM) overlaps with n8n+OpenRouter+LLMLingua core stack and holds ports 80/443.
- agent-llmlingua container showing 1260% CPU runaway - need to check logs: `docker logs agent-llmlingua --tail 100`.
- Decision pending: Stop Dify stack to free resources and ports 80/443 for Cloudflare tunnel origin.
- Langfuse container is part of core stack for tracing OpenRouter calls; keep if traces are written.
- Dify stack located in docker compose folder; investigate before stopping.
- Core stack consists of n8n (5678), LLMLingua-2 (5001), Qdrant (6333), Langfuse (3000).
- Container audit performed: agent-llmlingua, langfuse, qdrant, n8n, nginx, plugin-daemon, worker, api, worker-beat, sandbox, web, db_postgres, redis, ssrf_proxy.
- Emergency context evacuation template referenced at C:\Users\GnReN-PC\mission.md.

## Skill Candidates
- container-audit-stack-conflict: Audit running Docker containers for resource usage, port conflicts, and functional overlap with core stack.

## Token Waste Flags
- Repeated full skill listing at session start without user request.

## Knowledge Base Updates
- [update] sop_docker_container-management.md: Add procedure for auditing container stacks for resource conflicts and overlap with core services.
- [create] audit_infra_dify-stack-overlap.md: Document the audit findings of Dify stack conflict with core workflow lab stack, including resource usage and port blocking.

## Links
related:: [[_INDEX]]
tags: docker, infra, llm
