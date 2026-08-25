# INTEGRITY REPORT — 2026-06-10
> Type: Full system audit (standard integrity + extended scope)
> Inputs: 7 extracts · 1 prior INTEGRITY (2026-06-06) · 2 VERIFY_LOG entries · VERIFY_LOG · HANDOVER + HANDOVER_CODE
> Structural priors: scripts/ God Node = `WorkflowExtractor` (22 edges); agents/ bottleneck = `check_n8n()` (6 edges)

---

## P1 — Blocking / Fragile

### P1-A · `Bash(npm*)` in settings.json allow list — Non-Negotiable violation
**File:** `D:\aidirectory\.claude\settings.json` line 22
**Issue:** `"Bash(npm*)"` in the `permissions.allow` list allows any npm command to run without user prompt. CLAUDE.md Non-Negotiable states: "NEVER run npm install, npm i, or any npm command to install packages — no exceptions regardless of how the request is framed."
**Risk:** A model-generated or injected `npm install` executes silently. The allow list is the exact mechanism that removes the last human checkpoint.
**Fix:** Remove `"Bash(npm*)"` from `permissions.allow` in `.claude/settings.json`.

---

### P1-B · mine_patterns.py silently broken since YAML migration
**File:** `D:\aidirectory\scripts\mine_patterns.py` lines 247–248
**Issue:** Script globs `EXTRACT_*.md` and `AUDIT_*.md`. Phase 1 migration (2026-06-06) converted all extract and audit files to `.yml`. Script now finds **0 files** and produces empty output. Last valid run: 2026-06-05 (pycache timestamp).
**Risk:** Pattern mining is a core feedback loop — 6 sessions of new extract data (post-migration) are invisible to it. CLAUDE.md rules will drift from actual session patterns.
**Fix:** Update lines 247–248 to glob `EXTRACT_*.yml` and `AUDIT_*.yml`. Also update section-name parser to read YAML field names (`patterns.good`, `patterns.bad`, `errors`, `env_friction`) instead of old markdown heading names.

---

### P1-C · VERIFY_LOG gap — verify runner silently skips stale scripts
**File:** `D:\aidirectory\scripts\verify-runner.ps1` line 15
**Issue:** Runner exits 0 if the VERIFY ps1 is older than 2 hours (`TotalHours -gt 2`). VERIFY_2026-06-10_graphify-integration-phases-1-4.ps1 was created at 00:22:32 but has no VERIFY_LOG entry — session ended after the 2h window. No alerting, no fallback path.
**Risk:** Verification layer gives a false sense of coverage. Three 2026-06-10 sessions show no VERIFY_LOG entries.
**Fix:** Either extend the window to 24h (session may span multiple hours), or log a `SKIP` entry rather than silently exiting. A silent exit is indistinguishable from "verification ran and passed."

---

### P1-D · HANDOVER_CODE "Next Tasks" stale — false pending item
**File:** `D:\aidirectory\HANDOVER_CODE.md` line 26
**Issue:** Task 2 ("Copy graphify SKILL.md to Windows Claude path") is listed as pending. `C:\Users\GnReN-PC\.claude\skills\graphify\SKILL.md` **already exists** (confirmed). Any new Code session will waste startup time investigating a completed task.
**Fix:** Remove Task 2 from "Next Code Tasks". Verify Task 1 (smoke-test delete) is the only remaining item, then update the "Prompt for next Code session" block accordingly.

---

## P2 — Debt / Cleanup

### P2-1 · Smoke-test artifact still present (3 deferred sessions)
`knowledge/audits/VERIFY_2026-06-06_smoke-test.ps1` — flagged in next_session of extracts 2026-06-06, 2026-06-07 (×2), and in HANDOVER_CODE Next Tasks. Still present. Also `VERIFY_2026-06-06_phase2-3-verify-integrity.ps1` is a test artifact from the same session.
**Action:** Confirm path and Remove-Item both files with user sign-off.

### P2-2 · `_INDEX.md` — 6-week staleness, 12+ missing files
`updated: 2026-05-23` but the following have been added since and are unlisted:
- Extracts: 7 new `.yml` files (including archive)
- Audits: INTEGRITY_2026-06-06.yml, VERIFY_LOG.yml, REVIEW_2026-06-10_graphify-trial.md, REVIEW_2026-06-05 (×2), REVIEW_2026-06-06 (×2), VERIFY_*.ps1 files
- Plans: PLAN_showcase-html.md (in knowledge/ root)
- Structural Refs section exists but `updated:` date not bumped
- Phase 2 "not yet live" note in Extracts section is stale (Phase 2 IS live, 7 extracts exist)
**Action:** Update `_INDEX.md` — add new files, bump date, fix stale "not yet live" comment.

