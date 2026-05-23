---
type: sop
domain: claude-code
source: session
status: active
tags: [claude-code, windows, setup, openrouter, vscode, permissions]
created: 2026-05-12
updated: 2026-05-21
---

# SOP — Claude Code Full Environment Setup
## Workflow Lab | Danjkboks | Last updated: 2026-05-12

> **Purpose**: Complete setup guide for Claude Code + VS Code + OpenRouter on Windows 11.
> Covers CLI install, environment variables, CLAUDE.md hierarchy, and permissions files.
> Feed this file to the wiki/ folder so agents always have this context.

---

## Prerequisites

- Windows 11, PowerShell
- Node.js 18+ installed (via `nvm-windows` recommended)
- VS Code installed
- OpenRouter account with credits (`openrouter.ai`)
- GitHub account: `Danjkboks`
- Docker Desktop installed and running

---

## Step 1 — Install Claude Code CLI

```powershell
npm install -g @anthropic-ai/claude-code
claude --version
```

---

## Step 2 — Set Permanent Environment Variables

> ⚠️ "User" is a .NET keyword — do NOT replace it with your username.

```powershell
[System.Environment]::SetEnvironmentVariable("ANTHROPIC_BASE_URL", "https://openrouter.ai/api/v1", "User")
[System.Environment]::SetEnvironmentVariable("ANTHROPIC_API_KEY", "sk-or-v1-YOUR_KEY_HERE", "User")
[System.Environment]::SetEnvironmentVariable("ANTHROPIC_MODEL", "claude-haiku-3-5", "User")
```

Close PowerShell, reopen, then verify:

```powershell
[System.Environment]::GetEnvironmentVariable("ANTHROPIC_BASE_URL", "User")
[System.Environment]::GetEnvironmentVariable("ANTHROPIC_API_KEY", "User")
[System.Environment]::GetEnvironmentVariable("ANTHROPIC_MODEL", "User")
```

All three must return values.

---

## Step 3 — Delete the Hardcoded `.claude.json`

Claude Code auto-creates `%USERPROFILE%\.claude.json` on first launch with a hardcoded model
(e.g. `claude-opus-4-7`) that **overrides** your environment variables.

```powershell
# Optional backup
Copy-Item "$env:USERPROFILE\.claude.json" "$env:USERPROFILE\.claude.json.manual-backup"

# Delete it
Remove-Item "$env:USERPROFILE\.claude.json" -Force
```

> Claude Code recreates it automatically without the hardcoded model.
> If the problem returns after an update, repeat this step.

---

## Step 4 — Test the CLI

```powershell
cd D:\Claudi
claude --model claude-haiku-3-5 "say hello in one word"
```

If it responds, env vars are correctly loaded.

---

## Step 5 — Install the VS Code Extension

1. `Ctrl+Shift+X` → search **"Claude Code"** → install (publisher: Anthropic)
2. The extension auto-uses the already-configured CLI

---

## Step 6 — Force Model in VS Code

The extension may ignore `ANTHROPIC_MODEL` and fall back to Anthropic default.

Fix: **File → Preferences → Settings** → `{}` icon (Open Settings JSON) → add:

```json
{
  "claude.defaultModel": "claude-haiku-3-5"
}
```

Save (`Ctrl+S`), fully close and reopen VS Code.
Verify: bottom of Claude panel must show **"Claude Haiku 4.5"**.

---

## Step 7 — Open VS Code Terminal

```
Ctrl+` (backtick — key left of 1)
```

Opens PowerShell terminal at the bottom, already in workspace folder.

---

## Step 8 — Create the File Structure

```powershell
# Project workspace
New-Item -Path "D:\Claudi\.claude\rules" -ItemType Directory -Force

# Project settings files
New-Item -Path "D:\Claudi\.claude\settings.json" -ItemType File -Force
New-Item -Path "D:\Claudi\.claude\settings.local.json" -ItemType File -Force

