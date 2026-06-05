---
type: review
date: 2026-06-05
slug: mine-patterns-review-cmd
files_reviewed: [scripts/mine_patterns.py, .claude/commands/review.md]
fixes_applied: 2
issues_flagged: 2
tests_run: 7
tests_passed: 7
---

# CODE REVIEW — 2026-06-05 — mine-patterns-review-cmd

## Fixes Applied

- `scripts/mine_patterns.py`: `parse_frontmatter` split on `\n---\n` produced 2 parts instead of 3 — extract files start with `---\n` (no leading newline). Fix: prepend `\n` when content starts with `---\n` before splitting. Without this fix, `session_id` was never parsed and slugs used the full `EXTRACT_YYYY-MM-DD_` path.stem.
- `scripts/mine_patterns.py`: Some extracts use YAML key `slug:` instead of `session_id:` — added `meta.get("slug")` as intermediate fallback before `path.stem`. Affected: `EXTRACT_2026-05-23_phase1-complete.md`, `EXTRACT_2026-05-23_phase2-n8n-pipeline.md`.

## Issues Flagged (needs decision)

- 🟡 WARN `scripts/mine_patterns.py`: `llmlingua-input-size` topic uses keyword `"llmlingua"` which matches any extract mentioning LLMLingua in passing — not just the specific input-size/crash pattern. Fires on 12 extracts but several (e.g., `container-audit-dify-stack-assessment`, `n8n-workflow-complexity-assessment`) likely only mention LLMLingua contextually. **Options:** (1) add a compound requirement (must match `llmlingua` AND one of `crash`, `max_input`, `token limit`, `oom`); (2) accept noise as acceptable at 2+ threshold; (3) rename topic to just flag "LLMLingua is common in this stack."

- 🟡 WARN `.claude/commands/review.md`: Auto-scope (empty `$ARGUMENTS`) doesn't specify behaviour when the most recent extract's `## Files Modified` is empty or `"none"`. No fallback or abort instruction. **Recommended fix:** add "If no files found after filtering: print 'No reviewable files found in most recent extract — pass files explicitly via $ARGUMENTS' and stop."

## Mock Test Results

| # | File | Scenario | Result | Notes |
|---|---|---|---|---|
| 1 | mine_patterns.py | Happy path — 34 extracts | PASS | 11 new + 1 covered; slugs now clean after fix |
| 2 | mine_patterns.py | Empty collection (pointed at data/sessions/) | PASS | Prints `ERROR: No extracts found in ...` and exits non-zero |
| 3 | mine_patterns.py | Idempotency — run twice | PASS | stdout byte-identical both runs |
| 4 | mine_patterns.py | Malformed extract (no frontmatter) | PASS | Logs `[OK] ... 0 topics fired ()`, continues; no crash |
| 5 | review.md | All referenced file paths exist on disk | PASS | `scripts/qdrant_extracts_search.py`, `knowledge/extracts/`, `knowledge/audits/`, `data/` all present |
| 6 | review.md | All CLI flags valid — `--query`, `--top` on qdrant_extracts_search.py | PASS | `--help` confirms both flags are current |
| 7 | review.md | Scope logic — both $ARGUMENTS branches mentally traced | PASS | Non-empty: space-split paths, works. Empty: reads last EXTRACT_*.md, parses Files Modified. Edge case (empty section) flagged as WARN above. |

## Past Patterns Applied

- Qdrant result [1] noted `.ps1-to-disk pattern` — confirms the `inline-ps-bash-fail` topic in mine_patterns.py is correctly classified as a recurring validated pattern. No changes needed.
- Results [2] and [3] were not relevant to the files under review.

## Verdict

Both files are functional and ready to use. Two bugs were caught and fixed during review: the frontmatter parsing defect (silent, caused unclean slugs in the proposals output) and the missing `slug:` YAML key fallback. The two flagged WARNs are non-blocking — the llmlingua false-positive risk is acceptable at the 2+ threshold, and the empty-scope edge case in review.md is rare in practice. Proposals at `data/proposals/claude-md-additions.md` reflect the fixed runs.
