---
type: extract
date: 2026-06-06
slug: claude-md-rules-rtk-hook-fix
surface: claude-code
topics: [claude-code, infra, docker, n8n, memory]
skill_candidates: []
agent_candidates: []
duration_min: 60
---

## What Happened
- Renamed `.claude/commands/review.md` → `session-review.md` to fix built-in skill collision; verified with /session-review run (4/4 tests PASS, 0 fixes)
- Applied 11 approved rules from `data/proposals/claude-md-additions.md` to CLAUDE.md: 1 as new NEVER rule (token-waste-re-reading), 10 as a new "Pitfalls" section
- Added `[filters.docker]`, `[filters.git]`, `[filters.n8n]` to RTK `filters.toml` in WSL
- Fixed RTK hook: `wsl -d Ubuntu bash -l -c 'rtk hook claude'` replaces direct-path invocation; updated `rtk-hook.sh` and wired it in `settings.json`
- Diagnosed that broken RTK hook (exit 127) blocks ALL Bash tool calls in Claude Code — had to use Read/Edit/Write/Glob exclusively during repair

## Decisions Made
- RTK hook wrapper uses `bash -l` login shell (not direct path): MSYS2 mangles `/home/...` paths before they reach wsl, even with `export MSYS_NO_PATHCONV=1` — login shell finds `rtk` via PATH instead
- RTK filters use plain prefix match (`^docker`) without word boundary: avoids TOML escape complexity; no functional difference for filtering tool output
- `[filters.n8n]` matches `^mcp__n8n` to cover all n8n MCP tool names
- Pitfalls section added as its own CLAUDE.md section (not merged into Operational Notes) for scan speed
- token-waste-re-reading applied as NEVER rule (Claude behavioral instruction), not as a Pitfall

## Errors Encountered
- RTK filters first write: Python `'\\\b'` in heredoc produced U+0008 (backspace), not regex `\b`. Fixed by dropping word boundary — plain `^docker` prefix is sufficient.
- RTK hook MSYS2 mangling: `MSYS_NO_PATHCONV=1 wsl -- /home/...` still fails — MSYS2 converts path arguments at bash argument-evaluation time, before the env var applies to the process. Both inline and exported MSYS_NO_PATHCONV failed.
- Broken RTK hook created a loop: every Bash tool call triggered the hook, hook exited 127, Claude Code blocked the call. Required non-Bash tools to diagnose and fix settings.json.

## What Worked
- `wsl -d Ubuntu bash -l -c 'rtk hook claude'` — login shell (`-l`) picks up `~/.local/bin` in PATH; stdin/stdout chain through bash to rtk correctly
- Debug log to `/c/Users/GnReN-PC/rtk-debug.log` confirmed MSYS_NO_PATHCONV was unset at hook entry but `wsl -d Ubuntu -- bash -c '...'` (no POSIX path in args) reached WSL correctly
- Edit tool made targeted CLAUDE.md changes without rewriting the whole file

## What Didn't Work
- Direct path in settings.json args array: `["-d", "Ubuntu", "--", "/home/gnren-pc/.local/bin/rtk", "hook", "claude"]` — MSYS2 still mangles the path despite args-array format
- `export MSYS_NO_PATHCONV=1` before wsl call in .sh script — MSYS2 conversion happens at arg-evaluation time in the parent bash, before export takes effect for that command
- Inline `MSYS_NO_PATHCONV=1 wsl ...` — same issue; sets env for child, not for bash's own arg processing

## Suggested Improvements
- Add empty-args abort guard to `session-review.md`: if $ARGUMENTS empty AND Files Modified in latest extract is empty/none → print "No files to review — aborting." and stop (pre-existing WARN carried forward)
- Migrate RTK `strip_lines_matching` patterns to TOML literal strings (single-quoted) — current `"^\s*$"` is technically invalid TOML (unrecognized `\s` escape) but works with RTK's lenient parser

## Files Modified
- `.claude/commands/session-review.md`: created (renamed from review.md, title updated to /session-review)
- `.claude/commands/review.md`: deleted (old name, content preserved in session-review.md)
- `CLAUDE.md`: added `NEVER re-read HANDOVER.md in full mid-session` to Non-Negotiables; added Pitfalls section (10 rules: french-locale, llmlingua-input-size, inline-ps-bash-fail, qdrant-query-api, retry-storm-backoff, docker-volume-mount, n8n-restrict-file-access, async-webhook-no-feedback, filesystemwatcher-scope, wsl-path-mangling)
- `/home/gnren-pc/.config/rtk/filters.toml` (WSL): added [filters.docker] max_lines=50, [filters.git] max_lines=60, [filters.n8n] max_lines=40
- `C:\Users\GnReN-PC\.claude\hooks\rtk-hook.sh`: updated to `export MSYS_NO_PATHCONV=1` + `wsl -d Ubuntu bash -l -c 'rtk hook claude'`
- `C:\Users\GnReN-PC\.claude\settings.json`: RTK PreToolUse hook changed from inline wsl args to `C:/Users/GnReN-PC/.claude/hooks/rtk-hook.sh`
- `knowledge/audits/REVIEW_2026-06-06_session-review-cmd.md`: /session-review report (0 fixes, 1 WARN)
- `C:\Users\GnReN-PC\.claude\projects\D--aidirectory\memory\feedback_rtk-hook-wsl.md`: memory entry for RTK hook fix

## Next Session Should Know
- `/review` is now `/session-review` — the old skill name collided with Claude Code's built-in review skill
- RTK hook is WORKING: `rtk-hook.sh` uses `bash -l` to find rtk via PATH; docker/git/n8n output will be filtered
- HANDOVER fragility note "RTK hook cannot be wired in Windows settings.json" is now OBSOLETE — fixed via bash -l login shell approach
- CLAUDE.md Pitfalls section is new — check it before touching PS scripts, LLMLingua, Qdrant, n8n/Docker, watcher, or WSL paths
- Rejected rules from proposals: session-watcher-dedup, pwsh-version-pinned, ps51-get-scheduledtask (still valid concerns, not added to CLAUDE.md)

## Knowledge Candidates
- SOP: invoking WSL binaries from Windows Git Bash hooks — use `wsl -d Ubuntu bash -l -c 'binary args'` pattern; never pass POSIX paths as wsl `--` arguments from MSYS2 context
