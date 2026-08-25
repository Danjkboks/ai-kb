type: extract
date: 2026-06-08
slug: itsm-evs-html-enrichment-cleanup
surface: claude-code
duration_min: 75

task:
  type: new-build
  complexity: medium
  planned: true
  pivots: 2

tasks:
  total: 4
  pass: 4
  fail: 0

files_modified:
  - path: "Projects\ITSM\itsm_evsm_v4_1.html"
    change: "updated"

decisions:
  - what: "Use Python instead of inline PowerShell for file operations with UTF-8"
    why: "PowerShell quoting and encoding issues corrupted French text; Python script files handled encoding cleanly"
  - what: "Implement tooltips as a single JS-driven position:fixed div appended to body"
    why: "CSS ::after pseudo-elements clipped by card overflow; fixed positioning escapes container boundaries and prevents clipping"
  - what: "Label implementation cells with service names despite workflow ID mismatch"
    why: "HANDOVER IDs w-05/07/15 referenced different services than existing card titles; explicit labeling avoids confusion"

approach:
  steps_taken: 35
  retries:
    - what: "Extract xlsx data with inline Python in bash"
      attempts: 3
      fix: "Write standalone Python script and run via powershell -Command with UTF8 output"
    - what: "Search for w-15 rapport row with regex expecting leading newline"
      attempts: 2
      fix: "Use Python rfind('<tr>') and find('</tr>') to locate row directly"
  redundant_steps:
    - "Running review agent for validation before direct Python verification"
    - "Using PowerShell for substring analysis instead of Python's direct UTF-8 string operations"
  better_approach: "Skip the review agent step; go straight to targeted Python verification to avoid false positives from encoding issues."

efficiency:
  tokens_per_task: medium
  bottleneck: "PowerShell quoting and encoding problems that required fallback to Python scripts"
  tokens_input_k: 0
  tokens_output_k: 0

errors:
  - what: "Python syntax error: 'global html' declaration after variable use"
    fix: "Move global declaration to start of function"
    status: resolved
  - what: "Tooltip clipped by card container overflow with position:absolute"
    fix: "Replaced CSS ::after with JS-driven position:fixed div appended to document.body"
    status: resolved

env_friction:
  - domain: windows-wsl2
    issue: "PowerShell and bash terminal encoding garbled French characters (accents, special symbols)"
    fix: "Use Python with sys.stdout.reconfigure(encoding='utf-8') and write output via Out-File -Encoding UTF8"
    status: workaround

patterns:
  good:
    - "Write surgical Python scripts that load entire HTML, make multiple changes, then write once"
    - "Verify changes with targeted Python verification scripts instead of manual grep"
  bad:
    - "Relying on terminal output for data validation when locale encoding is mismatched"
    - "Deploying review agents before direct file verification when encoding issues are likely"

skill_candidates: []
agent_candidates: []
knowledge_candidates:
  - "For French Windows environments, run Python scripts via `powershell -Command 'python script.py | Out-File -Encoding UTF8 out.txt'` to preserve accented characters"
  - "When editing minified single-line HTML, use Python string methods (find, replace, slicing) rather than line-based operations"
  - "Tooltips with position:fixed appended to body never get clipped by parent container overflow"
  - "When renumbering IDs in sequence, process in reverse order (highest to lowest) to avoid collisions"

next_session: "File `itsm_evsm_v4_1.html` contains 17 continuous workflow cards (W-01 to W-17) after renumbering w-17ΓåÆw-14, w-18ΓåÆw-15, w-19ΓåÆw-16, w-20ΓåÆw-17. All badges have French tooltips via fixed-position div. Portefeuille section uses PF_DATA JS array from Excel; no groupes section."
verify_script: "python -c \"import sys; html=open('Projects\\\\ITSM\\\\itsm_evsm_v4_1.html', encoding='utf-8').read(); assert 'id=\"w-14\"' in html and 'id=\"w-17\"' in html and 'id=\"w-18\"' not in html; assert '17 Workflows' in html and '20 workflows' not in html; assert 'Portefeuille' in html and 'Groupes EVSM' not in html; print('OK')\""
