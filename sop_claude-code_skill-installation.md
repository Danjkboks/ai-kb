---
type: sop
domain: claude-code
source: session
status: active
tags: [claude-code, skills, n8n-skills, cc-safe-setup, memory-bank, installation, windows]
created: 2026-05-12
updated: 2026-05-21
---

# SOP — Claude Code Skill Installation (Windows)

**Purpose:** Install and verify Claude Code skills globally so they are available in every project session.

---

## Prerequisites

- Claude Code CLI installed at `C:\Users\GnReN-PC\.claude\`
- Node.js v18+ installed (`node --version` to verify)
- Git installed (`git --version` to verify)
- PowerShell terminal

---

## Skill Storage Location

All global skills live at:
```
C:\Users\GnReN-PC\.claude\skills\
```

⚠️ Do NOT install to `D:\Claudi\~\.claude\` — that is a leftover path from an earlier misconfiguration. Claude Code reads from `C:\Users\GnReN-PC\.claude\`.

---

## 1. Install n8n-skills (7 workflow-building skills)

**From local copy (preferred — no internet needed):**
```powershell
New-Item -ItemType Directory -Force -Path "C:\Users\GnReN-PC\.claude\skills"
Copy-Item -Recurse "D:\aidirectory\skills\n8n-skills\skills\*" "C:\Users\GnReN-PC\.claude\skills\"
```

**Or clone fresh from GitHub:**
```powershell
git clone https://github.com/czlonkowski/n8n-skills "$env:TEMP\n8n-skills-tmp"
New-Item -ItemType Directory -Force -Path "C:\Users\GnReN-PC\.claude\skills"
Copy-Item -Recurse "$env:TEMP\n8n-skills-tmp\skills\*" "C:\Users\GnReN-PC\.claude\skills\"
Remove-Item -Recurse -Force "$env:TEMP\n8n-skills-tmp"
```

⚠️ Copy `skills\*` (the subdirectories), NOT the whole repo. Skills must be at `~\.claude\skills\n8n-expression-syntax\`, not nested under `~\.claude\skills\n8n-skills\skills\`.

**Provides:**
- `n8n-expression-syntax` — correct `{{ }}` expression patterns
- `n8n-mcp-tools-expert` — using n8n-mcp MCP tools
- `n8n-workflow-patterns` — 5 proven workflow patterns
- `n8n-validation-expert` — validation error fixes
- `n8n-node-configuration` — operation-aware node setup
- `n8n-code-javascript` — JS in Code nodes
- `n8n-code-python` — Python in Code nodes

**Local copy at:** `D:\aidirectory\skills\n8n-skills\`

---

## 2. Install cc-safe-setup (8 safety hooks)

```powershell
npx cc-safe-setup
```

When prompted `Install all 8 safety hooks? [Y/n]` → type `y`. Or pipe it: `echo "y" | npx cc-safe-setup`

**Installs to:** `C:\Users\GnReN-PC\.claude\hooks\`

⚠️ **Windows + jq:** Hooks install but JSON-parsing hooks are limited without `jq`. Install via `winget install jqlang.jq` or `scoop install jq`.

| Hook | Protects against |
|---|---|
| `destructive-guard.sh` | `rm -rf`, `Remove-Item -Recurse` deleting user directories |
| `branch-guard.sh` | Pushing untested code to main |
| `syntax-check.sh` | Python syntax errors cascading through files |
| `context-monitor.sh` | Silent context loss at tool call 150+ |
| `comment-strip.sh` | Comments in bash commands breaking allowlists |
| `cd-git-allow.sh` | cd+git permission spam |
| `secret-guard.sh` | `.env` files committed accidentally |
| `api-error-alert.sh` | Autonomous sessions dying silently from rate limits |

⚠️ **Requires `jq`** for full hook functionality. On Windows, hooks still install but JSON parsing in hooks may be limited without jq.

---

## 3. Install memory-bank skill (optional, token optimization)

```powershell
git clone https://github.com/Nagendhra-web/memory-bank "C:\Users\GnReN-PC\.claude\skills\memory-bank"
```

**Purpose:** 3-tier memory system, reduces token waste by 60-80%, extends session length 3-5x.

---

## 4. Verify installation

```powershell
ls C:\Users\GnReN-PC\.claude\skills\
ls C:\Users\GnReN-PC\.claude\hooks\
```

Expected output:
```
skills\
  n8n-skills\
  memory-bank\   (if installed)

hooks\
  destructive-guard.sh
  branch-guard.sh
  syntax-check.sh
  context-monitor.sh
  comment-strip.sh
  cd-git-allow.sh
  secret-guard.sh
  api-error-alert.sh
```

---

## 5. Restart Claude Code

Hooks only activate after restart. Close and reopen Claude Code after installation.

---

## Troubleshooting

| Problem | Fix |
|---|---|
| `npx` not found | Install Node.js LTS from nodejs.org, reopen PowerShell |
| `git` not found | Install Git from git-scm.com |
| Skills installed to wrong path | Uninstall, re-run with explicit `--dest` or use `git clone` with full path |
| Hooks not triggering | Verify `C:\Users\GnReN-PC\.claude\hooks\` contains `.sh` files and Claude Code was restarted |

---

## Adding New Skills

To add any skill from GitHub:
```powershell
git clone https://github.com/{author}/{repo} "C:\Users\GnReN-PC\.claude\skills\{skill-name}"
```

To add via `npx skills` CLI:
```powershell
npx skills add {author}/{skill-name}
```

When prompted, choose **global** (all projects) not project-specific.
