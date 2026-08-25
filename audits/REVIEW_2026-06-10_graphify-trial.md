---
type: review
slug: graphify-trial
date: 2026-06-10
scope:
  - ~/.claude/skills/graphify/SKILL.md (WSL2)
  - D:\aidirectory\scripts\graphify-out\GRAPH_REPORT.md
  - D:\aidirectory\agents\graphify-out\GRAPH_REPORT.md
context_loaded:
  - CLAUDE.md
  - knowledge/_INDEX.md
  - HANDOVER.md
---

# Review — Graphify Trial (2026-06-10)

## Execution Summary

| Step | Result |
|---|---|
| SKILL.md install (curl) | ✅ 1207 lines written to WSL2 `~/.claude/skills/graphify/SKILL.md` |
| graphifyy pip install | ✅ v0.8.36 installed (WSL2, `~/.local/bin` PATH fix required) |
| graphify --version | ✅ 0.8.36 confirmed |
| graphify scripts/ --no-viz | ✅ 129 nodes · 152 edges · 36 communities — GRAPH_REPORT.md written |
| graphify agents/ --no-viz | ✅ 74 nodes · 70 edges · 13 communities — OpenRouter/deepseek-chat, ~$0.002 |

---

## Finding 1 — SKILL.md is in the wrong location for this environment

**Severity: HIGH**

The plan's Phase 1 install command runs in WSL2:
```bash
mkdir -p ~/.claude/skills/graphify
curl ... > ~/.claude/skills/graphify/SKILL.md
```

This writes to `/home/gnren-pc/.claude/skills/graphify/SKILL.md` (WSL2 filesystem).

Claude Code on this machine runs on **Windows**, not WSL2. Its skills directory is:
```
C:\Users\GnReN-PC\.claude\skills\
```

The graphify skill is **not present** there. The `/graphify` trigger will not activate in any Windows Claude Code session. All existing n8n-skills and memory-bank are correctly installed under the Windows path.

**Fix required before Phase 4:**
```powershell
New-Item -ItemType Directory -Force "C:\Users\GnReN-PC\.claude\skills\graphify"
wsl -d Ubuntu bash -l -c 'cat ~/.claude/skills/graphify/SKILL.md' | Out-File -Encoding UTF8 "C:\Users\GnReN-PC\.claude\skills\graphify\SKILL.md"
```

---

## Finding 2 — scripts/ output contains real structural signal

**Verdict: PASS**

24 code files processed (9 Python + 15 PowerShell). 129 nodes, 152 edges, 36 communities. Zero LLM cost. Zero tokens.

**God nodes (highest betweenness centrality):**
| Node | Edges | What it reveals |
|---|---|---|
| `WorkflowExtractor` | 22 | Core of workflow indexation — bridges 13 communities |
| `WorkflowSearch` | 9 | Search layer hub across index files |
| `WorkflowMigrator` | 6 | Migration pipeline bottleneck |
| `jsonl_to_transcript()` | 4 | Session processing bridge |

**_INDEX.md vs GRAPH_REPORT.md — are they duplicate?**
No. _INDEX.md is file-level metadata (what each file is for). GRAPH_REPORT.md is function/class-level call structure (who calls what, which functions are cross-community bridges). They are complementary, not overlapping.

**Surprising connections found:**
- `main() --calls--> WorkflowExtractor` bridges community 7 → community 6. This cross-script dependency is not visible in _INDEX.md and would not be surfaced by Qdrant semantic search (which retrieves by content similarity, not call structure).

**Community quality note:** All 36 communities are named "Community N" because no LLM key was provided for the labeling pass. This limits human readability but the structural graph (graph.json) is fully correct. Running `graphify label scripts/` with a DeepSeek backend via OpenRouter would name all communities for ~$0.01.

---

## Finding 3 — agents/ mapped via OpenRouter/DeepSeek (Pass 2 enabled)

**74 nodes · 70 edges · 13 communities — all community names LLM-labeled · ~$0.002 actual cost**

agents/ corpus: 9 code files (`.ps1` + `.py` + `.json`) + 8 doc files (6× MANIFEST.md + project-context.json + proposals).

**God nodes:**
| Node | Edges | What it reveals |
|---|---|---|
| `configuration` | 10 | Shared schema field across all agent project-context.json files |
| `state` | 9 | Cross-agent state tracking hub |
| `check_n8n()` | 6 | n8n API wrapper bottleneck in create-workflow.py |
| `triggers` | 5 | Agent trigger system (audit_due, new_sessions_detected, report_due) |

**Surprising connections (INFERRED, confidence 0.8):**
- `Docker Operations Agent` --shares_data_with--> `n8n Workflow Agent`
- `Infrastructure Operations Agent` --shares_data_with--> `Docker Operations Agent`
- `Python Operations Agent` --shares_data_with--> `Docker Operations Agent`
- `Git Operations Agent` --shares_data_with--> `Python Operations Agent`
- `Infrastructure Operations Agent` --shares_data_with--> `n8n Workflow Agent`

