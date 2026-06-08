---
type: audit
date: 2026-05-24
session_id: index-wikilinks-extract-links
surface: claude-code
duration_min: 75
---

## What Was Built / Changed
- `knowledge/_INDEX.md`: converted audits/extracts/prompts references to Obsidian wikilinks; corrected stale entries; added missing Prompts section.
- `data/proposals/workflow-session-extractor.json`: Build Extract Markdown node appends a `## Links` block (`related:: [[_INDEX]]` + `tags:`) to every extract; later fully synced to live (secret-free, credential-referenced).
- Live n8n workflow `session-knowledge-extractor` (id 7l8aP0slan6tRkMy): Links-block patch + full credential migration via GET-edit-PUT; verified active end-to-end.
- Stripped all inline secrets (OpenRouter key + GitHub PAT) from DeepSeek and both GitHub nodes.
- Created new n8n credential `GitHub PAT (header)` (id b73E0FblsizPixDq) from `.env`; repointed both GitHub nodes to it. Deleted stale `Header Auth account` credential (id fjcRlARgJHrwrDld) after removing its references.

## Decisions Made
- GET-edit-PUT the live workflow instead of PUTting local JSON: local source-of-truth had hardcoded secrets and no credential refs; a wholesale PUT would have clobbered the credential migration.
- Rendered actual `d.topics` values for `tags:` rather than literal `{{topics}}`: literal placeholder is not dataview/tag queryable.
- Created a NEW GitHub credential rather than editing in place: n8n public API has no credential update endpoint (only create/delete).
- Fetch System Prompt set to no auth: it hits public `raw.githubusercontent.com`, so it never needed the credential.

## Errors Encountered
- n8n PUT 400 `settings must NOT have additional properties`: live settings carried `binaryMode:separate`; trim PUT body settings to `{executionOrder:v1}`.
- Bash `<< 'EOF'` heredoc mangled `\n` escapes: moved logic into standalone `.py` files with raw strings.
- Test exec ECONNRESET at DeepSeek (transient OpenRouter socket reset, not auth): resolved on retry.
- After stripping inline auth, GitHub commit 401: the `Header Auth account` credential never actually held a valid PAT — prior commits rode on the inline header. Fixed by creating a working credential.

## Token Usage (estimate)
- Input: ~120K | Output: ~14K | Compression: N

## What Worked
- GET-first before every PUT; protected credentials and revealed the true auth path.
- Standalone `.py` files with raw strings for any code containing escapes.
- Real end-to-end webhook runs as the only reliable credential test (auth failures surface as 401 vs ECONNRESET network resets).

## What Didn't Work
- Inline `python << 'EOF'` heredocs for code with backslash escapes on this Windows/Git Bash setup.
- Assuming "credentials migrated" = "credentials working": the GitHub credential was referenced but invalid; only a live run exposed it.

## Suggested Improvements
- Add get-sha-then-update to GitHub commit nodes for idempotency (re-running same session_id 422s).
- Webhook test runs commit a real extract to the repo; consider a dry-run/test branch flag to avoid repo churn + manual cleanup.

## Files Modified
- `knowledge/_INDEX.md`: wikilinks for audits/extracts, new Prompts section, corrected audit list, Skills(Phase2) note.
- `data/proposals/workflow-session-extractor.json`: Links block + synced to final live state (no secrets, credential refs).
- n8n live workflow + credential store (via API): node patches, secret strip, new GitHub credential, old credential deleted.

## Next Session Should Know
- Workflow is fully secret-free and verified working (exec 36 all nodes success). Credentials: DeepSeek -> `Bearer Auth account`; GitHub nodes -> `GitHub PAT (header)` (b73E0FblsizPixDq); Fetch System Prompt -> no auth.
- Local `data/proposals/workflow-session-extractor.json` now uses instance-specific credential IDs; re-import on a fresh n8n needs credentials reattached.
- DeepSeek hallucinates the extract date from content (test extracts came out dated 2025-03-18/19); the `date` field is model-driven, not system clock.
- `knowledge/skills/` folder still does not exist (Phase 2 skill pipeline not emitting files).
- Tunnel URL this session: `pharmacology-suspected-further-dublin.trycloudflare.com` — changes on every n8n restart.
- n8n public API: PUT accepts only name/nodes/connections/settings (settings only `executionOrder`); no credential update endpoint (create/delete only).
