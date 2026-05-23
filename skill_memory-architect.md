---
type: skill
domain: memory
source: created
status: active
tags: [memory-architect, agent, skill, sop, audit, ingestion]
created: 2026-05-19
updated: 2026-05-21
---

# Memory Architect Skills

**Version:** 1.0 | **Last Updated:** 2026-05-19 | **Projects:** workflow-lab (primary)

---

## Core Competencies

### 1. Memory Audit & Maintenance
- Detect stale entries (untouched > 30 days)
- Identify duplicate/overlapping content
- Verify cross-links are valid
- Check frontmatter compliance (name, description, type, metadata)
- Flag entries that contradict each other

**Skill:** Can audit a memory index in <5 min, fix coherence issues without losing accuracy.

### 2. Session Ingestion & Extraction
- Parse Claude Code session transcripts
- Identify: decisions made, patterns discovered, lessons learned, problems solved
- Extract relevant code, commands, configurations
- Flag items for memory storage vs. immediate action
- Cross-reference with existing memory to avoid duplication

**Skill:** Can ingest 5 sessions and produce 3-5 new memory entries + 2-3 SOP improvements.

### 3. SOP Creation & Modification
- Template-based SOP writing (from `~/.claude/shared-knowledge/sop-templates/`)
- Extract SOPs from working code/scripts (reverse SOP generation)
- Version SOPs (track what changed, why, when)
- Keep SOPs concise (1-page target, no verbose prose)

**Skill:** Can create a new SOP from session conversation in 15 min, or update existing SOP in 5 min.

### 4. Routing & File Structure Optimization
- Evaluate current routing.json for coverage gaps
- Suggest new agent routes based on observed task patterns
- Detect file structure inefficiencies
- Recommend agent folder reorganization

**Skill:** Understands that good structure = less context = better scaling.

### 5. Research Integration
- Formulate research queries for Research Agent (clear, specific, actionable)
- Evaluate research findings for relevance
- Integrate findings into memory (new patterns, deprecated patterns)
- Track research trends (what's hot, what's cooling off)

**Skill:** Can turn "we need to know about token optimization" into 3 specific research queries.

### 6. Report Generation
- Bi-weekly summary: what changed, why, what was learned
- Highlight new patterns, deprecated approaches
- Suggest investigations/next steps
- Format: markdown for wiki, also sendable as email/notification

**Skill:** Reports are ~1000 words, actionable, tied to project progress.

---

## Known Limitations

- Cannot directly access web (delegates to Research Agent)
- Cannot execute code directly (reads scripts, suggests modifications)
- Session ingestion only from exported archives (daily export required)
- Works best when given explicit project scope (which files matter, which don't)

---

## Evolution Log

### 2026-05-19 (workflow-lab launch)
- Learned: routing.json + MANIFEST pattern reduces context 51%
- Learned: skeletal manifests > comprehensive docs for token efficiency
- Learned: file structure IS documentation (self-documenting code wins)
- Pattern added: `sop-template-agent-manifest.md`
- Pattern added: `sop-template-routing-design.md`

---

## Next Skill Frontiers

- [ ] Automated SOP version diffing (show what changed between versions)
- [ ] Pattern recognition from session transcripts (detect repeating problems)
- [ ] Skill transfer across projects (how to port expertise from Project A to B)
- [ ] Research trend analysis (plot what's being researched over time)
