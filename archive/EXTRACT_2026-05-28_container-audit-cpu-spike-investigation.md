---
type: extract
date: 2026-05-28
session_id: container-audit-cpu-spike-investigation
surface: claude-code
environment: env2-workflow-lab
topics: [docker, infra, llm]
source_file: 2026-06-05-095450_d24090f6-5ba2-48d0-a51e-6975950d2a20.jsonl
processed_at: 2026-06-05T10:18:43.765Z
---

# Session Extract: container-audit-cpu-spike-investigation

## Decisions
- **Stop Dify stack containers to free resources and ports**: Dify stack (10 containers, 1.5GB RAM) overlaps with n8n+OpenRouter+LLMLingua stack, holds ports 80/443 that block Cloudflare tunnel, and is unused in current workflow

## Problems Solved
- **agent-llmlingua container showing 1260% CPU usage (runaway process)**: Check docker logs to diagnose the CPU spike

## Errors Encountered
- [pending] agent-llmlingua container at 1260.39% CPU usage -> Run docker logs agent-llmlingua --tail 100 to investigate

## Patterns Identified
- Docker container audit pattern: check purpose, resource usage, and conflicts with core stack
- Port conflicts (80/443) block Cloudflare tunnel deployment

## Files Modified
none

## Next Session Must Know
- agent-llmlingua container has 1260% CPU spike - investigate with docker logs agent-llmlingua --tail 100
- Dify stack (10 containers, 1.5GB RAM) is unused and overlaps with n8n+OpenRouter+LLMLingua
- Dify's nginx container holds ports 80/443, blocking Cloudflare tunnel origin for future services
- Consider stopping Dify stack: docker-compose down in Dify folder to free ports and RAM
- Keep Langfuse if traces are being written to it
- Core stack confirmed: n8n (5678), LLMLingua (5001), Qdrant (6333), Langfuse (3000)
- Container audit completed: agent-llmlingua, n8n, qdrant, langfuse, docker-nginx, plugin-daemon, workers, db, redis, ssrf-proxy
- Emergency context shows 0% evacuation template at C:\Users\GnReN-PC\mission.md

## Skill Candidates
- docker-container-audit: Audit running containers for resource usage, purpose alignment, and conflicts with core stack

## Token Waste Flags
none

## Knowledge Base Updates
- [update] runbook_docker_container-management.md: Add container audit procedure and Dify stack removal steps
- [create] audit_container-resource-2026-05-28.md: Document CPU spike investigation and Dify stack conflict resolution

## Links
related:: [[_INDEX]]
tags: docker, infra, llm
