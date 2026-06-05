---
type: review
date: 2026-06-05
slug: mine-patterns-audit-extension
files_reviewed: [scripts/mine_patterns.py]
fixes_applied: 3
issues_flagged: 0
tests_run: 5
tests_passed: 5
---

# CODE REVIEW — 2026-06-05 — mine-patterns-audit-extension

## Fixes Applied

- `scripts/mine_patterns.py`: Two output strings still said "extracts" after audit support was added — `(seen Nx in extracts)` on filter line and `seen in N extracts` on section headers. Both changed to "files" / "sessions" to match actual scope.
- `scripts/mine_patterns.py`: Comment `# Filter topics that fired in 2+ extracts` updated to `# Filter topics that fired in 2+ distinct sessions (extract or audit, deduped by slug)` for accuracy.

## Issues Flagged (needs decision)

none

## Mock Test Results

| # | File | Scenario | Result | Notes |
|---|---|---|---|---|
| 1 | mine_patterns.py | Happy path — 34 extracts + 7 audits | PASS | 14 new patterns, 1 covered |
| 2 | mine_patterns.py | Idempotency — run twice | PASS | stdout byte-identical |
| 3 | mine_patterns.py | AUDIT_TEMPLATE.md exclusion | PASS | Template correctly excluded from audit glob |
| 4 | mine_patterns.py | Slug dedup — same session in both EX and AU files | PASS | watcher-retry-llmlingua-guard counted at most once per topic across all 41 files |
| 5 | mine_patterns.py | No extracts dir (audits only) | PASS | Processes 7 audit files cleanly, produces 5 new patterns |

## Past Patterns Applied

- Qdrant results were not relevant to this specific change (matched on "audit" as a word in extract titles, not the code patterns being reviewed).

## What the Audit Files Added (delta from extract-only run)

- `qdrant-query-api`: 2 → 5 sessions (phase3-retrieval-qdrant-extracts audit explicitly documents the `.search()` removal)
- `retry-storm-backoff`: 4 → 5 sessions (extractor-local-only-rebuild audit adds one more)
- `docker-volume-mount`: 3 → 5 sessions (memory-rag-architecture-build and extractor-local-only-rebuild audits confirmed)
- `n8n-restrict-file-access`: 2 → 3 sessions (extractor-local-only-rebuild audit)
- `session-watcher-dedup`: 0 → 3 sessions (new topic — only surfaced once audits included)
- `pwsh-version-pinned`: 1 → 2 sessions (watcher-scheduled-twice-daily audit tipped it over threshold)
- `wsl-path-mangling`: 1 → 2 sessions (phase3 audit added the MSYS_NO_PATHCONV evidence)
- `cloudflare-tunnel-rotation` (covered): 6 → 11 sessions (now confirmed more strongly)

## Verdict

Clean extension. The dedup logic is correct — sessions covered by both an extract and an audit are counted once. Three 2+ threshold patterns only crossed the bar after audit inclusion (`session-watcher-dedup`, `wsl-path-mangling`, `pwsh-version-pinned`), confirming audits add genuine signal. Ready to use.
