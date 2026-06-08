---
type: extract
date: 2026-05-24
slug: memory-rag-architecture-build
surface: chat
topics: [memory, knowledge-base, n8n, obsidian, phase-1, phase-2]
skill_candidates: [session-drain-pipeline, obsidian-graph-wikilink-setup]
agent_candidates: []
duration_min: 180
---

## What Happened
- Full audit of D:\aidirectory memory + RAG architecture -- identified 4 broken pipeline stages
- Clarified two environments: Env1=Claude Desktop, Env2=Workflow Lab (separate concerns)
- Designed and built Phase 1: directory structure, git init for knowledge\, Obsidian Git plugin, export-sessions.ps1 fix, /audit + /wrap + /handover slash commands, scheduled task via Cowork
- Created new private GitHub repo Danjkboks/ai-kb for knowledge base
- Designed Phase 2: n8n pipeline architecture, data vs knowledge separation principle
- Built Phase 2: session-watcher.ps1, session-knowledge-extractor n8n workflow (13 nodes), DeepSeek V3.2 extraction, GitHub API commit to ai-kb
- Verified end-to-end pipeline: .jsonl -> LLMLingua compress -> DeepSeek extract -> GitHub commit -> Obsidian sync
- Hardened credentials: OpenRouter + GitHub PAT moved from inline to n8n credential store
- Fixed Obsidian orphan nodes: _INDEX.md wikilinks updated, extracts now get ## Links dataview block
- Installed PowerShell 7 (pwsh.exe) -- fixes French Windows 11 encoding issues
- Manually drained 21 session queue via PowerShell loop script

## Decisions Made
- DeepSeek V3.2 over Kimi K2 for batch processing: output tokens ~9x cheaper, sufficient for extraction tasks
- data\ = local only, never git-tracked | knowledge\ = git-tracked -> ai-kb -> Obsidian
- GitHub as message bus between Claude Desktop and n8n: Obsidian Git plugin pushes, n8n reads via API
- PowerShell watcher POSTs file content to n8n webhook -- no Docker volume mount needed
- System prompt fetched from GitHub raw at runtime -- version-controlled, no JSON escaping pain
- Use pwsh.exe for all future scripts and scheduled tasks

## Errors Encountered
- export-sessions.ps1 wrong ProjectId (D--lab-Claudi): fixed to D--aidirectory-projects-claudi
- n8n API POST /workflows rejects active field: activate via separate endpoint
- n8n HTTP nodes empty body: wrong param contentType, correct is specifyBody
- Code node has no fetch/$helpers in n8n 2.19.5 sandbox: use dedicated HTTP Request node
- DeepSeek 400 JSON parsing failed: specifyBody raw mangles large body, use specifyBody json
- GitHub 401->403: PAT lacked Contents:write, fixed with fine-grained PAT
- Obsidian extracts orphaned: _INDEX.md had text refs not wikilinks, fixed

## What Worked
- Reading HANDOVER.md + MEMORY.md + CLAUDE.md before each Claude Code session prevented wrong-direction work
- Debugging n8n via GET /api/v1/executions/{id}?includeData=true
- Direct GitHub API write-probe to isolate PAT permission issues before re-running full pipeline
- Two-environment mental model kept scope clean throughout

## What Didn't Work
- Building n8n workflow JSON via PowerShell here-string with JS: $ interpolation corrupts file. Use Write tool with hand-authored JSON.
- Assuming Obsidian Git plugin was already installed and configured -- it wasn't
- Assuming knowledge\ was git-tracked -- it wasn't

## Suggested Improvements
- Schedule session-watcher.ps1 via Cowork (still pending)
- Manual drain script should include move-to-done logic to avoid confusion
- Add idempotency to GitHub commit node (get SHA before write to avoid 422 on re-run)

## Files Modified
- D:\aidirectory\data\ (new tree): created
- D:\aidirectory\knowledge\extracts\, skills\, prompts\, audits\: created
- D:\aidirectory\.gitignore: created
- D:\aidirectory\scripts\session-audit.ps1: created
- D:\aidirectory\scripts\session-watcher.ps1: created
- D:\aidirectory\.claude\commands\wrap.md: created
- D:\aidirectory\.claude\commands\audit.md: created
- D:\aidirectory\.claude\commands\handover.md: created
- D:\aidirectory\knowledge\_INDEX.md: updated (wikilinks + new sections)
- D:\aidirectory\knowledge\prompts\session-extractor-prompt.md: created
- D:\aidirectory\data\proposals\workflow-session-extractor.json: created
- D:\aidirectory\HANDOVER.md: updated
- D:\aidirectory\projects\workflow-lab\.claude\export-sessions.ps1: fixed ProjectId
- n8n workflow session-knowledge-extractor (id 7l8aP0slan6tRkMy): created + hardened

## Next Session Should Know
- Phase 1 + Phase 2 fully live as of 2026-05-24
- n8n workflow active at /webhook/session-ingest, credentials in n8n store (not inline)
- LLMLingua: http://host.docker.internal:5001/compress (n8n and agent-llmlingua on separate Docker networks)
- DeepSeek model string: deepseek/deepseek-v3.2
- Cost per session processed: ~$0.0035
- session-watcher.ps1 written but NOT yet scheduled (Cowork task pending)
- 21 sessions manually drained -- files remain in data\sessions\ (watcher won't re-process, event-based)
- data\queue\pending\ has 3 EXTRACT files from /wrap -- these feed Phase 3 (Qdrant), not current n8n pipeline
- PS7 installed (pwsh.exe) -- use for all new scripts and scheduled tasks
- French Windows 11 PS5.1 breaks on non-ASCII in .ps1 files -- ASCII only rule still applies to PS5.1 scripts
- GitHub commit node is create-only -- re-running same session_id returns 422, idempotency fix pending

## Knowledge Candidates
- New SOP: sop_n8n_workflow-api-deploy.md -- n8n public API quirks (no active on POST, specifyBody not contentType, no fetch in Code node, activate needs Content-Type+{})
- New SOP: sop_git_knowledge-base-setup.md -- initializing knowledge\ as standalone repo, Obsidian Git plugin config
- Update ref_llm_model-selection.md: DeepSeek V3.2 confirmed at $0.252/M input, $0.378/M output -- use for all batch/extraction tasks
- Add guide: data-vs-knowledge-separation.md -- data\ local only, knowledge\ git-tracked, why and how
