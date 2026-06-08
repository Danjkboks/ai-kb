---
type: extract
date: 2026-05-28
session_id: container-audit-llmlingua-cpu-spike
surface: claude-code
environment: env2-workflow-lab
topics: [docker, infra, llm]
source_file: 2026-06-05-095450_d24090f6-5ba2-48d0-a51e-6975950d2a20.jsonl
processed_at: 2026-06-05T09:20:20.962Z
---

# Session Extract: container-audit-llmlingua-cpu-spike

## Decisions
- **Audit running containers to identify resource hogs and overlaps with core stack**: Observed high CPU usage (1260%) on agent-llmlingua container; need to verify core stack alignment and free up resources/ports

## Problems Solved
- **Unidentified high CPU consumption (1260%) on a container**: Used docker stats to identify agent-llmlingua container as the runaway process

## Errors Encountered
- [resolved] docker stats command interrupted (exit code 130) before completion -> Command was manually interrupted; data was partially captured

## Patterns Identified
- Dify stack (10 containers, ~1.5GB RAM) overlaps with core n8n+OpenRouter+LLMLingua stack, consuming ports 80/443
- Container audit pattern: docker ps for inventory, docker stats for resource usage, docker logs for diagnostics

## Files Modified
none

## Next Session Must Know
- agent-llmlingua container at 1260% CPU - investigate with docker logs agent-llmlingua --tail 100
- Dify stack (10 containers) uses ports 80/443, blocking Cloudflare tunnel; consider stopping if unused
- Total RAM usage: ~1.5GB across all containers; Dify stack may overlap with n8n+OpenRouter+LLMLingua
- Check if Langfuse is actively writing traces before deciding to stop it
- Container inventory shows: n8n, langfuse, qdrant, docker-nginx, dify stack (api, worker, web, db, redis, etc.)
- Dify folder location needs investigation to understand its docker-compose setup
- Core stack should be: n8n (5678), LLMLingua (5001), Qdrant (6333), Langfuse (3000), OpenRouter gateway
- Emergency context evacuation template referenced at C:\Users\GnReN-PC\mission.md

## Skill Candidates
- container-audit: Audits running Docker containers for resource usage, port conflicts, and stack alignment

## Token Waste Flags
- Repeated full container listing in tool result output
- Raw session metadata included in transcript without filtering

## Knowledge Base Updates
- [update] runbook_stack_master-build.md: Add container audit procedure and Dify stack conflict resolution
- [create] sop_docker_container-audit.md: Document standardized container auditing process for resource monitoring and conflict detection

## Links
related:: [[_INDEX]]
tags: docker, infra, llm