# Global config folder (usually already exists)
New-Item -Path "$env:USERPROFILE\.claude" -ItemType Directory -Force
New-Item -Path "$env:USERPROFILE\.claude\CLAUDE.md" -ItemType File -Force
New-Item -Path "$env:USERPROFILE\.claude\settings.json" -ItemType File -Force
```

---

## Step 9 — Install Config Files

```powershell
# Copy downloaded files to correct locations
Copy-Item "$HOME\Downloads\1_global_CLAUDE.md" "$env:USERPROFILE\.claude\CLAUDE.md"
Copy-Item "$HOME\Downloads\2_global_settings.json" "$env:USERPROFILE\.claude\settings.json"
Copy-Item "$HOME\Downloads\3_project_settings.json" "D:\Claudi\.claude\settings.json"
Copy-Item "$HOME\Downloads\4_project_settings_local.json" "D:\Claudi\.claude\settings.local.json"
Copy-Item "$HOME\Downloads\CLAUDE.md" "D:\Claudi\CLAUDE.md"
```

Gitignore the local settings file:

```powershell
Add-Content "D:\Claudi\.gitignore" ".claude/settings.local.json"
```

---

## File Hierarchy Explained

Claude Code loads 4 layers on every session start, in this order:

| File | Scope | Location |
|---|---|---|
| `~\.claude\CLAUDE.md` | Global — all projects | `%USERPROFILE%\.claude\` |
| `~\.claude\settings.json` | Global permissions | `%USERPROFILE%\.claude\` |
| `D:\Claudi\CLAUDE.md` | This project only | Workspace root |
| `D:\Claudi\.claude\settings.json` | Project permissions (committed) | `.claude\` folder |
| `D:\Claudi\.claude\settings.local.json` | Personal overrides (gitignored) | `.claude\` folder |

---

## Global CLAUDE.md Content (`~\.claude\CLAUDE.md`)

```markdown
# Global Identity

## Who I Am
- Handle: Danjkboks | Location: Carpentras, France (EU)
- Role: Sysadmin student, beginner-to-intermediate in AI/automation
- Hardware: Gigabyte Aorus 16X — i7, 16GB RAM, RTX 4070 8GB (Windows 11)
- Secondary machine: older laptop, dual-boot PikaOS + Windows 10 (lab/test only)

## Budget
- Current ceiling: 40€/month
- Willing to increase if the setup generates measurable value or revenue
- Always show estimated cost before suggesting a paid service
- Flag anything above 10€/month as a line item

## Infrastructure Philosophy
- Self-hosted first — always check if it runs locally before going cloud
- Open to VPS (Hetzner, OVH, Scaleway — EU-hosted preferred) for always-on services
- Open to GPU rental (RunPod, Vast.ai) for one-off heavy inference tasks
- No AWS, GCP, or Azure — too expensive, too complex, wrong scale
- EU data residency preferred for anything touching personal or business data
- OpenRouter is the only LLM gateway — no direct provider API keys

## Universal Rules (Apply to Every Project)
- English only
- Terse. No preamble. No filler ("Great!", "Certainly!", "Sure thing!")
- Never commit `.env` files. Ever. No exceptions.
- Never suggest a solution that requires more than 3 setup steps without asking if simpler exists
- Simple solution first — complexity only on explicit request
- Flag cost with ⚠️ COST: prefix
- Flag destructive actions with ⚠️ DESTRUCTIVE: prefix
```

---

## Global settings.json Content (`~\.claude\settings.json`)

```json
{
  "permissions": {
    "allow": [
      "Bash(git status)",
      "Bash(git diff*)",
      "Bash(git log*)",
      "Bash(git add*)",
      "Bash(git commit*)",
      "Bash(git pull*)",
      "Bash(docker ps*)",
      "Bash(docker images*)",
      "Bash(docker logs*)",
      "Bash(echo*)",
      "Bash(cat*)",
      "Bash(ls*)",
      "Bash(Get-ChildItem*)",
      "Bash(python --version)",
      "Bash(node --version)",
      "Bash(claude --version)"
    ],
    "deny": [
      "Bash(rm -rf*)",
      "Bash(Remove-Item*)",
      "Bash(git push*)",
      "Bash(docker rm*)",
      "Bash(docker rmi*)",
      "Bash(format*)",
      "Bash(del /f*)"
    ]
  }
}
```

---

## Project settings.json Content (`D:\Claudi\.claude\settings.json`)

```json
{
  "permissions": {
    "allow": [
      "Bash(python*)",
      "Bash(pip*)",
      "Bash(docker build*)",
      "Bash(docker run*)",
      "Bash(docker compose*)",
      "Bash(docker-compose*)",
      "Bash(npm*)",
      "Bash(New-Item*)",
      "Bash(Copy-Item*)",
      "Bash(Move-Item*)",
      "Bash(Set-Content*)",
      "Bash(Get-Content*)",
      "Bash(Test-Path*)",
      "Bash(cloudflared*)"
    ],
    "deny": [
      "Bash(Remove-Item*)",
      "Bash(git push*)",
      "Bash(docker rm -f*)",
      "Bash(docker system prune*)"
    ]
  }
}
```

---

## Project settings.local.json Content (`D:\Claudi\.claude\settings.local.json`)

> ⚠️ Gitignored. Personal overrides only. These allow destructive commands at your own risk.

```json
{
  "permissions": {
    "allow": [
      "Bash(git push*)",
      "Bash(Remove-Item*)",
      "Bash(docker system prune*)"
    ]
  }
}
```

---

## Project CLAUDE.md Content (`D:\Claudi\CLAUDE.md`)

```markdown
# CLAUDE.md
> Workflow Lab — AI Automation Stack
> Owner: Danjkboks | Last reviewed: 2026-05-12

