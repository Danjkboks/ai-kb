# BUILD PLAN — Audit YAML Migration + Verification Layer + Integrity Agent
> Created: 2026-06-06 | Surface: Chat | Status: Ready for Code execution
> Covers: 3 Code sessions, sequential dependency order

---

## Context

This plan formalises decisions from the 2026-06-06 Chat session. Three problems addressed:

1. Current EXTRACT/AUDIT files are prose — token-expensive, not parseable for pattern learning
2. No post-session verification that what was built actually works
3. Integrity agent spec exists but lacks efficiency analysis scope and clean input format

Sessions must run in order. Phase 2 depends on Phase 1 (needs YAML schema live). Phase 3 depends on Phase 1 (needs YAML files to reason over).

---

## File Format Taxonomy (reference — do not change)

| File type | Format | Reason |
|---|---|---|
| `EXTRACT_*.yml` | **YAML** | Read by agents for data extraction |
| `AUDIT_*.yml` | **YAML** | Read by agents for data extraction |
| `VERIFY_*.yml` | **YAML** | Machine-generated, machine-read |
| `HANDOVER*.md` | Markdown | Context injection for Claude — narrative required |
| `CLAUDE.md` | Markdown | Instructions to Claude — prose more effective |
| `.claude/commands/*.md` | Markdown | Instructions to Claude |
| `PLAN.md`, `README.md` | Markdown | Human documentation |
| `knowledge/reference_*.md` | Markdown | Reference docs — humans read these |

Rule: **context/instructions → Markdown. Data accumulation → YAML.**

---

## Finalized YAML Schema

Single schema for both EXTRACT and AUDIT files. `type` field distinguishes them.

```yaml
# -- HEADER --
type: extract            # extract | audit | verify
date: YYYY-MM-DD
slug: kebab-case-slug
surface: claude-code     # chat | cowork | claude-code
duration_min: 45

# -- TASK PROFILE --
task:
  type: new-build        # new-build | debug | config | migration | refactor | research
  complexity: medium     # simple | medium | complex
  planned: true          # clear plan at session start?
  pivots: 0              # significant direction changes mid-session

# -- DELIVERABLES --
tasks:
  total: 6
  pass: 6
  fail: 0

files_modified:
  - path: "relative/path/to/file.md"
    change: "created"    # created | updated | deleted | refactored

# -- DECISIONS --
decisions:
  - what: "one-line decision"
    why: "rationale -- why this over alternatives"

# -- APPROACH TRACE -- (key for efficiency learning)
approach:
  steps_taken: 8         # rough tool-call count for main deliverable
  retries:
    - what: "describe the attempted action"
      attempts: 2
      fix: "what finally worked"
  redundant_steps:
    - "describe any step that could have been skipped in hindsight"
  better_approach: ""    # freeform -- what would have been faster?

# -- EFFICIENCY --
efficiency:
  tokens_per_task: medium  # low | medium | high -- subjective signal
  bottleneck: ""           # what consumed the most unnecessary tokens/steps?
  tokens_input_k: 45
  tokens_output_k: 12

# -- ERRORS --
errors:
  - what: "error description"
    fix: "fix applied"
    status: resolved     # resolved | pending

# -- ENVIRONMENT FRICTION -- (structured env issues)
env_friction:
  - domain: windows-wsl2  # windows-wsl2 | docker | n8n | powershell | qdrant | python
    issue: "description"
    fix: "what resolved it"
    status: resolved      # resolved | pending | workaround

# -- PATTERNS --
patterns:
  good:
    - "pattern or approach worth repeating"
  bad:
    - "pattern or approach to avoid"

# -- KNOWLEDGE --
skill_candidates: []
agent_candidates: []
knowledge_candidates: []

# -- NEXT SESSION --
next_session: "one-line critical context -- what a cold session must know"
```

**Schema rules:**
- All fields mandatory. Write `[]` or `""` or `null` -- never omit a field.
- `approach.retries` and `approach.redundant_steps`: write `[]` if none. Do not omit.
- `env_friction`: write `[]` if clean session. Critical for pattern clustering.
- `better_approach`: fill even if session was clean -- forces reflection.
- `next_session`: one line only. The most important carry-forward fact.

---

## PHASE 1 -- Format Foundation
> Dependency: none | Estimated time: 1 Code session (~60-90 min)

**Task 1 -- Update `/audit` command**

File: `D:\aidirectory\.claude\commands\audit.md`

- Replace markdown template in Step 2 with YAML schema above
- Update filename convention: `AUDIT_<date>_<slug>.yml` (not `.md`)
- Update output path: `D:\aidirectory\knowledge\audits\AUDIT_<date>_<slug>.yml`
- Keep Step 1 (session-audit.ps1 scaffold) -- update extension only
- Replace Rules prose with inline YAML field rules

**Task 2 -- Update `/wrap` command**

File: `D:\aidirectory\.claude\commands\wrap.md`

