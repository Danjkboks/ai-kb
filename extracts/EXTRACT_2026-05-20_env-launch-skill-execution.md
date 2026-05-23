---
type: extract
date: 2026-05-20
session_id: env-launch-skill-execution
surface: claude-code
environment: env1-claude-desktop
topics: [n8n, docker, infra, memory]
source_file: 2026-05-20-134142_7a4aed34-476e-4549-ad10-864367b3afa6.jsonl
processed_at: 2026-05-23T19:09:39.976Z
---

# Session Extract: env-launch-skill-execution

## Decisions
- **Use parallel tool calls for env-launch skill (Bash script + memory reads) instead of sequential execution**: Script takes 2-30s, memory reads are free; doing serially wastes user's time

## Problems Solved
- **Cloudflare tunnel failed to start due to missing origin certificate**: Tunnel exited with error 'No file cert specify origin certificate path'. Known failure mode - requires running cloudflared tunnel login in terminal and re-invoking skill

## Errors Encountered
- [workaround] Cloudflare tunnel exited 2026-05-20T11:26:10Z with error: 'No file cert specify origin certificate path' -> Run cloudflared tunnel login in terminal to fix certificate, then re-invoke skill

## Patterns Identified
- Parallel tool calls critical for env-launch skill to avoid serial time waste
- Cloudflare tunnel certificate error is a known failure mode with documented recovery steps

## Files Modified
- modified: workflow-lab/scripts/env-launch.ps1 -- Executed to launch Workflow Lab stack (Docker containers + health checks)
- modified: memory/SESSION_SNAPSHOT.md -- Updated with fresh stack status table and tunnel URL (though tunnel failed)

## Next Session Must Know
- Cloudflare tunnel failed due to missing origin certificate - run 'cloudflared tunnel login' in terminal to fix
- Local services (n8n:5678, LLMLingua:5001, Qdrant:6333, Langfuse:3000) are usable despite tunnel failure
- env-launch.ps1 script supports SkipTunnel flag for when certificate is broken
- Stack status: agent-llmlingua, langfuse, qdrant, n8n all healthy (langfuse-db also running)
- Total launch time: 20.1s for container startup and health checks
- Project context shows Phase 5 (Optimization Stack) completed 2026-05-19 with prompt caching, Qdrant indexing, Langfuse deployment
- Workflow Indexation System has 2,061 workflows indexed with 4 phases complete
- Budget constraint: 50€/month, OpenRouter only LLM gateway

## Skill Candidates
- env-launch: Launches Workflow Lab stack (Docker n8n, Qdrant, Langfuse, LLMLingua, Cloudflare tunnel) with health checks and memory refresh

## Token Waste Flags
none

## Knowledge Base Updates
- [update] runbook_stack_master-build.md: Add Cloudflare tunnel certificate error as known failure mode with recovery steps
- [update] sop_docker_container-management.md: Document env-launch.ps1 script behavior, SkipTunnel flag, and parallel execution pattern
