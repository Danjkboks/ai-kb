---
type: extract
date: 2026-05-23
slug: phase2-n8n-pipeline
surface: claude-code
topics: [n8n, phase-2, knowledge-base, python, docker]
skill_candidates: [n8n-workflow-deploy, github-pat-diagnose]
agent_candidates: []
duration_min: 90
---

## What Happened
- Built Phase 2 of the persistent-memory system: an n8n pipeline that ingests Claude Code session transcripts and extracts structured knowledge.
- Created `session-watcher.ps1` (watches `data\sessions\`, POSTs new .jsonl to n8n webhook).
- Created the DeepSeek V3.2 system prompt (`knowledge/prompts/session-extractor-prompt.md`), hosted on GitHub raw and fetched by the workflow at runtime.
- Added a `/compress` endpoint to the agent-llmlingua Flask app and rebuilt the container.
- Created + activated the n8n workflow `session-knowledge-extractor` (13 nodes) via the public API.
- Verified end-to-end: a real extract was committed to `Danjkboks/ai-kb/extracts/`. Cost ~$0.0035/run.

## Decisions Made
- Fetch the system prompt from GitHub raw instead of inlining it: avoids escaping a 7KB string in JSON and keeps the prompt version-controlled.
- Use a dedicated HTTP Request node for HTTP, not the Code node: n8n 2.19.5 sandbox has no `fetch` and no `$helpers`.
- Reach LLMLingua from n8n via `host.docker.internal:5001` (separate Docker networks, container is port-published).

## Errors Encountered
- POST /workflows 400: `active` field rejected by public API | removed it, activate via separate endpoint.
- POST /activate 415: needs Content-Type + `{}` body | fixed.
- HTTP nodes sent empty body: wrong param `contentType` | use `specifyBody` (n8n v4.2).
- Code node `fetch`/`$helpers` not defined | use dedicated HTTP Request node.
- DeepSeek 400 "JSON parsing failed": `specifyBody: raw` mangled large body | switch to `specifyBody: json`.
- GitHub 401→403: PAT lacked Contents:write | new fine-grained PAT with Contents:Read-and-write.

## What Worked
- Debugging n8n via `GET /api/v1/executions/{id}?includeData=true` + string-search (PS 5.1 can't JSON-parse n8n's duplicate-key serialization).
- Direct GitHub API write-probe to isolate the PAT permission before re-running the slow pipeline.
- Reasoning to the 400 root cause instead of node-by-node bisection.

## What Didn't Work
- Building the workflow JSON with a giant PowerShell here-string containing JS — `$`/backtick interpolation produced a 0-char file. Hand-authored JSON via the Write tool was reliable.

## Files Modified
- `scripts/session-watcher.ps1`: created.
- `knowledge/prompts/session-extractor-prompt.md`: created.
- `D:\lab\workflow-lab\agents\api_with_langfuse.py`: added `/compress`.
- `.env`: added `GITHUB_PAT`.
- `data/proposals/workflow-session-extractor.json`: created (workflow source of truth).

## Next Session Should Know
- Workflow id `7l8aP0slan6tRkMy`, ACTIVE, webhook `http://localhost:5678/webhook/session-ingest`.
- `/compress` is baked into the agent-llmlingua image (not volume-mounted) — rebuild from `D:\lab\workflow-lab\agents\` if the container is recreated from an old image.
- Secrets (OpenRouter key, GitHub PAT) are inline in the workflow JSON — converting to n8n credentials is the recommended next hardening step.
- GitHub commit is create-only (no sha) — re-running the same session_id will 422.

## Knowledge Candidates
- New SOP: `sop_n8n_workflow-api-deploy.md` — quirks of the n8n public API (no `active` on POST, `specifyBody` not `contentType`, Code-node has no fetch/$helpers, activate needs Content-Type+`{}`).
- Update `sop_python_flask-api-llmlingua.md` to document the new `/compress` endpoint.
- Update `sop_docker_container-management.md`: agent-llmlingua run command needs `--network workflow-lab` and the wiki volume mount.