- Replace markdown template with YAML schema above
- Update filename: `EXTRACT_<date>_<slug>.yml`
- Update output path: `D:\aidirectory\knowledge\extracts\EXTRACT_<date>_<slug>.yml`
- Step 4 (Code sessions): after writing EXTRACT, also write `VERIFY_<date>_<slug>.ps1` to `knowledge\audits\` (see Phase 2 for script spec)

**Task 3 -- Update `/session-review` command**

File: `D:\aidirectory\.claude\commands\session-review.md`

- Step 1: find most recent `EXTRACT_*.yml` (not `.md`)
- Parse `files_modified[].path` field instead of `## Files Modified` prose

**Task 4 -- Archive migration (one-time)**

- Create `D:\aidirectory\knowledge\archive\`
- Move all `EXTRACT_*.md` from `knowledge\extracts\` to `knowledge\archive\`
- Move all `AUDIT_*.md` from `knowledge\audits\` to `knowledge\archive\`
- Read all archived files, write digest:
  `D:\aidirectory\knowledge\extracts\EXTRACT_ARCHIVE_pre-2026-06-06.yml`

Digest format:
```yaml
type: extract
date: 2026-06-06
slug: archive-digest-pre-cutoff
surface: mixed
note: "Consolidated digest of all sessions before YAML format migration"
sessions:
  - slug: session-slug
    date: YYYY-MM-DD
    surface: chat
    summary: "one-line: what happened"
    decisions: ["decision"]
    errors: ["error: fix"]
    patterns_good: ["pattern"]
    patterns_bad: ["pattern"]
    env_friction: []
    next_session: "carry-forward note"
```

**Task 5 -- Update `session-audit.ps1`**

File: `D:\aidirectory\scripts\session-audit.ps1`
- Update generated file extension from `.md` to `.yml`

**Task 6 -- Verify end-to-end**
- Run `/wrap` in test session
- Confirm YAML written with all fields
- Confirm `/session-review` reads `.yml` extract correctly

---

## PHASE 2 -- Verification Layer
> Dependency: Phase 1 complete | Estimated time: 1 Code session (~60 min)

Flow:
```
/wrap runs -> writes EXTRACT_<date>_<slug>.yml + VERIFY_<date>_<slug>.ps1
           -> SessionEnd hook fires -> verify-runner.ps1
           -> finds latest VERIFY_*.ps1 -> executes it
           -> appends result to VERIFY_LOG.yml
```

**Task 1 -- Add SessionEnd hook**

File: `D:\aidirectory\.claude\settings.json` (create or merge)

```json
{
  "hooks": {
    "SessionEnd": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "pwsh.exe -NoProfile -ExecutionPolicy Bypass -File D:\\aidirectory\\scripts\\verify-runner.ps1"
          }
        ]
      }
    ]
  }
}
```

**Task 2 -- Create `verify-runner.ps1`**

File: `D:\aidirectory\scripts\verify-runner.ps1`

Logic:
1. Find most recent VERIFY_*.ps1 in `knowledge\audits\` by lastwritetime
2. If none: exit 0 silently
3. If found: check written in last 2 hours (session window guard)
4. If stale: exit 0 silently
5. Execute VERIFY_*.ps1, capture exit code + output
6. Append result to `knowledge\audits\VERIFY_LOG.yml`
7. Exit 0 always -- never block session close

VERIFY_LOG.yml entry format:
```yaml
- date: YYYY-MM-DD
  time: HH:MM
  slug: session-slug
  result: PASS       # PASS | FAIL | SKIP
  tests_run: 4
  tests_passed: 4
  failures: []
  output_tail: ""    # last 5 lines if FAIL
```

**Task 3 -- Update `/wrap` Step 3 (VERIFY script generation)**

After writing EXTRACT, write `VERIFY_<date>_<slug>.ps1` to `knowledge\audits\`:
- Test-Path existence check for every file in `files_modified[].path`
- .ps1 files: syntax/load check
- .py files: `python -c "import ast; ast.parse(open('path').read())"` syntax check
- .md command files: verify all @-referenced paths exist
- Exit 0 all pass, exit 1 with failure list
- Keep under 50 lines
- Add field to EXTRACT: `verify_script: "path"`

**Task 4 -- Test end-to-end**
- Modify file, run `/wrap`, close session
- Confirm hook fires, VERIFY_LOG.yml updated

---

## PHASE 3 -- Integrity Agent
> Dependency: Phase 1 complete | Estimated time: 1 Code session (~90 min)

Scope: cross-session coherence detection + execution efficiency analysis.

**Task 1 -- Create `/integrity` command**

File: `D:\aidirectory\.claude\commands\integrity.md`

```markdown
# /integrity [target] -- Cross-Session Coherence + Efficiency Agent
> Output: D:\aidirectory\knowledge\audits\INTEGRITY_<date>.yml

