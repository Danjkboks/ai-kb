---
type: extract
date: 2026-06-05
slug: llmlingua-oom-watcher-diagnosis
surface: chat
topics: [docker, session-watcher, llmlingua, pipeline, infra]
skill_candidates: []
agent_candidates: []
duration_min: 30
---

## What Happened
- Loaded HANDOVER.md, MEMORY.md, PLAN.md to restore full stack context
- Updated Cloudflare tunnel URL in HANDOVER.md (both header + SHARED section): https://approaches-organic-avi-finish.trycloudflare.com
- Checked Docker Desktop screenshot: all 10 containers running (n8n, agent-llmlingua, qdrant, langfuse + Dify stack)
- Spotted agent-llmlingua at 2300%+ CPU — investigated via docker logs
- Diagnosed full crash chain from watcher log + n8n Executions tab

## Root Cause Diagnosis

### Bug 1 — Watcher infinite retry storm
- session-watcher.ps1 started at 09:43:46 with 6 backlog .jsonl files in data/sessions/
- n8n was not yet up → connection refused on all attempts
- Watcher has NO backoff logic: re-detects same 6 files every 2s poll → ~144 failed attempts over 9 minutes
- Once n8n came up at 09:52, watcher dumped all 6 files simultaneously

### Bug 2 — Oversized raw JSONL → LLMLingua OOM
- The .jsonl files are raw Claude Code session exports, NOT pre-processed extracts
- File sizes: 65KB, 1343KB, 1343KB, 1453KB, 2301KB, 2509KB
- n8n async webhook accepted all 6 (200 OK each), queued all 6 for simultaneous processing
- LLMLingua received 4x multi-MB payloads at once → token sequences up to 1,437,294 >> 512 limit
- Gunicorn workers hit WORKER TIMEOUT (5min) → SIGKILL → crash loop → 2300% CPU

### Secondary bug — Files reappearing after move to done/
- 3 files (3d8df92b, fa1ddd54, 12b2c88a) were moved to done/ at 09:52:50-54
- Same files re-detected and re-moved at 09:53:42-49 (~50s later)
- Cause unknown: possibly n8n error handler restores file, or duplicate copy exists
- Not yet investigated

## Decisions Made
- Fix 1 (watcher): add per-file retry tracking + exponential backoff + skip after N failures — Claude Code task
- Fix 2 (LLMLingua): add input size guard in Flask app — truncate at 100K chars, log WARNING — Claude Code task
- Both fixes to be done in same Claude Code session with Kimi K2
- Watcher log .txt to be shared with Code as evidence artifact
- No screenshots needed for Code session (described in prompt)

## Errors Encountered
- LLMLingua WORKER TIMEOUT loop: ongoing at time of writing — self-resolved after ~14 min crash cycle, now idle
- n8n executions all Error in ~5min (async mask confirmed the fragility documented in HANDOVER.md)

## What Worked
- Docker Desktop provided clear CPU spike visibility
- Watcher log precisely traced the retry storm pattern
- Async webhook fragility (known fragility) played out exactly as documented

## What Didn't Work
- Watcher retry-on-every-poll: catastrophic when downstream is down
- Raw .jsonl files straight to LLMLingua: too large, no guard

## Suggested Improvements
- After Fix 1+2: consider adding n8n preprocessing step to extract only text content from JSONL before LLMLingua (reduces payload by ~80%)
- Add gunicorn --preload flag to load Wiki once at startup, not per-worker
- Investigate reappearing files bug in a separate session

## Files Modified
- D:\aidirectory\HANDOVER.md: updated Cloudflare URL (header + SHARED section)
- D:\aidirectory\data\queue\pending\EXTRACT_2026-06-05_llmlingua-oom-watcher-diagnosis.md: this file

## Next Session Should Know
- Fix 1 and Fix 2 are the active Claude Code tasks — see prompt prepared in Chat session
- Cloudflare URL: https://approaches-organic-avi-finish.trycloudflare.com (updated in HANDOVER.md)
- LLMLingua is currently idle/stable — crash loop self-resolved
- Watcher is currently running (started 09:43, processed 6 files at 09:52, log truncated at 09:53)
- 3-file reappear bug is NOT yet investigated — note for future session

## Knowledge Candidates
- SOP: session-watcher must implement per-file retry tracking with backoff — raw poll loop is unsafe
- Pattern: raw Claude Code .jsonl exports are too large for LLMLingua — always preprocess or guard input size
- Fragility confirmed: async webhook masks pipeline failures — n8n Executions tab is the only health signal
```
