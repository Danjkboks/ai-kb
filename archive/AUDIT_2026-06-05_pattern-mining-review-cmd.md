---
type: audit
date: 2026-06-05
session_id: pattern-mining-review-cmd
surface: claude-code
duration_min: 60
---

## What Was Built / Changed

- `scripts/mine_patterns.py`: New script. Scans all `EXTRACT_*.md` (34) and `AUDIT_*.md` (7) in knowledge/, clusters content by 18 keyword-based topics, filters for 2+ sessions, checks CLAUDE.md coverage, outputs proposals. Idempotent. Result: 14 new patterns, 1 covered.
- `data/proposals/claude-md-additions.md`: Generated proposals file — 14 CLAUDE.md rule candidates + RTK filter table. Awaiting human review before any CLAUDE.md edits.
- `.claude/commands/review.md`: New /review slash command. Determines scope (from $ARGUMENTS or last EXTRACT), queries Qdrant for past patterns, static reviews each file, runs mock tests, writes REVIEW report to knowledge/audits/.
- `knowledge/audits/REVIEW_2026-06-05_mine-patterns-review-cmd.md`: Dog-food review of mine_patterns.py + review.md. 2 bugs caught and fixed, 7 tests passed.
- `knowledge/audits/REVIEW_2026-06-05_mine-patterns-audit-extension.md`: Review of the audit-extension change. 3 stale string fixes, 5 tests passed.

## Decisions Made

- **Topic-based keyword clustering over exact string normalisation**: Exact normalised text matching produced 0 recurring patterns (same topic, different phrasing per session). Topic clusters with keyword lists are robust to paraphrase — correct approach for this use case.
- **Scan both extracts and audits**: Audits have explicit `## What Didn't Work` and `## Suggested Improvements` sections (higher-signal than DeepSeek-generated extracts). 3 patterns only crossed the 2-session threshold after audit inclusion (`session-watcher-dedup`, `wsl-path-mangling`, `pwsh-version-pinned`).
- **Slug dedup across extract+audit pairs**: If the same `session_id` appears in both an extract and an audit, count it once. Prevents double-counting a single session's evidence.
- **Prepend `\n` before frontmatter split**: YAML frontmatter starts with `---\n` (no leading newline), so splitting on `\n---\n` gives 2 parts not 3. Prepending `\n` fixes this without regex.
- **CLAUDE.md hard stop after A3**: Pattern proposals written to `data/proposals/` for human review — not applied automatically. Correct gate.

## Errors Encountered

- `parse_frontmatter` always returned `{}` because files start with `---\n` not `\n---\n` — fixed by prepending `\n` before split | resolved
- Some extracts use YAML key `slug:` instead of `session_id:` — fixed by adding `meta.get("slug")` fallback | resolved
- Two stale `"in extracts"` strings survived the audit extension — caught during /review, fixed immediately | resolved
- Built-in `review` skill intercepted `/review` invocation (it's a PR-review skill) — executed custom command manually by following review.md steps directly | workaround, not a code bug

## Token Usage (estimate)

- Input: ~180K | Output: ~25K | Compression: N

## What Worked

- Dog-fooding `/review` on files just written: caught 2 real bugs (frontmatter parse, slug key fallback) that a silent happy-path run wouldn't surface.
- Reading 2–3 sample extracts before writing the script: confirmed actual section names differ from PLAN's assumed names, preventing a full redesign after the fact.
- Topic-based clustering with keyword lists: immediately yielded actionable results (11 patterns) where exact-match produced zero.
- Incremental test-after-each-change approach: each fix verified before moving to next.

## What Didn't Work

- First approach (exact normalised string match per PLAN spec) produced 0 recurring patterns — same topic is phrased differently in each extract. Required a full redesign to keyword clustering before any output was produced.
- The built-in `review` skill shadows `.claude/commands/review.md` — skill takes precedence when names match. /review cannot be invoked via the Skill tool and must be executed manually.

## Suggested Improvements

- Rename `.claude/commands/review.md` to avoid collision with built-in `review` skill (e.g. `session-review.md`) — or document that it must always be executed manually.
- Add `HF_TOKEN` to `.env` and pass to mine_patterns (and indexer/search) to suppress HuggingFace unauthenticated-request warning on every run.
- `llmlingua-input-size` topic: keyword "llmlingua" alone fires on contextual mentions — tighten to require compound match (llmlingua + one of: crash, max_input, oom, token limit) if false-positive rate becomes a concern.
- `/review` command: add fallback message when $ARGUMENTS is empty and most recent extract's `## Files Modified` is empty or "none".

## Files Modified

- `scripts/mine_patterns.py`: New. Topic-based keyword clustering across extracts + audits, slug dedup, CLAUDE.md coverage check, proposals output.
- `.claude/commands/review.md`: New. /review slash command — scope detection, Qdrant lookup, static review, mock tests, REVIEW report.
- `HANDOVER.md`: Updated — session slug, [CHAT] and [CODE] sections refreshed.
- `knowledge/audits/REVIEW_2026-06-05_mine-patterns-review-cmd.md`: New. Dog-food review report (2 bugs fixed, 7 tests).
- `knowledge/audits/REVIEW_2026-06-05_mine-patterns-audit-extension.md`: New. Review of audit extension (3 fixes, 5 tests).

## Next Session Should Know

- `data/proposals/claude-md-additions.md` has 14 CLAUDE.md rule candidates — review in Chat before next Code session. Top 3 by frequency: `french-locale` (15 sessions), `llmlingua-input-size` (13, some noise), `inline-ps-bash-fail` (8). Approve in Chat, apply in Code.
- `/review` command at `.claude/commands/review.md` is functional but cannot be invoked via the Skill tool — built-in `review` skill intercepts it. Execute manually by following the steps in the file.
- `mine_patterns.py` is idempotent — re-run anytime after new extracts/audits land. Proposals always overwritten.
- 18 topics defined in TOPICS dict in mine_patterns.py — extend it as new recurring failure modes emerge from future sessions.
- All review reports from this session are in `knowledge/audits/` — they form the baseline for future /review dog-food runs.
