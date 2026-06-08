---
type: audit
date: 2026-06-05
session_id: phase3-retrieval-qdrant-extracts
surface: claude-code
duration_min: 120
---

## What Was Built / Changed
- `scripts/qdrant_extracts_index.py`: reads all `EXTRACT_*.md`, parses YAML frontmatter + `##` sections, embeds with `all-MiniLM-L6-v2`, upserts to Qdrant `extracts` collection (353 chunks from 34 extracts). Deterministic point IDs via `md5(slug::section)`.
- `scripts/qdrant_extracts_search.py`: CLI search (`--query`, `--top 5`). Handles Qdrant-down and missing-collection errors cleanly. UTF-8 stdout reconfigured.
- `.claude/commands/search.md`: `/search` slash command — runs search script, Claude reasons over results, summarises relevant sessions.
- `scripts/reindex-extracts.ps1`: wrapper that runs indexer and appends to `data/audits/reindex.log`.
- `scripts/_register-reindex-task.ps1`: registers Windows Scheduled Task `aidirectory-reindex-extracts` (daily 08:00).
- `HANDOVER.md`: updated with Phase 3 complete status, RTK WSL-only limitation, 3 new fragilities.

## Decisions Made
- `query_points()` over `search()`: `search()` removed in current qdrant-client; `query_points()` is the correct API.
- Deterministic MD5 point IDs: makes re-runs idempotent without tracking state separately.
- Frontmatter delimiter search scoped to first 2KB: prevents `---` inside body content from truncating YAML parse early.
- Batch upsert at 200 points: guards against qdrant-client request size limits as collection grows.
- UTF-8 stdout reconfigure in search script: Windows cp1252 crashes on arrow characters common in extract previews.
- RTK hook removed from Windows settings: Git bash (MSYS2) converts POSIX paths (`/home/...` to Scoop git root), breaking WSL binary invocations. RTK is WSL2-only.

## Errors Encountered
- `pydantic-core==2.47.0` incompatible with `pydantic==2.13.4` (requires 2.46.4): `pip install pydantic-core==2.46.4` | resolved
- `transformers` version rejected `tokenizers==0.23.1`: `pip install transformers --upgrade` | resolved
- `qdrant_client.search()` AttributeError: method removed in current version, replaced with `query_points()` | resolved
- `UnicodeEncodeError` on arrow characters in print statements (cp1252): replaced arrow in indexer, added `sys.stdout.reconfigure(encoding='utf-8')` in search | resolved
- Slug fallback used full filename stem instead of parsed slug: added `re.sub` to strip `EXTRACT_YYYY-MM-DD_` prefix. Required collection recreate to clear stale duplicate-ID points | resolved
- RTK hook broke all Bash calls: MSYS2 expanded `/home/gnren-pc/` to Scoop git root path. Workaround `MSYS_NO_PATHCONV=1` for direct calls; hook removed from settings | resolved
- Code review (post-build) found: missing Qdrant error handling in indexer, unused `Filter` import, single-batch upsert risk, loose frontmatter delimiter | all 4 fixed before audit

## Token Usage (estimate)
- Input: ~150K | Output: ~20K | Compression: N (RTK hook inactive this session)

## What Worked
- Iterative smoke-test pattern (run, check error, fix, re-run): caught encoding, slug, and duplicate-ID issues before they compounded.
- Collection recreate + clean re-index: faster than trying to delete individual bad-ID points.
- Code review before audit: found 4 real issues, all fixed. Pattern worth repeating every session.
- 7-test mock protocol (happy path, service-down x2, missing resource, idempotency, empty input, no-match query): complete coverage for CLI utility scripts of this type.
- `MSYS_NO_PATHCONV=1` prefix: reliable workaround for WSL binary calls from Windows bash.

## What Didn't Work
- Wiring RTK hook in Windows `settings.json` via `wsl -e bash`: MSYS2 path mangling is fundamental. RTK cannot hook Windows Claude Code sessions.
- Running PowerShell with `$variables` via `-Command "..."` in bash: bash expands `$var` before pwsh sees it. Write PS logic to `.ps1` file, invoke with `-File`.

## Suggested Improvements
- Add `/review` slash command: encode the code-review + mock-test protocol as a reusable command. User deferred to next session for design discussion.
- Add `HF_TOKEN` to `.env` and pass to indexer/search: suppresses HuggingFace unauthenticated-request warning on every run.
- Add empty-query guard to search script: `if not args.query.strip(): print("ERROR: --query cannot be empty"); sys.exit(1)`.
- Consider installing RTK on Windows natively (not just WSL2) for Windows Claude Code hook support.

## Files Modified
- `scripts/qdrant_extracts_index.py`: new
- `scripts/qdrant_extracts_search.py`: new
- `.claude/commands/search.md`: new
- `scripts/reindex-extracts.ps1`: new
- `scripts/_register-reindex-task.ps1`: new
- `HANDOVER.md`: Phase 3 complete block, RTK limitation, 3 new fragilities (RTK path mangling, pydantic-core pin, qdrant-client API change)
- `C:\Users\GnReN-PC\.claude\settings.json`: RTK hook added then removed (net: no change vs session start)

## Next Session Should Know
- Phase 3 retrieval is LIVE: `extracts` collection has 353 points (34 extracts, avg 10.4 sections each). Use `/search <query>` before starting any task.
- Run `python scripts\qdrant_extracts_index.py` after new extracts land, or wait for daily 08:00 scheduled task (`aidirectory-reindex-extracts`). Log at `data/audits/reindex.log`.
- qdrant-client API: `.search()` is gone. Use `.query_points()` returning `.points`. In HANDOVER fragilities.
- `pydantic-core` pinned to `2.46.4` — do not upgrade independently of pydantic.
- RTK is WSL2-only. Windows query: `MSYS_NO_PATHCONV=1 wsl -d Ubuntu -- /home/gnren-pc/.local/bin/rtk gain`.
- Next deferred design task: `/review` slash command (code-review + mock-test agent). Read `knowledge/integrity-agent-spec.md` for background, use this session's 7-test mock protocol as the reference implementation.
- Backfill decision still pending: 26 historical `queue\done` sessions have no extract — backfill or accept as loss.