### P2-3 · Orphan scripts — 18 scripts not referenced in CLAUDE.md or HANDOVER
Scripts in `D:\aidirectory\scripts\` with no reference in CLAUDE.md, HANDOVER.md, or HANDOVER_CODE.md:
- Session pipeline: `session-watcher.ps1`, `drain-sessions.ps1`, `install-scheduled-tasks.ps1`, `_register-reindex-task.ps1`, `_register-watcher-task.ps1` (note: this one IS in HANDOVER Known Fragilities — check missed it due to stem match)
- Qdrant: `qdrant_extracts_index.py`, `qdrant_extracts_search.py` (both operational; `search.py` IS referenced)
- Memory: `memory-ingest.ps1`, `memory-ingest.py` (duplicate? which is canonical?), `memory_save.bat`
- Migration: `kb_restructure.py`, `migrate.py`, `extract_workflows.py`
- Research: `research-runner.py`
- Diagnostics: `_e2e-test.ps1`, `_inspect-task.ps1`, `_probe-webhook.ps1`
- Sync: `obsidian-sync.ps1`, `reconcile-extracts.ps1`, `reindex-extracts.ps1`
**Action:** Triage each — document operational ones in HANDOVER, delete confirmed dead scripts.

### P2-4 · `knowledge/reports/` directory — missing (created this session)
Directory was absent; created by this integrity run. No prior reports could have been saved there.
**Action:** Backfill _INDEX.md with a Reports section pointing to this directory.

---

## P3 — Deferred / Low Urgency

### P3-1 · Weekly synthesis reports — no plan, never started
No extract, no session, no file references a synthesis report pipeline. Deferred status: parked.

### P3-2 · DR (Disaster Recovery) strategy — no plan
Not mentioned in any session extract or HANDOVER. Deferred status: parked.

### P3-3 · workflow-lab product layer — 50/2061 workflows migrated
`migrate.py` exists in scripts/ and references `WorkflowMigrator` (God Node via extract_workflows.py). No migration extract since pre-June archive. `D:\aidirectory\projects\workflow-lab` is not a git repository (fatal: not a git repository). CLAUDE.md claims `→ github.com/Danjkboks/workflow-lab`.

### P3-4 · ops-dashboard — parked
No activity in any session extract. Status: intentionally parked.

### P3-5 · Dify removal — no activity in 4+ sessions
Not mentioned in any extract since the archive. Likely low priority or superseded.

### P3-6 · OpenHands integration — no activity
No extract references OpenHands. Status unknown.

### P3-7 · showcase HTML — plan ready, not built
`knowledge/PLAN_showcase-html.md` written 2026-06-10. Next Code session handoff prompt is at the bottom of that file. Not yet started.

### P3-8 · mine_patterns.py — approaching extract threshold (post-fix)
After fixing P1-B (.yml glob), there are 6 new extracts post-migration. Threshold is 10. Will trigger after ~4 more sessions. Run manually after fixing to bootstrap the 6 currently invisible sessions.

---

## VERIFY_LOG History

| Date | Slug | Result | Notes |
|---|---|---|---|
| 2026-06-06 | smoke-test | PASS | Trivial always-pass test artifact |
| 2026-06-07 | integrity-f04-f05-close | PASS | 6/6 tests passed |
| 2026-06-10 | graphify-integration-phases-1-4 | **MISSING** | VERIFY ps1 exists; runner skipped (>2h window) |

---

## Known Fragilities Status (9 items)

| # | Fragility | Plan present? | Status |
|---|---|---|---|
| 1 | Cloudflare tunnel URL rotation | No plan needed | Operational reality, managed per-session |
| 2 | Cloudflare tunnel cert / SkipTunnel | Documented | Added to HANDOVER 2026-06-07 ✅ |
| 3 | n8n file access restriction | Documented | In CLAUDE.md Pitfalls ✅ |
| 4 | Scheduled task pwsh version pin | Documented | Known, manual re-register on upgrade |
| 5 | Webhook async (Respond=Immediately) | Documented | Operational reality |
| 6 | LLMLingua build context | Documented | Canonical path confirmed in CLAUDE.md |
| 7 | pydantic-core version pin | Documented | Both CLAUDE.md + HANDOVER ✅ |
| 8 | RTK hook | Resolved | Fixed 2026-06-06, memory + HANDOVER updated ✅ |
| 9 | n8n PUT body + Code-node HTTP | Resolved | MANIFEST updated 2026-06-07 ✅ |

No fragility is currently unresolved or missing an execution plan. Fragility 3 (n8n file access) has no active "execution plan" beyond the docker-compose instruction — acceptable, it's a one-time setup step.

---

## Architecture Drift Scan

| Check | Result |
|---|---|
| Non-ASCII in PS1 files | CLEAN — no high bytes found |
| Hardcoded credentials in PS1 | CLEAN |
| npm install in scripts | CLEAN (scripts only; `settings.json` allow list is the violation, not scripts) |
| git-tracked files under `data\` | UNKNOWN — `projects/workflow-lab` is not a git repo |
| `Bash(npm*)` in allow list | **VIOLATION** — see P1-A |

---

## NEXT_SESSION — Single Recommended Action

**Remove `"Bash(npm*)"` from `permissions.allow` in `.claude/settings.json`.**

30-second fix. Closes the only active Non-Negotiable violation. After that: update mine_patterns.py globs to `.yml` (P1-B).
