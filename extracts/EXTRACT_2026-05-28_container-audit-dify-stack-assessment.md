---
type: extract
date: 2026-05-28
session_id: container-audit-dify-stack-assessment
surface: claude-code
environment: env2-workflow-lab
topics: [docker, infra, llm]
source_file: 2026-05-28-204939_d24090f6-5ba2-48d0-a51e-6975950d2a20.jsonl
processed_at: 2026-06-05T07:53:09.140Z
---

# Session Extract: container-audit-dify-stack-assessment

## Decisions
- **Identify Dify stack as unused and competing with n8n+OpenRouter+LLMLingua core stack**: Dify overlaps with n8n functionality, consumes significant resources (1.5GB RAM, 10 containers), and holds ports 80/443 that block Cloudflare tunnel for future services.

## Problems Solved
- **agent-llmlingua container showing 1260% CPU usage (runaway process)**: Plan to check docker logs for agent-llmlingua to investigate CPU spike.

## Errors Encountered
- [pending] agent-llmlingua container at 1260% CPU -> Check docker logs agent-llmlingua --tail 100

## Patterns Identified
- Unused Dify stack consuming resources and conflicting with core workflow stack

## Files Modified
none

## Next Session Must Know
- Dify stack (10 containers, 1.5GB RAM) identified as unused and competing with n8n+OpenRouter+LLMLingua core stack
- Dify holds ports 80/443 blocking Cloudflare tunnel origin for future services
- agent-llmlingua container showing 1260% CPU - need to check logs
- Decision pending: shut down Dify stack to free resources and ports
- Langfuse traces may be written to Dify stack - verify before shutdown
- Check docker logs agent-llmlingua --tail 100 to investigate CPU spike
- Dify docker-compose folder location needs investigation
- Core stack components: n8n (5678), LLMLingua (5001), Qdrant (6333), Langfuse (3000)

## Skill Candidates
none

## Token Waste Flags
none

## Knowledge Base Updates
- [update] sop_docker_container-management.md: Add procedure for auditing and shutting down unused stacks like Dify that conflict with core services.
- [create] audit_infra_dify-stack-assessment.md: Document the discovery of unused Dify stack consuming resources and conflicting with core workflow stack.

## Links
related:: [[_INDEX]]
tags: docker, infra, llm
