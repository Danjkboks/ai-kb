---
type: extract
date: 2026-05-20
session_id: workflow-lab-env-launch-test
surface: claude-code
environment: env2-workflow-lab
topics: [n8n, docker, infra, memory]
source_file: 2026-05-20-175050_7a4aed34-476e-4549-ad10-864367b3afa6.jsonl
processed_at: 2026-05-23T22:49:19.558Z
---

# Session Extract: workflow-lab-env-launch-test

## Decisions
- **Use parallel tool calls for env-launch skill (Bash script + memory reads) instead of sequential execution**: Script takes 2-30s and memory reads are free - doing serially wastes user's time

## Problems Solved
- **Cloudflare tunnel failed with 'No file cert specify origin certificate path' error**: Known failure mode - need to run cloudflared tunnel login to fix certificate issue

## Errors Encountered
- [workaround] Cloudflare tunnel exited with 'No file cert specify origin certificate path' -> Run cloudflared tunnel login in terminal to re-establish certificate

## Patterns Identified
- Parallel tool calls critical for env-launch skill: Bash script (2-30s) + memory reads (free) should not be serial
- Cloudflare tunnel certificate errors are recurring issue requiring manual cloudflared login

## Files Modified
- modified: workflow-lab/scripts/env-launch.ps1 -- Executed to launch Workflow Lab stack with Docker containers and health checks
- modified: memory/SESSION_SNAPSHOT.md -- Updated with fresh stack status table and tunnel URL after env-launch execution

## Next Session Must Know
- Cloudflare tunnel certificate error: 'No file cert specify origin certificate path' - run cloudflared tunnel login to fix
- Local services (n8n:5678, Langfuse:3000, Qdrant:6333, LLMLingua:5001) are usable despite tunnel failure
- env-launch skill uses parallel tool calls: Bash script (2-30s) + memory reads should not be serialized
- Workflow Lab stack status: 5 containers running, Qdrant has 2,061 workflows indexed, LLMLingua provides 43-46% token compression
- Script flags available: SkipTunnel (skip Cloudflare tunnel), HealthTimeoutSec 120 (raise health-check ceiling for cold starts)
- Docker Desktop auto-starts if not running (waits up to 60s), then starts 5 containers with per-container success recording
- Project context memory files: project_progress.md (milestones), project_stack_status.md (component status), MEMORY.md (general)

## Skill Candidates
- env-launch: Launch Workflow Lab environment (n8n, LLMLingua, Qdrant, Langfuse, Cloudflare tunnel) with parallel tool calls for script execution and memory reads

## Token Waste Flags
none

## Knowledge Base Updates
- [update] sop_docker_container-management.md: Add Cloudflare tunnel certificate error pattern and fix (cloudflared tunnel login)
- [update] runbook_stack_master-build.md: Document env-launch parallel execution pattern and common failure modes
