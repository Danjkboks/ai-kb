---
type: audit
date: 2026-05-23
session_id: phase2-n8n-pipeline
surface: claude-code
duration_min: 90
---

## What Was Built / Changed
- `scripts/session-watcher.ps1` — poll-based watcher (2s) for `data\sessions\*.jsonl`; base64-encodes new files, POSTs to n8n webhook, moves to `queue\done\` on 200, logs failures to `data\audits\watcher.log`. ASCII-only, parse-clean.
- `knowledge/prompts/session-extractor-prompt.md` — DeepSeek V3.2 system prompt with full env context + 13-field JSON extraction schema. Hosted on GitHub raw (Obsidian Git auto-sync) and fetched by the workflow at runtime.
- Added `/compress` endpoint to `D:\lab\workflow-lab\agents\api_with_langfuse.py` (LLMLingua-2 standalone compression); rebuilt + recreated the `agent-llmlingua` container.
- n8n workflow `session-knowledge-extractor` (id `7l8aP0slan6tRkMy`, 13 nodes, ACTIVE) created via public API. Saved to `data\proposals\workflow-session-extractor.json`.
- Added `GITHUB_PAT` to `D:\aidirectory\.env`.
- Verified end-to-end: real extract committed to `Danjkboks/ai-kb/extracts/EXTRACT_2026-05-20_env-launch-skill-execution.md`.

## Decisions Made
- Fetch system prompt from GitHub raw inside the workflow (not inline in the Code node): avoids 7KB JSON-escaping hell and makes the prompt version-controlled/updatable without touching the workflow.
- Used a dedicated HTTP Request node ("Fetch System Prompt") instead of fetching inside the Code node: n8n 2.19.5 Code sandbox exposes neither `fetch` nor `$helpers`.
- LLMLingua reached from n8n via `http://host.docker.internal:5001` (n8n on `n8n_default`, agent-llmlingua on `workflow-lab` — separate Docker networks; container is port-published to host).
- Added a real `/compress` endpoint rather than reusing `/ask` (which bundles compression + an LLM call): we need compression only.
- PAT wired inline in the workflow (matches the existing inline OpenRouter key); stored canonically in `.env`. Tradeoff noted: proper fix is an n8n httpHeaderAuth credential.

## Errors Encountered
- POST /workflows returned 400: top-level `active` field is rejected by the public API schema (read-only) | fixed: removed `active`, activate via separate endpoint.
- POST /activate returned 415: needs `Content-Type: application/json` + `{}` body | fixed.
- LLMLingua node sent body `{"":""}`: used wrong param name `contentType` instead of `specifyBody` (n8n httpRequest v4.2) | fixed across all HTTP nodes.
- Code node `fetch is not defined`, then `$helpers is not defined`: n8n 2.19.5 sandbox locks both | fixed by adding dedicated HTTP Request fetch node.
- DeepSeek 400 "JSON parsing failed": `specifyBody: raw` mangled the large body | fixed by switching to `specifyBody: json`.
- GitHub 401 then 403 "Resource not accessible by personal access token": first PAT lacked Contents:write | fixed: user generated new fine-grained PAT with Contents:Read-and-write on ai-kb.
- `/compress` initially 404: the agent-llmlingua container is a wiki Q&A app (`/ask`, `/health`, `/cache-stats`), not a raw compression API | fixed by adding the endpoint + rebuild.

## Token Usage (estimate)
- Input: ~120K | Output: ~25K | Compression: N (LLMLingua used inside the pipeline, not on this session)

## What Worked
- Diagnosing n8n failures via `GET /api/v1/executions/{id}?includeData=true` + string-search (PS 5.1 `ConvertFrom-Json` chokes on n8n's duplicate-key execution serialization).
- Reasoning about the 400 root cause (the `active` field) instead of slow node-by-node bisection.
- Direct GitHub API write-probe to isolate the PAT permission issue before re-running the full 46s pipeline.
- Background tasks for the long (45-50s) webhook calls so the session stayed responsive.

## What Didn't Work
- Building the workflow JSON via a giant PowerShell here-string with embedded JS: `$`/backtick interpolation produced a 0-char file. The Write tool with a hand-authored JSON file was far more reliable.
- Guessing n8n Code-node HTTP helper names — two wasted iterations. Lesson: use a dedicated HTTP Request node for HTTP in n8n, not the Code node.

## Suggested Improvements
- Convert the inline OpenRouter key + GitHub PAT to proper n8n credentials (httpHeaderAuth / generic credential) so secrets aren't stored in the workflow JSON.
- Add an error-branch / "Respond 500" path so webhook callers get a real failure status instead of an empty 200 when an upstream node errors.
- GitHub commit uses create-only semantics (no `sha`); a re-run with the same `session_id` will 422. Add a get-sha-then-update step if idempotent re-processing is needed.
- Wire `session-watcher.ps1` into a scheduled task or run-on-login so ingestion is automatic; consider Qdrant re-index + weekly synthesis (remaining Phase 2 Preview items 6-7).

## Files Modified
- `scripts/session-watcher.ps1`: created — the FileSystemWatcher/poll ingestion script.
- `knowledge/prompts/session-extractor-prompt.md`: created — DeepSeek system prompt.
- `D:\lab\workflow-lab\agents\api_with_langfuse.py`: added `/compress` Flask endpoint.
- `D:\aidirectory\.env`: added `GITHUB_PAT`.
- `data/proposals/workflow-session-extractor.json`: created — the n8n workflow definition (source of truth for re-import).

## Next Session Should Know
- n8n workflow `session-knowledge-extractor` = id `7l8aP0slan6tRkMy`, ACTIVE. Webhook: `http://localhost:5678/webhook/session-ingest` (tunnel path same, but tunnel URL changes per restart).
- LLMLingua `/compress` endpoint now exists but lives in the IMAGE (baked, not volume-mounted). If the container is recreated from an OLD image it loses `/compress` — rebuild from `D:\lab\workflow-lab\agents\` (`docker build -t agent-llmlingua:latest .` then run with `--network workflow-lab -v D:/lab/workflow-lab/wiki:/wiki --env-file .env`).
- n8n public API: never send `active` on POST; activate via `POST /workflows/{id}/activate` with `{}` + JSON content-type. For HTTP nodes use `specifyBody` (json/raw), NOT `contentType`.
- n8n Code nodes (2.19.5) have no `fetch` and no `$helpers` — do HTTP via a dedicated HTTP Request node.
- GitHub PAT (fine-grained, Contents:RW on ai-kb) is in `.env` as `GITHUB_PAT` and inline in the workflow. Secrets are still inline in the workflow JSON — converting to n8n credentials is the recommended next hardening step.
- A stray orphan container `upbeat_mestorf` (langfuse/langfuse:2, no DB env) keeps exiting — unrelated to the stack; safe to delete.
- agent-llmlingua shows periodic gunicorn WORKER TIMEOUT/SIGKILL under load (CPU LLMLingua-2). Compression of a 63KB file worked but watch memory if processing many/large sessions.
- Pipeline cost ~$0.0035/run via `deepseek/deepseek-v3.2` (provider SiliconFlow resolved `deepseek-v3.2-20251201`).
