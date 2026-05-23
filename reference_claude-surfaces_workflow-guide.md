# reference_claude-surfaces_workflow-guide.md
> Type: reference | Domain: claude-surfaces | Created: 2026-05-23
> Tags: claude-code, cowork, chat, routing, surface-split, CLAUDE.md, bootstrap, handoff

---

## What this covers

Three-surface Claude workflow — Chat / Cowork / Claude Code — decision framework,
handoff conventions, and project bootstrap pattern. Decisions made 2026-05-23.

---

## Surface routing rules

| Task | Surface | Rule |
|---|---|---|
| Plan, design, research, rubber-duck | Chat | Cheapest. No file access needed. |
| Code, git, tests, scripts, MCP, agents | Claude Code | Primary engine. |
| Office artifacts (.docx/.pptx/.xlsx) | Cowork | Only if Code can't produce a formatted deliverable. |
| Scheduled / recurring tasks | Cowork | One-time setup via Cowork scheduler. |
| Browser scraping / connector ops | Cowork | Connectors > Chrome > computer-use (cost order). |
| Quick one-shot question | Chat | Never burn Code or Cowork tokens on a question. |

---

## CLAUDE.md hierarchy

Claude Code loads CLAUDE.md files based on working directory — NOT all projects at once.

```
C:\Users\GnReN-PC\.claude\CLAUDE.md     ← global user (always loaded)
D:\aidirectory\CLAUDE.md                ← root rules (loaded in all D:\aidirectory sessions)
D:\aidirectory\projects\[project]\CLAUDE.md  ← project-specific (loaded only when inside that project)
```

- Having 10 projects = 10 project CLAUDE.md files on disk, but only 2 load per session.
- Rule: root CLAUDE.md ≤ 100 lines, project CLAUDE.md ≤ 80 lines.
- Custom instruction in Claude Desktop Settings = Chat-side context (no file access, always active).

---

## Handoff conventions

- End of every session (any surface): write/update `HANDOVER.md` in the project root.
- Format: goal · current status · key decisions · what to avoid · next step.
- Reference in project CLAUDE.md with `@HANDOVER.md` so Claude Code picks it up automatically.
- Chat → Code: write decisions into `PLAN.md`, reference with `@PLAN.md` in CLAUDE.md.

---

## /bootstrap command

Location: `D:\aidirectory\.claude\commands\bootstrap.md`
Usage: `/bootstrap "description"` in any Claude Code session.

Generates:
1. Surface split table for the project
2. Ready-to-use project CLAUDE.md
3. HANDOVER.md stub
4. PowerShell folder scaffold commands

Use once per new project at the start. Never use on an existing project.

---

## What was built this session (2026-05-23)

| File | What it does |
|---|---|
| `D:\aidirectory\CLAUDE.md` | Added `## Surfaces` section — surface routing + handoff conventions |
| `D:\aidirectory\.claude\commands\bootstrap.md` | New project bootstrap slash command |
| `D:\aidirectory\projects\workflow-lab\CLAUDE.md` | Project-level CLAUDE.md (vision + stack + agents + surface split) |
| `D:\aidirectory\projects\workflow-lab\HANDOVER.md` | Handoff stub |
| Claude Desktop Settings | Custom instruction updated (lean global context) |
| Cowork | workflow-lab project created manually by user |

---

## Decisions made

- Project-bootstrap agent: **rejected** (redundant — Chat already does this with more context, less overhead).
- /bootstrap slash command: **built** (narrow, fixed output, no agent overhead).
- Self-hosted stack details: **removed from global custom instruction** (project-specific, belongs in project CLAUDE.md).
- workflow-lab project vision: enterprise workflow auditing platform (audit → analyse → propose → build in n8n). Replicable and deployable. Foundation layer (indexation) already complete.

---

## Next open item

Persistent memory / RAG architecture audit.
Goal: every session (Chat + Code + Cowork) auto-extracts learnings → tags → indexes into Qdrant.
Current state: Qdrant running, knowledge/ vault exists, memory-bank skill installed.
Gap suspected: no automatic ingestion pipeline from Chat → Qdrant.
SOPs exist but pipeline implementation unverified.
→ Dedicated session needed. Read SOPs + audit current wiring before proposing anything.
