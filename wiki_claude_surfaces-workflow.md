# Claude Surfaces & Workflow Architecture
> Type: wiki | Domain: claude-desktop | Created: 2026-05-23
> Tags: #claude #surfaces #workflow #chat #cowork #claude-code #routing #bootstrap #CLAUDE.md

---

## What was built (2026-05-23 session)

| File | Path | Purpose |
|---|---|---|
| Global custom instruction | Claude Desktop Settings | Surface routing + identity, every Chat session |
| Global CLAUDE.md (updated) | `D:\aidirectory\CLAUDE.md` | Added ## Surfaces section |
| Project CLAUDE.md | `D:\aidirectory\projects\workflow-lab\CLAUDE.md` | Full project context |
| /bootstrap command | `D:\aidirectory\.claude\commands\bootstrap.md` | New project scaffold |
| HANDOVER.md stub | `D:\aidirectory\projects\workflow-lab\HANDOVER.md` | Session handoff template |

---

## Surface routing rules

| Task | Surface | Rule |
|---|---|---|
| Plan / design / research / rubber-duck | Chat | Cheapest. No file access needed. |
| Code / git / scripts / MCP / agents | Claude Code | Primary engine. |
| Office artifacts (.docx/.pptx/.xlsx) | Cowork | Only when Code can't format deliverables. |
| Scheduled / recurring tasks | Cowork | One-time setup. Don't re-prompt manually. |
| Browser / connector ops | Cowork | Connectors > Chrome > computer-use. |
| Quick question | Chat | Never burn Code/Cowork tokens on a question. |

---

## CLAUDE.md hierarchy

Claude Code loads CLAUDE.md files based on working directory — NOT all at once.

Working in `workflow-lab\` → loads: root CLAUDE.md + project CLAUDE.md (2 files only).
Other projects' CLAUDE.md files are never loaded unless you cd into them.
Rule: root ≤100 lines · project ≤80 lines · per file, not total.

---

## Cross-surface handoff conventions

- End of any session → write HANDOVER.md (goal · status · decisions · avoid · next step)
- Code → Code new session → @HANDOVER.md in CLAUDE.md auto-loads it
- Chat → Code → write decisions to PLAN.md, reference with @PLAN.md in project CLAUDE.md
- New project → /bootstrap "description" in Claude Code

---

## workflow-lab real goal

Replicable, deployable enterprise workflow auditing platform.
Audit company workflows per department/position/task → analyse → propose → build in n8n.
Foundation layer complete (indexation + search + migration tools).
Audit engine not yet started.

---

## Key decisions from this session

- "Auto-generate CLAUDE.md agent" = redundant (grade 4/10). Chat does it better with more context.
- Stack description does NOT belong in global custom instruction. Project-specific only.
- Persistent memory / RAG = NOT fully wired. Dedicated session needed to audit + fix.
- Suspicion: storage layers exist (Qdrant + knowledge/) but ingestion pipelines are missing.
