---
type: extract
date: 2026-05-28
session_id: container-audit-dify-stack-cpu-spike
surface: claude-code
environment: env1-claude-desktop
topics: [docker, infra, llm]
source_file: 2026-06-05-095450_d24090f6-5ba2-48d0-a51e-6975950d2a20.jsonl
processed_at: 2026-06-05T08:55:20.943Z
---

# Session Extract: container-audit-dify-stack-cpu-spike

## Decisions
- **Identify Dify stack as non-core and candidate for shutdown**: Dify (10 containers, 1.5GB RAM) overlaps with n8n + OpenRouter + LLMLingua core stack, holds ports 80/443 blocking Cloudflare tunnel, and is unused.

## Problems Solved
- **agent-llmlingua container showing 1260% CPU usage (runaway)**: Identified via docker stats; next step is to check logs with docker logs agent-llmlingua --tail 100

## Errors Encountered
- [resolved] docker stats command interrupted with '760 error' -> Command was manually interrupted; no fix needed.

## Patterns Identified
- Container audit pattern: check docker ps for inventory, docker stats for resource usage, docker logs for diagnostics

## Files Modified
none

## Next Session Must Know
- Dify stack (10 containers, 1.5GB RAM) is non-core, overlaps with n8n+OpenRouter+LLMLingua, holds ports 80/443 blocking Cloudflare tunnel.
- agent-llmlingua container has 1260% CPU spike - check logs with docker logs agent-llmlingua --tail 100.
- Core stack: n8n (5678), LLMLingua (5001), Qdrant (6333), Langfuse (3000).
- Dify containers: docker-api, worker, worker-beat, web, sandbox, plugin-daemon, db_postgres, redis, nginx, ssrf_proxy.
- Decision pending: stop Dify stack to free ports 80/443 and reclaim 1.5GB RAM.
- Langfuse may need to be kept if traces are being written.
- Container audit performed via docker ps and docker stats.
- Path: D:\aidirectory

## Skill Candidates
- container-audit: Audits running containers: lists inventory, checks resource usage, identifies core vs non-core stacks, and logs diagnostics.

## Token Waste Flags
none

## Knowledge Base Updates
- [update] sop_docker_container-management.md: Add container audit procedure (ps, stats, logs) and decision framework for identifying non-core stacks.
- [create] audit_infra_dify-stack-assessment.md: Document findings on Dify stack overlap, resource usage, and shutdown rationale.

## Links
related:: [[_INDEX]]
tags: docker, infra, llm
