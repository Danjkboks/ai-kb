---
type: extract
date: 2026-05-20
session_id: docker-crash-restart-workflow-lab
surface: claude-code
environment: env1-claude-desktop
topics: [docker, infra, n8n]
source_file: 2026-05-20-221515_59bc4de7-f57f-41e9-a86a-9e62bb002571.jsonl
processed_at: 2026-05-23T23:07:02.786Z
---

# Session Extract: docker-crash-restart-workflow-lab

## Decisions
- **Reset Docker to factory defaults when encountering socket path corruption**: Docker Desktop showed 'socket path error' and 'file accessed' corruption; resetting to factory defaults is safer than trying to troubleshoot specific corruption issues

## Problems Solved
- **Docker Desktop crashed with socket path corruption error preventing container startup**: Used MCP computer-use tools to request access to Docker Desktop, close error dialog, reset to factory defaults, then restart Docker daemon
- **Workflow lab services not starting due to Docker daemon being down**: After Docker reset, ran env-launch.ps1 script which successfully started langfuse-db, langfuse, n8n, qdrant, and agent-llmlingua containers

## Errors Encountered
- [resolved] Docker Desktop error: 'socket path error' and 'file accessed' corruption -> Reset Docker to factory defaults via Docker Desktop UI, then restart Docker daemon
- [pending] Cloudflare tunnel failed with 'No file cert.pem' error -> Cloudflare tunnel requires origin certificate configuration; tunnel exited but core services are running
- [resolved] python D:\lab\workflow-lab\claude_memory_save.py: [Errno 2] No such file or directory -> Script path was incorrect; used SESSION_SNAPSHOT.md file instead to check current state

## Patterns Identified
- Docker socket corruption on Windows requires factory reset via Docker Desktop UI
- env-launch.ps1 is the primary startup script for workflow lab services
- Cloudflare tunnel certificate issues persist across restarts and need manual configuration

## Files Modified
- read: D:\aidirectory\memory\SESSION_SNAPSHOT.md -- Read to check current stack status before attempting restart

## Next Session Must Know
- Docker was reset to factory defaults - any custom configurations are lost
- Cloudflare tunnel is still failing with cert.pem error - needs manual certificate configuration
- Core services (n8n, LLMLingua, Qdrant, Langfuse) are running on localhost ports 5678, 5001, 6333, 3000
- env-launch.ps1 at D:\aidirectory\workflow-lab\scripts\ is the primary startup script
- Memory snapshot was updated at 17:55:47 during startup process
- Docker Desktop MCP access was granted with 'full tier' permissions
- Workflow lab startup takes ~172 seconds (2m52s) for full container initialization

## Skill Candidates
- docker-factory-reset: Reset Docker Desktop to factory defaults when encountering socket corruption errors
- workflow-lab-start: Start workflow lab services using env-launch.ps1 script

## Token Waste Flags
- Repeatedly reading SESSION_SNAPSHOT.md without new information
- Multiple attempts to run python memory script with incorrect path

## Knowledge Base Updates
- [update] sop_docker_container-management.md: Add Windows-specific Docker socket corruption recovery procedure using factory reset
- [update] runbook_stack_master-build.md: Document Cloudflare tunnel certificate configuration requirement and persistent cert.pem error
- [create] ref_docker_windows-crash-recovery.md: Document specific procedure for Docker Desktop socket corruption on Windows 11 with factory reset steps
