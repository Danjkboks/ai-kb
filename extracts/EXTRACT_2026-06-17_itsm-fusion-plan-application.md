type: extract
date: 2026-06-17
slug: itsm-fusion-plan-application
surface: claude-code
duration_min: 0

task:
  type: new-build
  complexity: medium
  planned: true
  pivots: 2

tasks:
  total: 1
  pass: 1
  fail: 0

files_modified:
  - path: "Projects/ITSM/itsm_evsm_v4_1.html"
    change: "updated"
  - path: "Projects/ITSM/HANDOVER.md"
    change: "updated"

decisions:
  - what: "EDIT 1 uses header value 22 workflows (not 23) because W-25 is 'Nouveau workflow' with no ticket data, exluding it from baseline count."
    why: "Plan asked to resolve 22 vs 23; count of 25 id='w-' cards minus 2 fusions and minus 1 hors-comptage gives 22."

approach:
  steps_taken: 18
  retries:
    - what: "EDIT 14b badge replacement failed due to 5 matches instead of expected 1"
      attempts: 2
      fix: "Scoped replacement to index window between w-03 and w-04 markers only"
    - what: "Composants section collapsible not working ΓÇö initially appeared visible and click did nothing"
      attempts: 3
      fix: "CSS specificity issue: changed .comp-coll{display:none} to .comp-grid.comp-coll{display:none !important}"
  redundant_steps:
    - "Multiple probe scripts for anchor checking could have been consolidated into one script upfront"
  better_approach: "Directly use index-based replacement strategy for all edits, not just EDIT 21 ΓÇö would avoid false-positive count matches for shared substrings."

efficiency:
  tokens_per_task: medium
  bottleneck: "Debugging CSS specificity and nested structure errors consumed extra steps/tokens"
  tokens_input_k: 0
  tokens_output_k: 0

errors:
  - what: "EDIT 21 index-based strategy inadvertently omitted the closing </div> of the W-06 wrapper"
    fix: "Not fixed directly; later deletion of W-06 removed the unclosed opener, implicitly fixing the DOM nesting"
    status: resolved
  - what: "Composants section collapsible failure due to CSS cascade: .comp-grid{display:grid} overrode .comp-coll{display:none}"
    fix: "Increased selector specificity to .comp-grid.comp-coll{display:none !important}"
    status: resolved

env_friction:
  - domain: windows-wsl2
    issue: "Python command 'python' not found; py launcher required"
    fix: "Used 'py -3' instead of 'python'"
    status: resolved
  - domain: windows-wsl2
    issue: "Console encoding mismatch causing Unicode decode errors"
    fix: "Wrote standalone Python scripts with explicit utf-8 encoding instead of inline python -c"
    status: resolved

patterns:
  good:
    - "Verify anchor uniqueness before replacement by counting matches and scoping to index windows"
  bad:
    - "Using single-class CSS selectors that lose cascade to equally-specific selectors later in stylesheet"

skill_candidates: []
agent_candidates: []
knowledge_candidates:
  - "In ITSM fusion plan edits, numeric values in HTML may contain U+202F (narrow no-break space) characters, not regular spaces; replacement strings must match exactly."
  - "For collapsible sections in this codebase, CSS selector must be at least as specific as the existing grid rule (e.g., .comp-grid.comp-coll, not .comp-coll alone) to override display property."
  - "When deleting HTML blocks that were previously modified by a slice-based edit, verify DOM nesting integrity ΓÇö missing closing tags can cause subsequent siblings to be nested incorrectly."

next_session: "EDIT 21 left structural bug: W-06 wrapper missing closing </div>, causing W-07 through W-25 to be nested inside W-06 in DOM; deletion of W-06 fixed it. If future edits replace cbody content, ensure the parent .wf closing tag is preserved in slice."
verify_script: ""
