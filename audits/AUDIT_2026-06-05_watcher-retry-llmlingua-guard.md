---
type: audit
date: 2026-06-05
session_id: watcher-retry-llmlingua-guard
surface: claude-code
duration_min: 45
---

## What Was Built / Changed
- scripts\session-watcher.ps1: replaced bare polling loop with retry/backoff/skip logic (per-file failure counter, BACKOFF after 3 fails for 5min, SKIP after backoff retry, 30s global PAUSE on connection-refused). Removed eager startup drain so startup files go through the same retry path.
- D:\lab\workflow-lab\agents\api_with_langfuse.py: /compress endpoint now truncates `text` to MAX_INPUT_CHARS=100_000 with a WARNING log before calling LLMLingua (truncate-not-reject).
- agent-llmlingua container: patched file docker cp'd into /app and container restarted; gunicorn healthy.

## Decisions Made
- Truncate oversize /compress input rather than reject with 4xx: partial compression beats OOM crash loop.
- Watcher SKIP state held in-memory only (resets on restart): scheduled-task relaunch at next logon is the natural retry — no need to persist.
- Conn-refused detection by SocketErrorCode enum + multilingual regex (`refus` stem, Windows code 10061): locale-proof on this French Windows host.
- Patch container in place via docker cp instead of full rebuild: faster feedback while host source stays canonical for next rebuild.

## Errors Encountered
- Initial conn-refused regex was English-only ("refused"); on French Windows the exception string is "refusee" so the 30s PAUSE branch never fired. Caught by audit agent's mock test. Fix applied: SocketErrorCode enum check + `refus`/`10061` patterns.
- docker exec with /app path got mangled by Git Bash MSYS path conversion. Fix applied: use `sh -c "... //app/..."` double-slash form.
- /compress returns 500 on degenerate tokenless input (e.g. `'x' * 250000`): pre-existing LLMLingua-2 percentile-math edge case, not reachable from real JSONL. Fix pending (low priority).

## Token Usage (estimate)
- Input: ~25K | Output: ~6K | Compression: N

## What Worked
- Fix -> launch general-purpose Agent with explicit mock-test checklist -> patch findings. Caught the French-locale regex bug static review would have missed.
- docker cp + restart for live container patch: faster than rebuild for single-file Python changes.
- Root-cause diagnosis (retry storm + LLMLingua input size) instead of band-aiding ("just restart n8n").

## What Didn't Work
- Trusting English-only string matching against .NET exception messages on a French Windows host.
- Initial assumption that D:\aidirectory\projects\workflow-lab\agents\ was the LLMLingua build context — it's actually D:\lab\workflow-lab\agents\ (former is stale copy missing /compress).

## Suggested Improvements
- Sync or clearly document the divergence between D:\aidirectory\projects\workflow-lab\agents\api_with_langfuse.py (stale, no /compress) and D:\lab\workflow-lab\agents\api_with_langfuse.py (canonical, in container).
- Wrap LLMLingua compressor.compress_prompt in try/except returning truncated raw text on failure — closes tokenless-input 5xx hole.
- Audit other PS scripts for English-only regex on exception messages (locale-naive matching).
- Persist watcher SKIP state to disk if a tight restart loop would re-hammer a permanently-broken file.

## Files Modified
- scripts\session-watcher.ps1: retry/backoff/skip loop + locale-proof conn-refused detection.
- D:\lab\workflow-lab\agents\api_with_langfuse.py (outside repo): MAX_INPUT_CHARS guard in /compress.
- agent-llmlingua container /app/api_with_langfuse.py: synced via docker cp.

## Next Session Should Know
- LLMLingua build context is D:\lab\workflow-lab\agents\ — NOT D:\aidirectory\projects\workflow-lab\agents\. Edit the lab\ path when patching the container.
- Watcher scheduled task must be Stop/Start'd to load new code: `Stop-ScheduledTask -TaskName aidirectory-session-watcher; Start-ScheduledTask -TaskName aidirectory-session-watcher`. Running instance is still on old code as of this audit.
- Mock-test pattern (audit agent + temp-dir watcher run + docker exec verification) caught a critical locale bug — keep as the post-fix verification flow.
- French Windows error strings WILL bite locale-naive matching elsewhere — flagged for future audit.
