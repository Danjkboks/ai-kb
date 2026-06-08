---
type: extract
date: 2026-05-24
slug: index-wikilinks-extract-links
surface: claude-code
topics: [obsidian, n8n, knowledge-base, infra]
skill_candidates: [n8n-live-node-patch, n8n-credential-migration]
agent_candidates: []
duration_min: 75
---

## What Happened
- Fixed `knowledge/_INDEX.md`: wikilinks for audits/extracts/prompts, new Prompts section, corrected stale audit list and Phase-2 Skills note.
- Added a dataview `## Links` block (`related:: [[_INDEX]]` + `tags:`) to the Build Extract Markdown node; patched live workflow via GET-edit-PUT.
- Stripped all inline secrets (OpenRouter key + GitHub PAT) from DeepSeek and both GitHub nodes (live + local JSON).
- Discovered the GitHub `Header Auth account` credential was invalid (prior commits rode on the inline header); created `GitHub PAT (header)` cred (b73E0FblsizPixDq) from `.env`, repointed GitHub nodes, deleted the stale credential.
- Verified end-to-end with real webhook runs (exec 36 all success); deleted the two test extracts committed to the ai-kb repo; cleaned all scratch files.

## Decisions Made
- GET-edit-PUT live instead of PUTting local JSON: local had inline secrets / no cred refs and would clobber the migration.
- Created a NEW credential rather than editing in place: n8n public API has no credential-update endpoint (create/delete only).
- Fetch System Prompt set to no auth: it hits a public raw URL.
- Rendered actual `d.topics` for `tags:` so dataview queries resolve.

## Errors Encountered
- n8n PUT 400 `settings must NOT have additional properties`: trim body settings to `{executionOrder:v1}` (drop binaryMode).
- DeepSeek ECONNRESET (transient OpenRouter socket reset, not auth): resolved on retry.
- GitHub commit 401 after stripping inline auth: the referenced credential held no valid PAT; fixed by creating a working credential.
- `python << 'EOF'` heredoc mangled `\n` escapes: use standalone `.py` files with raw strings.

## What Worked
- GET-first before every PUT; real webhook runs as the only reliable credential test.
- Distinguishing 401 (auth) from ECONNRESET (network) to diagnose correctly.

## What Didn't Work
- Trusting "credentials migrated" without a live run — the GitHub cred was referenced but invalid.
- Inline heredocs for escape-heavy code on Windows/Git Bash.

## Suggested Improvements
- Add get-sha-then-update to GitHub commit nodes for idempotency (same session_id 422s).
- Add a dry-run/test-branch flag so credential tests don't commit junk extracts to the repo.

## Files Modified
- `knowledge/_INDEX.md`: wikilinks, Prompts section, corrected audit list.
- `data/proposals/workflow-session-extractor.json`: Links block + synced to final live (secret-free, credential-referenced).
- n8n live workflow + credential store (via API): node patches, secret strip, new GitHub cred, old cred deleted.

## Next Session Should Know
- Workflow fully secret-free and verified working. Creds: DeepSeek -> `Bearer Auth account`; GitHub -> `GitHub PAT (header)` (b73E0FblsizPixDq); Fetch System Prompt -> no auth.
- Local JSON uses instance-specific credential IDs; fresh-machine re-import needs credentials reattached.
- DeepSeek sets the extract `date` from content (test extracts dated 2025-03-18/19), not the system clock.
- n8n public API: PUT accepts only name/nodes/connections/settings (settings only `executionOrder`); no credential-update endpoint.

## Knowledge Candidates
- SOP: "Migrate n8n inline secrets to credentials safely" — GET live; strip inline auth; create credential via POST /credentials (no update endpoint exists); repoint nodes; PUT (settings=executionOrder only); verify with a real webhook run; 401=bad cred, ECONNRESET=transient; remove stale credential only after all refs are gone.
