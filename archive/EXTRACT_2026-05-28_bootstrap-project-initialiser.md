---
type: extract
date: 2026-05-28
session_id: bootstrap-project-initialiser
surface: claude-code
environment: env1-claude-desktop
topics: [memory, infra]
source_file: 2026-06-05-095450_da2bbde1-c919-4774-862f-d8fab8afb381.jsonl
processed_at: 2026-06-05T08:55:22.477Z
---

# Session Extract: bootstrap-project-initialiser

## Decisions
- **Use the 3-section HANDOVER.md architecture (CHAT/COWORK/CODE + SHARED) for all new projects**: Redesigned to enforce merge/overwrite rules and provide clear separation of surface responsibilities
- **Enforce rule: never create a new agent if existing one in D:\aidirectory\agents\ covers the use case**: Prevents agent proliferation and ensures reuse of established capabilities

## Problems Solved
none

## Errors Encountered
none

## Patterns Identified
- Project bootstrap follows strict template: CLAUDE.md + HANDOVER.md stub + PowerShell folder creation
- Surface planning table (Phase | Surface) used to allocate work across Claude surfaces

## Files Modified
none

## Next Session Must Know
- HANDOVER.md uses 3-section architecture: CHAT/COWORK/CODE + SHARED
- Rule: never create new agent if existing one in D:\aidirectory\agents\ covers use case
- Project bootstrap creates: CLAUDE.md (complete template), HANDOVER.md (stub), folder structure via PowerShell
- Surface planning table allocates work: Ideation→Chat, Scaffold→Claude Code, Implementation→Claude Code, Docs/Cowork→Cowork
- Critical URLs: Cloudflare tunnel endpoints change on restart, LLMLingua host docker:5001
- Fragilities: GitHub commit node returns 422 on re-run, DeepSeek hallucinates dates (not system clock)
- Phase 3 Qdrant re-index weekly synthesis of 2 weeks extracts is pending
- Session-watcher (aidirectory-watcher.ps1) runs on logon as hidden window with no elevation

## Skill Candidates
- project-bootstrap: Creates new project folder structure with CLAUDE.md and HANDOVER.md files following standardized templates

## Token Waste Flags
none

## Knowledge Base Updates
- [update] sop_memory_project-bootstrap.md: Session demonstrates standardized project initialization process with CLAUDE.md/HANDOVER.md templates and PowerShell folder creation
- [update] ref_memory_handover-architecture.md: 3-section HANDOVER.md architecture (CHAT/COWORK/CODE + SHARED) is now enforced for all projects

## Links
related:: [[_INDEX]]
tags: memory, infra
