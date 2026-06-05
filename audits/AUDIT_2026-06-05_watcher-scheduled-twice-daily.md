---
type: audit
date: 2026-06-05
session_id: watcher-scheduled-twice-daily
surface: claude-code
duration_min: 30
---

## What Was Built / Changed
- `session-watcher.ps1`: added `-Once` (single-pass drain then exit), `-Prompt` (WScript popup Yes/No gate), `-PromptTimeoutSec` (120s default, auto-skips on timeout). Loop changed from `while($true)` to `do/while(-not $Once)`.
- `_register-watcher-task.ps1`: replaced AtLogOn trigger with two Daily triggers (09:00 + 18:00). Args changed to `-Once -Prompt`. ExecutionTimeLimit set to 1h (was unlimited). Added trigger verification to output.
- Scheduled task re-registered elevated via `Start-Process -Verb RunAs`. Verified: State=Ready, two triggers, correct args.
- Independent audit agent deployed: verified all 8 PLAN tasks, mock-tested 3 historical sessions end-to-end.

## Decisions Made
- Popup timeout → SKIP (not run): user intent is "don't drain CPU during important work" — unanswered = busy.
- Daily 09:00 + 18:00: natural rhythm for end-of-morning and end-of-workday session harvests.
- COM WScript.Shell popup: available in interactive user session, no external dependency, simple Yes/No/timeout.

## Errors Encountered
- Task re-registration denied without elevation: used `Start-Process pwsh -Verb RunAs -Wait` | resolved.
- Audit agent initial manual trigger returned Ready in 3s (no popup): correct — no pending files in sessions\ dir. Verified via log entry `PROMPT: no pending session files - exiting without prompt`.

## Token Usage (estimate)
- Input: ~20K | Output: ~4K | Compression: N

## What Worked
- `do/while(-not $Once)` pattern: clean way to make the existing loop one-shot without restructuring logic.
- `Start-Process -Verb RunAs -Wait`: reliable elevation pattern for task registration from non-admin shell.
- Popup fast-exit on no-pending-files: avoids bothering user when nothing to do.
- Audit agent (independent subagent): surfaced the real bug (watcher racing tests) that motivated this change.

## What Didn't Work
- nothing — changes were straightforward.

## Suggested Improvements
- Live popup test with a synthetic session to confirm dialog appearance end-to-end for user.
- HANDOVER [COWORK] section still describes old AtLogOn task — update on next /handover.

## Files Modified
- `scripts\session-watcher.ps1`: -Once/-Prompt flags, do/while loop
- `scripts\_register-watcher-task.ps1`: twice-daily triggers, -Once -Prompt args, 1h limit

## Next Session Should Know
- Watcher is now SCHEDULED (09:00 + 18:00), not always-on. It fires once, shows a popup, drains, exits.
- Popup timeout (120s) = SKIP, not run. User must actively say Yes.
- Task still uses pwsh from WindowsApps path (pinned to 7.6.2.0). Re-run _register-watcher-task.ps1 elevated after PS upgrades.
- To re-register: `Start-Process pwsh -Verb RunAs -Wait -ArgumentList '-File D:\aidirectory\scripts\_register-watcher-task.ps1'`
