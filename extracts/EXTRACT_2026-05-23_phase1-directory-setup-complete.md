---
type: extract
date: 2026-05-23
session_id: phase1-directory-setup-complete
surface: claude-code
environment: env1-claude-desktop
topics: [claude-code, memory, git, windows]
source_file: 2026-06-05-160219_3d8df92b-b89a-423e-b473-acd42d79f7de.jsonl
processed_at: 2026-06-07T16:02:32.372Z
---

# Session Extract: phase1-directory-setup-complete

## Decisions
- **Keep knowledge/ git-tracked but data/ untracked**: Operational data (sessions, queue, audits, proposals) should not be in git; only curated knowledge outputs (extracts, skills) should be tracked
- **Use PowerShell.exe for Windows directory creation instead of WSL bash mkdir**: WSL mkdir creates directories in WSL filesystem, not Windows filesystem; PowerShell New-Item works natively
- **Ban non-ASCII characters (especially em-dash) in .ps1 files**: Windows 11 PS 5.1 with French locale uses Windows-1252 encoding; UTF-8 em-dash (E2 80 94) becomes 0x94 which PS tokenizer interprets as string delimiter

## Problems Solved
- **WSL bash mkdir creating directories in WSL filesystem instead of Windows**: Switched to PowerShell.exe -Command New-Item -ItemType Directory -Force -Path
- **Em-dash in .ps1 files causing PowerShell parsing failures on French Windows 11**: Replace all non-ASCII characters with ASCII equivalents (em-dash → hyphen) in .ps1 files
- **Claude Code session export using wrong project path (D-lab-Claudi)**: Updated export-sessions.ps1 with correct ProjectId and ArchivePath to D:\aidirectory\data\sessions

## Errors Encountered
- [resolved] Exit code 127: 'Item' command not found when using Bash tool for directory creation -> Use PowerShell.exe wrapper instead of Bash for Windows native operations
- [resolved] PowerShell parsing failure due to em-dash in .ps1 files -> Replace non-ASCII characters with ASCII equivalents

## Patterns Identified
- Windows 11 PS 5.1 with French locale uses Windows-1252 encoding, breaking UTF-8 files with non-ASCII chars
- Bash tool lacks Windows native commands (schtasks, cmd.exe) and mangles Windows paths
- WSL mkdir creates directories in WSL filesystem, not Windows filesystem

## Files Modified
- modified: aidirectory/.gitignore -- Added data/ to exclude operational data from git tracking
- created: aidirectory/data/sessions/ -- Created directory for raw Claude Code session exports
- created: aidirectory/data/queue/pending/ -- Created directory for pending session extracts
- created: aidirectory/data/queue/done/ -- Created directory for processed session extracts
- created: aidirectory/data/audits/ -- Created directory for audit logs
- created: aidirectory/data/proposals/ -- Created directory for draft suggestions before human review
- created: aidirectory/knowledge/extracts/ -- Created directory for processed session extracts (git-tracked)
- created: aidirectory/knowledge/skills/ -- Created directory for generated/updated skill files (git-tracked)
- modified: aidirectory/projects/workflow-lab/claude/export-sessions.ps1 -- Updated ProjectId and ArchivePath to correct values, added Start-Transcript logging
- created: aidirectory/scripts/session-audit.ps1 -- Created session audit script for end-of-session hook
- created: aidirectory/knowledge/audits/AUDIT_TEMPLATE.md -- Created template for session audit files
- created: aidirectory/claude/commands/audit -- Created audit slash command for Claude Code
- created: aidirectory/claude/commands/wrap -- Created wrap slash command for Chat/Cowork session exports
- created: aidirectory/claude/commands/handover -- Created handover command for session context transfer
- modified: aidirectory/MEMORY.md -- Updated directory layout and routing table for new structure
- modified: aidirectory/memory/directory_structure.md -- Rewritten to reflect new data/ folder structure
- modified: aidirectory/knowledge/INDEX.md -- Added Audits, Extracts, Skills sections
- modified: aidirectory/CLAUDE.md -- Updated routing table to cover new file types and navigate new sections

## Next Session Must Know
- Phase 1 complete: data/ folder structure created, gitignore updated, knowledge repo live at github.com/Danjkboks/ai-kb
- NEVER use em-dash or non-ASCII characters in .ps1 files on Windows 11 PS 5.1
- Use PowerShell.exe -Command New-Item for Windows directory creation, not bash mkdir
- Scheduled task 'aidirectory-session-export' runs daily at 13:00 with 10min boot delay
- Obsidian Git plugin auto-syncs knowledge/ every 10min
- Claude Code project folders: claude projects D:\aidirectory\projects\claudi (old stale path: D-lab-Claudi)
- Phase 2 trigger: wait 2+ weeks for extracts in data/queue/pending/, then design watcher workflow
- All .ps1 files must be ASCII-safe to avoid parsing failures on French Windows locale

## Skill Candidates
- windows-ps1-safe-write: Creates .ps1 files with ASCII-only characters to avoid encoding issues on French Windows 11
- scheduled-task-registration: Registers Windows scheduled tasks with proper encoding and elevation handling

## Token Waste Flags
- Repeated reading of PLAN.md content multiple times
- Generated directory creation commands that failed due to WSL/Bash issues

## Knowledge Base Updates
- [create] guide_windows_ps1-encoding.md: Document the em-dash encoding bug and ASCII-safe requirement for PowerShell scripts on French Windows 11
- [create] sop_claude-code_session-audit-hook.md: Document the session audit system, wrap command, and handover workflow implemented in Phase 1
- [update] ref_memory_directory-structure.md: Update with new data/ folder structure and git tracking strategy decisions

## Links
related:: [[_INDEX]]
tags: claude-code, memory, git, windows