---

## Project

Self-hosted AI automation lab on Windows 11 (Aorus 16X, i7, 16GB RAM, RTX 4070 8GB).
Stack: n8n (Docker) · LLMLingua-2 (Docker) · Claude Code CLI · OpenRouter API · Obsidian + GitHub wiki.
Goal: Build and iterate AI agent workflows with a 40€/month budget ceiling.
Repo: D:\workflow-lab\ → github.com/Danjkboks/workflow-lab

---

## Non-Negotiables

- NEVER run Remove-Item or rm -rf without explicit user confirmation on the same line.
- NEVER git push without showing the diff and getting a go-ahead.
- NEVER modify files outside D:\workflow-lab\ or D:\Claudi\ unless explicitly told to.
- NEVER suggest AWS, GCP, or Azure. Self-hosted or bust. EU-hosted cloud only as last resort.
- NEVER hardcode API keys. All secrets go in .env. .env is never committed.
- NEVER exceed 3 steps to set something up without asking if a simpler path exists.
- OpenRouter is the ONLY LLM gateway. No direct Anthropic / OpenAI / Mistral keys.

---

## Operational Notes

- Shell is PowerShell on Windows 11. Use \ paths, not /. Use $env:VAR not $VAR.
- Docker Desktop must be running before any docker build or docker run command.
- All Docker builds run from D:\workflow-lab\agents\. Missing this = "no Dockerfile found" error.
- Cloudflare tunnel URL changes on every restart. Note it down after each cloudflared run.
- n8n production webhook = /webhook/ path. Test webhook = /webhook-test/ (one-time use only).
- LLMLingua model microsoft/llmlingua-2-bert-base-multilingual-cased-meetingbank runs CPU-only.
- [transformers] sequence > 512 warning is harmless — LLMLingua auto-chunks.
- Budget ceiling: 40€/month total. Flag any suggestion that would exceed this.
- Secondary machine (PikaOS dual-boot) is test/lab only. No production deployments there.

---

## Stack Reference

D:\workflow-lab\
├── agents\         ← Python scripts + Dockerfiles
│   ├── ask.py      ← Wiki Q&A with LLMLingua compression (validated, 46% token savings)
│   ├── Dockerfile
│   ├── requirements.txt
│   └── .env        ← NEVER commit this
└── wiki\           ← All .md SOPs = agent long-term memory
    └── MASTER_BUILD.md  ← Source of truth for the whole stack

Services:
- n8n: port 5678 (Live)
- Flask compressor (planned): port 5001
- Cloudflare tunnel: dynamic URL (restart = new URL)

---

## Agent Docs (Read When Relevant)

