---
type: extract
date: 2026-05-27
session_id: itsm-document-clarification-non-technical-readers
surface: claude-code
environment: env1-claude-desktop
topics: [documentation, html, itsm]
source_file: 2026-06-04-120808_12b2c88a-c46e-483d-8dab-de1fbe529135.jsonl
processed_at: 2026-06-05T09:20:17.562Z
---

# Session Extract: itsm-document-clarification-non-technical-readers

## Decisions
- **Create a new version (v4.0) of the ITSM EVSM HTML document with non-technical reader clarifications**: The original document contains technical jargon, symbols, and abbreviations that are difficult for non-technical readers to understand. A clarified version will improve accessibility and comprehension.
- **Keep technical component names but add simple French explanations in gray text**: Technical component names (REF_COLLEGE, AUTO_DEMANDEUR, etc.) need to be preserved for system consistency, but adding simple explanations makes them understandable to non-technical readers.
- **Replace ON_ENTER state descriptions with plain French formulations**: ON_ENTER (Ouvert) → ouverture, ON_ENTER (En cours) → démarrage traitement, etc. makes the lifecycle states more intuitive for non-technical readers.

## Problems Solved
- **ITSM EVSM document contains technical jargon and symbols (§, REF_COLLEGE, AUTO_DEMANDEUR) that are inaccessible to non-technical readers**: Create v4.0 with: 1) Remove parasitic symbols (§), 2) Add simple French explanations for technical terms, 3) Replace ON_ENTER formulations with plain French, 4) Clarify abbreviations, 5) Add 'Key Figures' section with simple cards
- **Document lacks context for ratio explanation (22,414 distinct designations · 42,564 tickets · average ratio 1:1.9)**: Add explanatory box: 'On average, each request label covers 1.9 tickets. Agents write requests differently - problem: 22,000 unique labels for 42,000 tickets. Structured catalog: each request has a fixed title.'

## Errors Encountered
none

## Patterns Identified
- Technical documentation often needs parallel non-technical versions for different audiences
- Preserving technical identifiers while adding plain-language explanations maintains system compatibility while improving accessibility
- Visual design consistency (colors, typography, card styles) should be maintained across document versions

## Files Modified
- modified: aidirectory/ITSM/itsm_evsm_v3_1.html -- Created v4.0 with non-technical clarifications: removed symbols, added French explanations, simplified lifecycle states, clarified abbreviations, added Key Figures section

## Next Session Must Know
- ITSM EVSM v4.0 document created at aidirectory/ITSM/itsm_evsm_v4_0.html
- Key changes: 1) Removed § symbols, 2) Added French explanations for technical terms, 3) Simplified ON_ENTER states, 4) Clarified abbreviations, 5) Added Key Figures section with 3-column cards
- Technical component names preserved (REF_COLLEGE, AUTO_DEMANDEUR, etc.) with added gray text explanations
- Visual design maintained from v3.1 (colors, typography, card styles) for consistency
- Navigation arrows (OUVERT → COURS) preserved for visual flow
- Badges (CONSISTENT, SPLIT, FUSION) removed, but technical identifiers (W-xx, M-x) kept for reference
- HTML validated and saved without errors

## Skill Candidates
- document-technical-to-plain: Convert technical documentation to plain language for non-technical audiences while preserving technical accuracy
- html-document-clarify: Clean HTML documents by removing parasitic symbols, adding explanations for technical terms, and improving readability

## Token Waste Flags
- Repeated full HTML structure display in transcript
- Tool search results included extensive skill listings unrelated to the document clarification task

## Knowledge Base Updates
- [create] guide_documentation_technical-to-plain.md: Session demonstrated a clear pattern for converting technical documentation to plain language while preserving technical accuracy - should be documented as a reusable process
- [update] ref_itsm_evsm-documentation.md: New v4.0 document created with non-technical clarifications - should be referenced in ITSM documentation

## Links
related:: [[_INDEX]]
tags: documentation, html, itsm
