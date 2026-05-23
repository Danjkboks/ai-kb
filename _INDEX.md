---
type: index
description: Master navigation map for D:\aidirectory\knowledge\ — flat structure, named by type_domain_topic
updated: 2026-05-21
---

# Knowledge Vault — Master Index

> **Naming convention:** `{type}_{domain}_{topic}.md`
> **Types:** `sop_` · `ref_` · `guide_` · `runbook_` · `report_` · `audit_` · `skill_` · `log_`
> **Domains:** `claude-code` · `docker` · `n8n` · `python` · `infra` · `memory` · `llm` · `security` · `stack`
> **Source field:** `session` = from a Claude Code session · `imported` = from Perplexity/web · `external` = external reference · `created` = manually authored

---

## SOPs — Step-by-step procedures

| File | What it covers |
|---|---|
| [sop_claude-code_full-environment-setup.md](sop_claude-code_full-environment-setup.md) | Complete Claude Code + VS Code + OpenRouter setup on Windows 11 |
| [sop_claude-code_vscode-openrouter-windows.md](sop_claude-code_vscode-openrouter-windows.md) | Claude Code + VS Code + OpenRouter (Windows, validated) |
| [sop_claude-code_skill-installation.md](sop_claude-code_skill-installation.md) | Install n8n-skills, cc-safe-setup hooks, memory-bank globally |
| [sop_docker_container-management.md](sop_docker_container-management.md) | Start/stop/rebuild all stack containers, env-launch.ps1 usage |
| [sop_infra_cloudflare-tunnel.md](sop_infra_cloudflare-tunnel.md) | Cloudflare quick tunnel for n8n and Langfuse — URL changes on restart |
| [sop_infra_langfuse-setup.md](sop_infra_langfuse-setup.md) | Deploy Langfuse, connect OpenRouter Broadcast, SDK integration |
| [sop_memory_obsidian-github-sync.md](sop_memory_obsidian-github-sync.md) | Obsidian + GitHub as version-controlled agent memory |
| [sop_n8n_agent-bridge.md](sop_n8n_agent-bridge.md) | Expose n8n webhooks to external agents via Cloudflare tunnel |
| [sop_n8n_mcp-setup.md](sop_n8n_mcp-setup.md) | n8n instance-level MCP server — JWT config, .mcp.json, Claude Code integration |
| [sop_python_flask-api-llmlingua.md](sop_python_flask-api-llmlingua.md) | api_with_langfuse.py — compression API, endpoints, rebuild, prompt caching |
| [sop_python_qdrant-embeddings.md](sop_python_qdrant-embeddings.md) | Index workflows + wiki into Qdrant for semantic search |
| [sop_python_scrapegraphai-wiki-ingestion.md](sop_python_scrapegraphai-wiki-ingestion.md) | Autonomous ScrapeGraphAI → Obsidian wiki ingestion pipeline |

---

## References — Frameworks, decision trees, lookup tables

| File | What it covers |
|---|---|
| [ref_llm_model-selection.md](ref_llm_model-selection.md) | When to use Haiku / Sonnet / Opus — decision tree + cost |
| [ref_security_agent-framework.md](ref_security_agent-framework.md) | Security standards for all agents — permissions, audit, kill switch |
| [ref_llm_agent-models-2026.md](ref_llm_agent-models-2026.md) | AI agent model landscape 2026 — comparison and use cases |
| [ref_llm_claude-prompt-shortcuts.md](ref_llm_claude-prompt-shortcuts.md) | 100 Claude prompt shortcuts by category (/ghost, /mirror, /raw…) |
| [ref_stack_n8n-indexation-architecture.md](ref_stack_n8n-indexation-architecture.md) | Architecture of the n8n workflow indexation system |

---

## Guides — Conceptual explanations, how things work

| File | What it covers |
|---|---|
| [guide_stack_self-hosted-ai.md](guide_stack_self-hosted-ai.md) | Complete self-hosted AI stack guide — external reference |
| [guide_memory_ai-architecture.md](guide_memory_ai-architecture.md) | AI memory architecture patterns — RAG, embeddings, vector DBs |
| [guide_memory_claude-obsidian.md](guide_memory_claude-obsidian.md) | Claude + Obsidian integration guide |
| [guide_memory_second-brain-fr.md](guide_memory_second-brain-fr.md) | Second brain with Claude + Obsidian (French) |

---

## Runbooks — Operational references

| File | What it covers |
|---|---|
| [runbook_stack_master-build.md](runbook_stack_master-build.md) | Full stack build reference — all components, versions, commands |

---

## Skills — Agent skill definitions

| File | What it covers |
|---|---|
| [skill_memory-architect.md](skill_memory-architect.md) | Memory Architect agent capabilities — audit, SOP creation, session ingestion |
| [skill_research-agent.md](skill_research-agent.md) | Research Agent capabilities — web research, doc analysis, trend detection |

---

## Logs — Evolution and expertise logs

| File | What it covers |
|---|---|
| [log_memory-architect-expertise.md](log_memory-architect-expertise.md) | What Memory Architect has learned across sessions |

---

## Reports — Phase completions, deployments

| File | What it covers |
|---|---|
| [report_2026-05-19_workflow-indexation-deployment.md](report_2026-05-19_workflow-indexation-deployment.md) | Workflow indexation deployment sign-off (all 4 phases) |
| [report_2026-05-19_workflow-indexation-phases.md](report_2026-05-19_workflow-indexation-phases.md) | Detailed phase completion report with metrics |

---

## Audits

| File | What it covers |
|---|---|
| [audit_2026-05-19_phase-1.md](audit_2026-05-19_phase-1.md) | Phase 1 audit — metadata extraction |
| [audit_2026-05-19_phase-2.md](audit_2026-05-19_phase-2.md) | Phase 2 audit — search interface |
| [audit_2026-05-19_phase-3.md](audit_2026-05-19_phase-3.md) | Phase 3 audit — workflow migration |
| [audit_2026-05-19_phase-1-3-fixes.md](audit_2026-05-19_phase-1-3-fixes.md) | Fixes applied across phases 1-3 |

---

## Quick Navigation

```
Need to...                                    → File
──────────────────────────────────────────────────────────────────
Set up Claude Code from scratch              → sop_claude-code_full-environment-setup
Install skills (n8n-skills, cc-safe-setup)   → sop_claude-code_skill-installation
Start all containers                         → sop_docker_container-management
Start Cloudflare tunnel                      → sop_infra_cloudflare-tunnel
Use n8n MCP with Claude Code                 → sop_n8n_mcp-setup
Use the Flask compression API                → sop_python_flask-api-llmlingua
Re-index workflows/wiki in Qdrant            → sop_python_qdrant-embeddings
Choose model (Haiku/Sonnet/Opus)             → ref_llm_model-selection
Check agent security rules                   → ref_security_agent-framework
Understand full stack build                  → runbook_stack_master-build
```