- wiki/MASTER_BUILD.md — Full stack status, what is built, what is next
- wiki/SOP_n8n_Agent_Bridge.md — n8n + Cloudflare tunnel setup
- wiki/SOP_LLMLingua_Token_Reduction_Docker.md — ask.py, Docker build, compression params
- wiki/SOP_Obsidian_Github_Agent_Memory.md — Git sync, Obsidian vault
- wiki/SOP_ClaudeCode_VSCode_OpenRouter_Windows.md — This environment config

---

## Communication

- English only. Terse. No preamble before answers.
- No filler phrases.
- When proposing a plan: numbered steps, one line each. Ask before executing.
- When writing code: show the full file, not snippets, unless change is < 5 lines.
- Flag budget risk with ⚠️ COST: prefix. Flag destructive actions with ⚠️ DESTRUCTIVE: prefix.

---

## Principles

1. Simple first — if it takes more than 3 steps, find a simpler way.
2. Atomic commits — one change per commit. Small PRs. Meaningful messages.
3. Decisions over behaviors — explain why a choice was made.
4. The wiki is memory — any new SOP, fix, or config goes into wiki/ as a .md file.
5. Verify before trust — never assume a port is open, container running, or token valid. Check first.
```

---

## CLAUDE.md Best Practices (2026 Research Summary)

- **Under 250 lines** — above that, agents skim and ignore rules
- **Every rule must be falsifiable** — "write good code" = useless. "Never run Remove-Item without confirmation" = enforceable
- **No linter-replaceable rules** — don't tell Claude about formatting, that's what linters are for
- **Progressive disclosure** — root CLAUDE.md stays short, points to wiki/ files for task details
- **Capture decisions, not behaviors** — "we use OpenRouter because budget" > "use OpenRouter"
- **No philosophy sections** — they dilute real rules

---

## Model Reference (OpenRouter)

| Model string | Speed | Cost | Best for |
|---|---|---|---|
| `anthropic/claude-3-5-haiku` | Fast | Low ($0.80/$4 per Mtok) | Default — 90% of tasks |
| `anthropic/claude-3-7-sonnet` | Medium | Medium ($3/$15 per Mtok) | Complex reasoning |
| `openai/gpt-4o-mini` | Fast | Low | General tasks |
| `google/gemini-2.0-flash` | Fast | Very low | Large context |
| `mistralai/mistral-small` | Fast | Low | EU-hosted option |
| `deepseek/deepseek-r1` | Medium | Low ($0.55/$2.19 per Mtok) | Reasoning, alternative |

Switch model temporarily:
```powershell
claude --model claude-sonnet-4-5 "your question"
```

---

## Troubleshooting

| Symptom | Cause | Fix |
|---|---|---|
| Model stays on Opus | `.claude.json` hardcoded | Delete `%USERPROFILE%\.claude.json` |
| Env vars ignored | Active PowerShell session | Close and reopen PowerShell |
| VS Code shows Opus | Extension ignores env vars | Add `claude.defaultModel` in settings.json |
| `$env:VAR = "..."` doesn't persist | Session scope only | Use `SetEnvironmentVariable(..., "User")` |
| `requires 1 argument` on docker build | Missing `.` | `docker build -t name .` |
| `no Dockerfile found` | Wrong folder | `cd D:\workflow-lab\agents` first |
| Cloudflare 403/502 from agent | Headless browser blocked | Use `urllib` not `requests` in Python |

---

## What Is NOT Built Yet (Next Steps)

- [ ] Flask compressor microservice — `ask.py` as always-on API on port 5001
- [ ] n8n workflow using wiki Q&A via HTTP call to Flask
- [ ] Scheduled wiki health check — weekly cron in n8n
- [ ] ScrapeGraphAI ingestion pipeline — scrape → summarize → write to wiki
- [ ] PikaOS secondary machine setup as test/lab environment

---

## First Claude Code Task (Flask Microservice)

Paste this in the Claude panel in VS Code:

```
Read D:\workflow-lab\agents\ask.py then create a Flask API wrapper.
- File: D:\workflow-lab\agents\compress_api.py
- Endpoint: POST /compress on port 5001
- Input JSON: {"text": "...", "question": "..."}
- Output JSON: {"compressed_prompt": "...", "tokens_saved_pct": 0}
- Add GET /health → {"status": "ok"}
- Max 60 lines
Do not run anything. Show me the file first.
```
