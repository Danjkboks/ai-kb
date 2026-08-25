type: extract
date: 2026-06-11
slug: itsm-evsm-fable-build
surface: claude-code
duration_min: 0

task:
  type: new-build
  complexity: medium
  planned: true
  pivots: 0

tasks:
  total: 1
  pass: 1
  fail: 0

files_modified:
  - path: "Projects\ITSM\itsm_evsm_v4_Fable.html"
    change: "created"

decisions:
  - what: "Used canonical W-numbering scheme (remapped after dropping workflows) over raw top-20 rank"
    why: "Sources had two conflicting numbering schemes; canonical scheme is used in evsm_classified_tickets.json and suggestion engine, ensuring consistency."
  - what: "Compressed Media.png to JPEG width 1280, kept as base64 in collapsible catalogue section"
    why: "Original PNG was 260KB; recompression kept total file under 600KB target while preserving image presence."

approach:
  steps_taken: 8
  retries:
    - what: "Attempted to fix a regex substitution in the JavaScript norm() function via command-line Python"
      attempts: 2
      fix: "Wrote a separate fix_norm.py script to handle complex replacement pattern reliably."
  redundant_steps:
    - "Initial attempt to read plan_Fable.md ΓÇö file didn't exist, spec in transcript was sufficient."
  better_approach: "Could have started directly with parse_sources.py after confirming source files; the initial plan file read step was unnecessary."

efficiency:
  tokens_per_task: medium
  bottleneck: "Intermittent WSL rtk hook failures causing command execution retries."
  tokens_input_k: 0
  tokens_output_k: 0

errors:
  - what: "rtk command not found in WSL Ubuntu login shell PATH"
    fix: "Manual retries of Bash commands eventually succeeded."
    status: pending

env_friction:
  - domain: windows-wsl2
    issue: "rtk PreToolUse hook failing intermittently with 'rtk: command not found'"
    fix: "Retried Bash commands; rtk binary likely missing from login-shell PATH."
    status: pending

patterns:
  good:
    - "Separate data extraction (parse_sources.py) from HTML assembly (assemble.py with 80KB chunked appends) for large, self-contained builds."
    - "Synthesizing missing specification details from existing reference files (itsm_evsm_v4_1.html) and embedded data patterns."
  bad:
    - "Attempting complex string replacements via multiline command-line Python prone to quoting issues; better to write a small script immediately."

skill_candidates: []
agent_candidates: []
knowledge_candidates:
  - "Two conflicting W-numbering schemes exist in ITSM EVSM data: tickets_index.json uses raw top-20 rank order; canonical scheme (from v4_1 and classified tickets) renumbers after dropping Safenet/r├⌐union/Guacamole workflows."
  - "Media.png (1605├ù1822 PNG, 260KB) can be recompressed to JPEG width 1280 (~190KB) to keep total HTML file under 600KB while preserving visual reference."
  - "The HTML build pipeline should use chunked appends (80KB blocks) to avoid memory issues and follow project constraints; reassemble via script after part modifications."

next_session: "Itsm_evsm_v4_Fable.html built and validated; reference fable_build directory contains parse_sources.py, data.js, part files, and assemble.py for regeneration; note W-number remapping (W-14ΓåÆW-17, W-15ΓåÆW-18, W-16ΓåÆW-19) due to canonical scheme."
verify_script: ""
