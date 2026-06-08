---
type: extract
date: 2026-06-05
session_id: itsm-evsm-document-clarification-non-technical
surface: claude-code
environment: env1-claude-desktop
topics: [memory, documentation]
source_file: 2026-06-05-210709_12b2c88a-c46e-483d-8dab-de1fbe529135.jsonl
processed_at: 2026-06-07T16:03:02.431Z
---

# Session Extract: itsm-evsm-document-clarification-non-technical

## Decisions
- **Create separate ITSM Plan.md file instead of overwriting HANDOVER.md**: To maintain HANDOVER.md structure rules while preserving detailed project context for future sessions
- **Use light professional theme for ITSM EVSM document**: Better readability for non-technical readers and printability
- **Keep CSS pure (no D3/graphviz dependencies) for EVSM interactive map**: Lighter, more maintainable, no external dependencies

## Problems Solved
- **HTML document had parasitic symbols (§) and technical jargon incomprehensible to non-technical readers**: Removed § symbols, replaced technical acronyms with French explanations, simplified navigation text
- **Color badges (pastilles) were stripped leaving empty spans in HTML**: Restored 6 badges with inline CSS display:block, width:10px, border-radius:50%, and added zero-width space (​) to prevent future stripping

## Errors Encountered
- [resolved] Context remaining at 0-6% during session, triggering emergency evacuation -> Used compact context monitoring and immediate handover procedures

## Patterns Identified
- When context is low (<10%), immediately write to HANDOVER.md or create separate plan file
- HTML badge stripping can be prevented with zero-width space characters
- Non-technical document clarification requires: remove symbols, explain acronyms, simplify navigation, add key figures section

## Files Modified
- modified: aidirectory/Projects/ITSM/itsm_evsm_v4_0.html -- Clarified for non-technical readers: removed § symbols, explained technical acronyms in French, simplified navigation, added key figures section, restored color badges
- created: aidirectory/Projects/ITSM/Plan.md -- Created project plan with context, current state, inventory, prioritized tasks P1-P8, open questions, and architectural decisions

## Next Session Must Know
- ITSM EVSM document at v4.0 with light professional theme - located at aidirectory/Projects/ITSM/itsm_evsm_v4_0.html
- Project plan created at aidirectory/Projects/ITSM/Plan.md with 8 prioritized tasks (P1-P8)
- P1 priority: Create 15 missing popup cards for workflows W-03, W-05, W-09, W-11, W-16, W-18, W-20
- P2 priority: Connect orphaned W-xx workflows to catalog tree nodes
- Color badges restored with zero-width space (​) to prevent stripping - CSS classes: .catalogue, .fusion-recommandee, etc.
- Key architectural decisions documented: light theme, pure CSS, lateral popups, normalized resolution groups
- Open questions: Should W-11 be removed (merged with W-04 as M-1)? Distinguish user-facing vs back-office branches?

## Skill Candidates
- document-clarification-non-technical: Transform technical documents for non-technical audiences by removing symbols, explaining acronyms, simplifying navigation, adding key figures
- emergency-context-handover: When context is critically low (<10%), immediately create structured handover or plan file before evacuation

## Token Waste Flags
- Repeated context emergency warnings without immediate action
- Large tool listing deltas loaded at session start

## Knowledge Base Updates
- [create] guide_documentation_non-technical-clarification.md: Session demonstrated repeatable pattern for clarifying technical documents: remove symbols, explain acronyms, simplify navigation, add key figures section, restore UI elements
- [update] sop_memory_handover-procedures.md: Add emergency context procedure: when <10% remaining, create separate plan file instead of overwriting HANDOVER.md to preserve project context

## Links
related:: [[_INDEX]]
tags: memory, documentation
