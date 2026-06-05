---
type: extract
date: 2026-06-05
slug: watcher-retry-llmlingua-guard
surface: claude-code
topics: [n8n, infra, python, claude-code, phase-1]
skill_candidates: [audit-and-mocktest]
agent_candidates: []
duration_min: 45
---

## What Happened
- Diagnosed cascade failure at 09:43 today: 6 backlog .jsonl files in data\sessions\ + n8n down at watcher start caused ~144 retry POSTs over 9 minutes (no backoff). When n8n came up, all 6 dumped to LLMLingua async, token sequences up to 1,437,294 >> 512 limit, gunicorn workers timed out and SIGKILL-looped at 2300% CPU.
- Patched scripts\session-watcher.ps1: per-file failure counter, BACKOFF (5min) after 3 fails, SKIP after backoff retry, 30s global PAUSE on connection-refused; removed eager startup drain so existing files go through retry path.
- Patched D:\lab\workflow-lab\agents\api_with_langfuse.py /compress: MAX_INPUT_CHARS=100_000 truncate-not-reject guard with WARNING log. docker cp into agent-llmlingua + docker restart, gunicorn back up clean.
- Launched general-purpose audit agent with mock tests. Caught a critical bug: conn-refused regex only matched English; on this French Windows host the error string is "refusee" -> PAUSE branch never fired. Fixed by combining SocketErrorCode enum check + broader regex including French stem + Win error 10061.
- Confirmed LLMLingua 500-on-tokenless-input is a pre-existing library quirk only reachable with synthetic 'x'*250000 input, not real .jsonl payloads.

## Decisions Made
- Truncate, do not reject, oversize /compress input: partial compression beats OOM. Rationale: better to drop tail of a session than crash the pipeline.
- Watcher giving-up policy is per-run (in-memory hashtable), not persistent: SKIP state resets on restart. Rationale: scheduled task restart at next logon naturally retries skipped files.
- Conn-refused detection by both SocketErrorCode enum AND localized message regex. Rationale: enum is locale-proof; regex covers cases where exception chain doesn't expose SocketException directly.

## Errors Encountered
- Initial conn-refused regex `refused|actively refused|connection.*refused` missed French `refusee` -> 30s PAUSE never triggered on fr-FR host: fix applied (added SocketErrorCode check + `refus` stem + `10061`).
- docker exec with /app path got mangled by Git Bash MSYS path conversion: worked around using `sh -c "... //app/..."` (double slash).
- /compress crashes 500 on degenerate tokenless input (single repeated char): not fixed (pre-existing LLMLingua-2 percentile math, unreachable from real JSONL).

## What Worked
- Two-bug attack plan (watcher retry storm + LLMLingua OOM guard) addressed root causes, not symptoms. No "just restart n8n" band-aid.
- Audit agent with explicit mock test checklist caught the French-locale regex bug that static review missed. Worth keeping the pattern: fix -> launch agent with mock-test brief -> patch findings.
- docker cp + restart for in-place container patch (vs full rebuild) — fast feedback loop while host source D:\lab\workflow-lab\agents\ stays canonical for next rebuild.

## What Didn't Work
- Trusting English-only regex matching on French Windows. Caught only because audit was thorough.
- Assumption that D:\aidirectory\projects\workflow-lab\agents\ was the LLMLingua build context — it's actually D:\lab\workflow-lab\agents\. The former is an older copy (no /compress route).

## Suggested Improvements
- Sync D:\aidirectory\projects\workflow-lab\agents\api_with_langfuse.py with the canonical D:\lab\workflow-lab\agents\ version, or document which is build context (fragility candidate for HANDOVER).
- Wrap LLMLingua compressor.compress_prompt in try/except returning truncated raw text on failure — closes the tokenless-input 5xx hole (low priority; not reachable from real JSONL).
- Add an n8n Executions audit script (already in HANDOVER backlog) — async webhook hides downstream failures from watcher.
- Consider persisting watcher SKIP state to disk so a tight restart loop doesn't keep rehammering the same dead file.

## Files Modified
- D:\aidirectory\scripts\session-watcher.ps1: replaced polling loop with retry/backoff/skip logic; removed eager startup drain; conn-refused detection now locale-proof (SocketErrorCode + multilingual regex).
- D:\lab\workflow-lab\agents\api_with_langfuse.py: /compress now truncates text to 100_000 chars with WARNING log before LLMLingua call.
- agent-llmlingua container /app/api_with_langfuse.py: docker cp'd patched file; container restarted, healthy.

## Next Session Should Know
- LLMLingua build context is D:\lab\workflow-lab\agents\ (NOT D:\aidirectory\projects\workflow-lab\agents\ — that copy is stale and lacks /compress route). Edit the lab\ path when patching the container.
- Watcher must be restarted via Stop-ScheduledTask + Start-ScheduledTask aidirectory-session-watcher to load the new logic; currently running instance is on old code.
- Mock-test pattern for fix verification: launch general-purpose Agent with explicit checklist + run-against-temp-dir instructions. Caught the locale bug that static review wouldn't have.
- French Windows error strings WILL bite locale-naive string matching elsewhere — audit other scripts for English-only regex on .NET exception messages.

## Knowledge Candidates
- SOP: Patching agent-llmlingua container (edit D:\lab\workflow-lab\agents\api_with_langfuse.py -> docker cp -> docker restart -> verify in logs).
- Pattern: Per-file retry/backoff/skip hashtable structure for any future poller-style script (reusable shape).
- Reference: Windows socket error code 10061 == ConnectionRefused; prefer SocketErrorCode enum over message-string matching for locale safety.
- Fragility note: D:\aidirectory\projects\workflow-lab\agents\ vs D:\lab\workflow-lab\agents\ divergence — clarify or merge.
