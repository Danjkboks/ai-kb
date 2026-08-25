type: extract
date: 2026-06-10
slug: itsm-evsm-workflow-insert
surface: claude-code
duration_min: 0

task:
  type: new-build
  complexity: complex
  planned: true
  pivots: 0

tasks:
  total: 1
  pass: 1
  fail: 0

files_modified:
  - path: "Projects/ITSM/itsm_evsm_v4_1.html"
    change: updated

decisions:
  - what: "Use Python binary mode read/write for UTFΓÇæ8 HTML file >100KB"
    why: "Avoid truncation from bash heredoc limits and preserve exact byte encoding"
  - what: "Adapt WFDATA entries to actual panelΓÇærenderer schema (validation/fields/crumb/statut)"
    why: "Handover specs used different property names; matching file's JS prevents runtime errors"
  - what: "Use exact PF_WREF key names from PF_DATA, not workflow card titles"
    why: "PF_WREF must match existing nom keys exactly for proper routing"
  - what: "Replace 'lΓÇÖanalyse' with 'l'analyse' in WFDATA anchor"
    why: "File used straight apostrophe (0x27), not right single quotation mark (U+2019)"

approach:
  steps_taken: 17
  retries:
    - what: "Apply full transformation script with b'...' literals containing nonΓÇæASCII bytes"
      attempts: 3
      fix: "Replace literal UTFΓÇæ8 bytes with escaped \\xnn sequences in .py source via binary edit"
    - what: "Locate WFDATA closing anchor with wrong apostrophe type"
      attempts: 2
      fix: "Found correct anchor byte pattern using grep on actual file content"
  redundant_steps:
    - "Manual hex debugging of b'...' literals; could have preΓÇæescaped all nonΓÇæASCII characters"
  better_approach: "PreΓÇæescape all nonΓÇæASCII characters when generating Python b'...' literals for UTFΓÇæ8 files; use Python's .encode('utfΓÇæ8') instead of handΓÇæcrafted byte strings"

efficiency:
  tokens_per_task: medium
  bottleneck: "Debugging b'...' literal encoding issues and WFDATA anchor mismatch"
  tokens_input_k: 0
  tokens_output_k: 0

errors:
  - what: "SyntaxError: bytes can only contain ASCII literal characters"
    fix: "Replace literal UTFΓÇæ8 bytes (e.g., C3 A0) with escaped \\xnn sequences via binary edit of .py source"
    status: resolved
  - what: "UnicodeEncodeError: 'charmap' codec can't encode character '\u2192'"
    fix: "Set PYTHONIOENCODING=utfΓÇæ8 environment variable before running Python"
    status: resolved
  - what: "S4a anchor not found (wrong apostrophe type in 'lΓÇÖanalyse')"
    fix: "Updated anchor to use straight apostrophe ' (0x27) instead of curly ΓÇÖ (U+2019)"
    status: resolved

env_friction:
  - domain: windowsΓÇæwsl2
    issue: "Windows console default cp1252 encoding cannot print Unicode arrows (ΓåÆ)"
    fix: "Set PYTHONIOENCODING=utfΓÇæ8 before Python execution"
    status: resolved

patterns:
  good:
    - "Verify all anchor strings occur exactly once before applying changes"
    - "Use binary grep to locate exact byte patterns before str_replace"
  bad:
    - "Embedding literal UTFΓÇæ8 bytes in Python b'...' literals without escaping"

skill_candidates: []
agent_candidates: []
knowledge_candidates:
  - "EVSM HTML panel renderer expects WFDATA entries with validation, fields[], crumb[], statut ΓÇö not sla/evsm/note from handover"
  - "PF_WREF keys must match nom values from PF_DATA exactly, not workflow card titles"
  - "French curly apostrophe U+2019 (ΓÇÖ/0xE28099) differs from straight apostrophe ' (0x27); anchors must match file's encoding"

next_session: "ITSM EVSM v4.1 HTML now contains 24 workflows (WΓÇæ01 through WΓÇæ24) with all navigation, JS data, and tree structure updated per HANDOVER_S2_S4.md spec"
verify_script: "python -c \"with open(r'Projects/ITSM/itsm_evsm_v4_1.html','rb') as f: c=f.read(); print('WΓÇæ18 card:', b'id=\"w-18\"' in c, '24 Workflows nav:', b'24 Workflows' in c, 'WFDATA WΓÇæ18:', b'\"WΓÇæ18\"' in c, 'Size:', len(c))\""
