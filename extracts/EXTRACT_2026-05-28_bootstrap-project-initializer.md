---
type: extract
date: 2026-05-28
session_id: bootstrap-project-initializer
surface: claude-code
environment: env1-claude-desktop
topics: [memory, claude-code, project-management]
source_file: 2026-06-05-095450_da2bbde1-c919-4774-862f-d8fab8afb381.jsonl
processed_at: 2026-06-05T10:18:54.453Z
---

# Session Extract: bootstrap-project-initializer

## Decisions
- **Use structured project initialization with CLAUDE.md and HANDOVER.md templates**: Standardizes project setup across all sessions and ensures proper documentation from the start
- **Implement surface-based task routing (Chat/Cowork/Code) for project planning**: Optimizes workflow by matching tasks to appropriate Claude interfaces based on their nature

## Problems Solved
- **Unstructured project setup leading to inconsistent documentation**: Created bootstrap project initializer with standardized CLAUDE.md and HANDOVER.md templates

## Errors Encountered
none

## Patterns Identified
- Project classification by type (coding/automation/research/mixed), duration (one-shot/multi-session/ongoing), and agent needs
- Surface-based task routing: Chat for ideation, Claude Code for implementation, Cowork for docs/artifacts
- Never create new agent if existing one covers use case (check aidirectory/agents MANIFEST first)

## Files Modified
none

## Next Session Must Know
- Project initialization template includes: Type classification, Surface routing table, Stack definition, Non-negotiables
- HANDOVER.md structure: Updated date, Goal, Current status, Key decisions, Next step
- Never create agent if existing one covers use case - check aidirectory/agents MANIFEST first
- Project types: coding/automation/research/mixed, Duration: one-shot/multi-session/ongoing
- Surface routing: Chat (ideation, cheap), Claude Code (implementation, git), Cowork (docs, artifacts, recurring tasks)
- Research projects get notes/ not src/ folder structure
- PowerShell commands for project creation: New-Item -ItemType Directory -Path
- Session memory system saves 67% tokens through progressive loading and compact encoding

## Skill Candidates
- project-bootstrap: Creates standardized project structure with CLAUDE.md and HANDOVER.md templates
- surface-task-router: Analyzes project requirements and recommends optimal surface split (Chat/Cowork/Code)

## Token Waste Flags
- Repeated full skill listing (85 skills) at session start

## Knowledge Base Updates
- [create] sop_claude-code_project-initialization.md: Standardizes project bootstrap process with CLAUDE.md and HANDOVER.md templates
- [create] guide_project_surface-routing.md: Documents surface-based task routing methodology (Chat/Cowork/Code) for optimal workflow

## Links
related:: [[_INDEX]]
tags: memory, claude-code, project-management