## Mandatory read sequence (always, regardless of target)
1. D:\aidirectory\CLAUDE.md
2. D:\aidirectory\HANDOVER.md + HANDOVER_CODE.md
3. D:\aidirectory\knowledge\extracts\ -- all EXTRACT_*.yml
4. D:\aidirectory\knowledge\audits\ -- all AUDIT_*.yml + INTEGRITY_*.yml
5. D:\aidirectory\.claude\commands\*.md
6. D:\aidirectory\scripts\ -- read .ps1/.py files referenced in extracts

Additional reads per target:
  folder:<path> -> read all files in path
  agent:<name>  -> read projects/<name>/ in full
  idea:<topic>  -> no extra files, use loaded context only

## Reasoning loop

Step 1 -- MAP
Build internal model: rules, builds, plans, actuals.

Step 2 -- COHERENCE CHECK
- CLAUDE.md rules vs extract behaviour (contradicted = HIGH)
- HANDOVER decisions vs scripts (never implemented = MEDIUM)
- HANDOVER_CODE Next Tasks vs extract history (stale = LOW)
- env_friction in extracts vs HANDOVER Known Fragilities (unfixed = HIGH)

Step 3 -- EFFICIENCY ANALYSIS
- Find same task.type across multiple sessions
- Compare approach.steps_taken for same task type
- Cluster approach.retries -- same fix repeated = systemic gap
- Cluster approach.redundant_steps recurring across sessions
- Flag: steps_taken consistently > 2x minimum observed for that task type

Step 4 -- PATTERN CLUSTER
- env_friction by domain: 3+ entries = systemic
- errors: same description in 2+ sessions = recurring
- patterns.bad: 2+ sessions, no CLAUDE.md rule = unaddressed pattern

Step 5 -- OUTPUT
Apply caps. Write YAML.

## Output format

```yaml
type: integrity
date: YYYY-MM-DD
target: global
inputs:
  extracts: N
  audits: N
  prev_integrity_reviews: N

summary:
  - "bullet 1"   # max 3 bullets
  - "bullet 2"
  - "bullet 3"

findings:
  - id: F01
    severity: CRITICAL  # CRITICAL(max 3) | HIGH(max 5) | MEDIUM | LOW (max 5 combined)
    confidence: HIGH    # HIGH | MEDIUM | LOW
    category: coherence # coherence | efficiency | env_friction | pattern
    component: "exact file or script path"
    issue: "one sentence"
    evidence: "quote or field value [source: path]"
    prediction: "what breaks if ignored"
    fix: "concrete action"

unknowns:
  - "unresolvable -- needs user clarification"

delta:
  - id: F01
    prev_date: YYYY-MM-DD
    status: RESOLVED    # RESOLVED | ESCALATED | UNCHANGED

handover_candidates:
  - "proposal for HANDOVER Known Fragilities -- user decides"
```

## Hard rules
- Evidence required on every finding. No evidence = finding rejected.
- UNKNOWN = flag, never guess.
- Never write HANDOVER directly. handover_candidates only.
- Recurring finding (3+ reviews UNCHANGED): auto-escalate to CRITICAL.
- If prev integrity review exists: always populate delta.
```

**Task 2 -- Add efficiency fields reminder to `/audit`**

Add to Rules section:
```
- approach.retries and approach.redundant_steps: fill honestly.
  Highest-signal fields for long-term efficiency learning. [] acceptable. Omit = not acceptable.
- better_approach: always fill. "none identified" is valid. Ran same command twice = redundant step.
```

**Task 3 -- Test**
- Run `/integrity` against archive digest + 1-2 new YAML files
- Verify finding caps enforced, evidence populated, YAML written

---

## Handoff Prompts

### Phase 1
```
@HANDOVER.md @HANDOVER_CODE.md

Build plan: D:\aidirectory\knowledge\audits\BUILD_PLAN_audit-yaml-integrity-agent.md
Execute Phase 1 in order: update /audit, /wrap, /session-review, archive migration, session-audit.ps1, test.
Schema field names are final -- do not deviate.
All .ps1: plain ASCII only, pwsh.exe.
```

### Phase 2
```
@HANDOVER.md @HANDOVER_CODE.md

Phase 1 complete. Build plan: D:\aidirectory\knowledge\audits\BUILD_PLAN_audit-yaml-integrity-agent.md
Execute Phase 2: SessionEnd hook, verify-runner.ps1, /wrap VERIFY step, end-to-end test.
Hook exits 0 always. verify-runner.ps1 handles missing VERIFY file gracefully.
```

### Phase 3
```
@HANDOVER.md @HANDOVER_CODE.md

Phase 1 complete. Build plan: D:\aidirectory\knowledge\audits\BUILD_PLAN_audit-yaml-integrity-agent.md
Execute Phase 3: /integrity command, /audit efficiency fields, test.
Scope: coherence + efficiency. Never writes HANDOVER directly.
```

---

## Summary

| Phase | What | Dependency |
|---|---|---|
| 1 | YAML format migration + archive | None -- do first |
| 2 | SessionEnd hook + verify layer | Phase 1 |
| 3 | Integrity agent | Phase 1 |

Phases 2 and 3 can run in parallel after Phase 1.
