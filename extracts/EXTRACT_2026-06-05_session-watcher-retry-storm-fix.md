---
type: extract
date: 2026-06-05
session_id: session-watcher-retry-storm-fix
surface: claude-code
environment: env1-claude-desktop
topics: [n8n, python, infra, memory]
source_file: 2026-06-05-105002_dbd9ecf2-b5aa-45c7-b90d-9e7ca00ff517.jsonl
processed_at: 2026-06-05T08:56:06.525Z
---

# Session Extract: session-watcher-retry-storm-fix

## Decisions
- **Implement per-file retry tracking with backoff in session-watcher.ps1**: To prevent retry storms where failed files get re-detected and hammer n8n, causing LLMLingua crashes
- **Add input size guard to LLMLingua Flask app before compression**: Large session files (2.5MB+) cause LLMLingua to crash with 1.4M token sequences exceeding 512 limit

## Problems Solved
- **Retry storm: 6 backlog JSONL files detected every 2s poll, n8n hammered 144 times in minutes, causing LLMLingua crash loop at 2300% CPU**: Add per-file retry tracking with 3 attempts → 5min backoff → mark SKIPPED after backoff fails
- **LLMLingua Flask app crashes on large session files (2.5MB+) exceeding token limit**: Add MAX_INPUT_CHARS = 100,000 guard before LLMLingua compression, truncate or reject with 4xx

## Errors Encountered
- [resolved] LLMLingua crash loop with 2300% CPU due to 1,437,294 token sequences exceeding 512 limit -> Add input size validation before compression call
- [resolved] n8n webhook hammered 144 times for 6 files due to retry storm -> Implement per-file retry tracking with exponential backoff

## Patterns Identified
- PS 5.1 event action blocks lose variable scope - polling is more reliable
- Large session exports (>2.5MB) can overwhelm LLMLingua token limits
- File watcher needs per-file state tracking to avoid infinite retry loops
- ASCII strings required in PS1 files due to Windows-1252 encoding issues

## Files Modified
- modified: aidirectory/scripts/session-watcher.ps1 -- Added per-file retry tracking hashtable, backoff logic (3 attempts → 5min backoff → SKIP), and ASCII-safe logging
- modified: workflow-lab/agents/llmlingua/app.py -- Added MAX_INPUT_CHARS = 100,000 guard before LLMLingua compression to prevent crash on large inputs

## Next Session Must Know
- session-watcher.ps1 now has retry tracking: 3 attempts → 5min backoff → SKIP after backoff fails
- LLMLingua Flask app has MAX_INPUT_CHARS = 100,000 guard to prevent crash on large session exports
- ASCII strings required in all PS1 files due to Windows-1252 encoding
- Backlog of 6 JSONL files caused retry storm - check aidirectory/data/sessions for stuck files
- Log format: [BACKOFF] filename attempt 3/3 waiting 5min, [SKIP] filename, [RETRY] backoff expired retrying
- Wait 30s before ANY connection attempt after backoff to prevent hammering
- Flask app source at workflow-lab/agents/llmlingua/app.py - need to restart container after edit
- Check Dockerfile build context at aidirectory/workflow-lab for agent source location

## Skill Candidates
- file-watcher-retry-pattern: Implements per-file retry tracking with exponential backoff for file processing pipelines

## Token Waste Flags
- Repeated same file content in transcript multiple times
- Excessive logging details in handover file could be summarized

## Knowledge Base Updates
- [update] runbook_stack_master-build.md: Add session watcher retry pattern and LLMLingua input size guard to prevent crash loops
- [update] sop_python_flask-api-llmlingua.md: Document MAX_INPUT_CHARS guard and restart procedure after modifying Flask app
- [create] ref_infra_file-watcher-patterns.md: Document per-file retry tracking with backoff pattern for reliable file processing

## Links
related:: [[_INDEX]]
tags: n8n, python, infra, memory
