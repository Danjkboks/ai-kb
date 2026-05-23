---
type: audit
date: 2026-05-24
session_id: index-wikilinks-extract-links
surface: claude-code
duration_min: 35
---

## What Was Built / Changed
- `knowledge/_INDEX.md`: converted audits/extracts/prompts references to Obsidian wikilinks; corrected stale entries; added missing Prompts section.
- `data/proposals/workflow-session-extractor.json`: Build Extract Markdown node now appends a `## Links` block (`related:: [[_INDEX]]` + `tags:`) to every extract.
- Live n8n workflow `session-knowledge-extractor` (id 7l8aP0slan6tRkMy): same node patched via GET-edit-PUT; verified active.

## Decisions Made
- Used GET-edit-PUT against the live workflow instead of PUTting the local JSON: local source-of-truth still has hardcoded secrets and would have clobbered the n8n credential references added last session.
- Rendered actual `d.topics` values for `tags:` rather than literal `{{topics}}`: literal placeholder would not be dataview/tag queryable, defeating the stated purpose.
- Replaced 4 phantom audit entries in _INDEX.md (referenced non-existent files) with the 3 audit files that actually exist on disk.

## Errors Encountered
- PUT returned 400 `settings must NOT have additional properties`: live `settings` carried `binaryMode:separate`; trimmed body settings to `{executionOrder:v1}`.
- Bash heredoc mangled `\n` escapes (turned backslash-n into real newlines) breaking the anchor match: moved the patch logic into a standalone `_patch.py` file using raw strings.

## Token Usage (estimate)
- Input: ~40K | Output: ~6K | Compression: N

## What Worked
- GET-first before PUT surfaced that credentials are stored as references and protected the prior migration.
- Writing the patch to a .py file with raw strings avoided shell escaping entirely.

## What Didn't Work
- Inline `python << 'EOF'` heredocs for code containing backslash escapes — unreliable on this Windows/Git Bash setup.

## Suggested Improvements
- Strip the redundant inline `Authorization` header values (OpenRouter key + GitHub PAT) still present in both the live workflow and local JSON nodes — credential refs now handle auth.
- Add get-sha-then-update to GitHub commit nodes for idempotency (re-running same session_id 422s).

## Files Modified
- `knowledge/_INDEX.md`: wikilinks for audits/extracts, new Prompts section, corrected audit list, Skills(Phase2) note.
- `data/proposals/workflow-session-extractor.json`: appended Links block to Build Extract Markdown node jsCode.

## Next Session Should Know
- SECURITY: inline OpenRouter key and GitHub PAT still live in node headerParameters (DeepSeek + both GitHub nodes), redundant now that credential refs exist. Strip them from live workflow and local JSON.
- Temp files left in `data/proposals/`: `_live_workflow_fetched.json`, `_put_body.json`, `_put_response.json`, `_patch.py` — safe to delete.
- `knowledge/skills/` folder does not exist yet (Phase 2 skill pipeline not producing files); _INDEX note updated to reflect this.
- Tunnel URL this session: `pharmacology-suspected-further-dublin.trycloudflare.com` — changes on every n8n restart.
- n8n public API PUT only accepts name/nodes/connections/settings, and settings only `executionOrder` (no binaryMode).