These inter-agent relationships are inferred from shared terminology in MANIFEST.md files. The ROUTING table in CLAUDE.md documents them explicitly, but graphify independently surfaced the infra-ops → docker-ops → n8n dependency chain, which is the correct blast-radius order for container/tunnel changes.

**Community "n8n API Wrapper" (Community 5, cohesion 0.39):** surfaces `check_n8n()`, `activate_workflow()`, `create_workflow()`, `get_executions()`, `list_workflows()` as a cohesive unit from `agents/n8n-workflow/create-workflow.py`. This is the only Python code in agents/ and graphify correctly isolated it.

**45 isolated nodes:** mostly JSON key names from project-context.json and queue.json (project, agent, version, created_at, etc.). Expected — JSON field names without call relationships generate no edges.

**OpenRouter provider registration:** Required one-time setup via `graphify provider add openrouter --base-url https://openrouter.ai/api/v1 --default-model deepseek/deepseek-chat --env-key OPENROUTER_API_KEY`. Config stored in `~/.graphify/providers.json` (WSL2).

---

## Finding 4 — No conflict with Qdrant / _INDEX.md

Qdrant indexes knowledge/extracts/ and knowledge/ vault content — semantic chunks from session extracts and SOPs. The graphify graph maps structural Python/PS1 call relationships in scripts/ and agents/. These are orthogonal:
- Qdrant answers: "what knowledge do we have about X?"
- graphify answers: "which scripts call which functions, and which are architectural bottlenecks?"

No duplication risk. The proposed /integrity pre-step (read GRAPH_REPORT.md before examining session extracts) would add structural context that Qdrant cannot provide.

---

## Finding 5 — WSL2 path and tooling friction

Issues encountered during Phase 1–2:

| Issue | Impact | Status |
|---|---|---|
| `pip` not installed in WSL2 Ubuntu | Blocked install 10+ min | Fixed via `pip3 install` |
| `~/.local/bin` not in PATH | `graphify --version` failed | Fixed via `~/.bashrc` |
| curl timeout in WSL2 (get-pip bootstrap) | Failed download | Worked around via apt |
| `\| head -30` killed pip mid-install | Install never completed first attempt | Re-ran without pipe |

None of these are graphify bugs — all are WSL2 environment issues. Add a WSL2 pip/PATH check to any future skill SOP.

---

## Phase 3 Gate Evaluation

Per the plan: PASS if 2+ of the following are YES.

| Question | Answer |
|---|---|
| Does it surface call relationships _INDEX.md does NOT show? | **YES** — function-level, community bridges, god nodes. _INDEX.md has none of this. |
| Does it find non-obvious chains between PS1 scripts and agents? | **YES** — PS1 scripts are in the scripts/ graph (15 of 24 files). agents/ graph independently surfaced the infra-ops → docker-ops → n8n dependency chain matching CLAUDE.md routing order. Cross-boundary PS1↔Python edges not found (they run via subprocess, not import — AST can't trace that). |
| Would feeding this into /integrity add signal or just noise? | **YES** — god node analysis (`WorkflowExtractor` as 13-community bridge in scripts; `check_n8n()` as n8n wrapper bottleneck in agents) is exactly the structural fragility signal /integrity should surface. Both graphs have zero import cycles. |

**Result: 2.5/3 → PASS**

---

## Recommendations Before Phase 4

1. **[REQUIRED]** Copy SKILL.md to Windows Claude Code path (Finding 1).
2. **[OPTIONAL]** Run `graphify label scripts/ --backend=deepseek` with OpenRouter key to name the 36 communities. ~$0.01. Makes /integrity integration 10× more readable.
3. **[DONE]** agents/ mapped. OpenRouter provider config is persistent in WSL2 `~/.graphify/providers.json` — future runs just need `--backend=openrouter` with `.env` sourced.
4. **[LOW]** Add `export PATH="$HOME/.local/bin:$PATH"` to WSL2 env-launch.ps1 wrapper or document in the graphify SOP.

---

## What Phase 4 Should Look Like (if proceeding)

Add to `.claude/commands/integrity.md` pre-step:

```
Before running integrity analysis:
1. Check if D:\aidirectory\scripts\graphify-out\GRAPH_REPORT.md exists.
2. If present, read God Nodes and Surprising Connections sections only (skip community detail).
3. Use god node list to identify which scripts are highest-risk change targets.
4. Note any bridges between communities that touch the component under review.
```

Do NOT read the full GRAPH_REPORT.md — it is 105 lines but community detail is noise for integrity checks. The God Nodes and Surprising Connections sections (~20 lines) are sufficient.
