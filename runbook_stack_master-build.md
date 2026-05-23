---
type: runbook
domain: stack
source: created
status: active
tags: [n8n, docker, llmlingua, qdrant, langfuse, cloudflare, openrouter, master-build]
created: 2026-05-08
updated: 2026-05-19
---

# MASTER BUILD — Workflow Lab Infrastructure
## AI Agent Memory · n8n Automation · LLMLingua Token Reduction

> **Purpose of this file**: This is the single source of truth for the entire Workflow Lab build.
> It is designed to be loaded at the start of every new conversation so the AI assistant
> has full context of what exists, what was built, how it works, and what comes next.
> Last updated: 2026-05-18

---

## 1. Who You Are (User Profile)

- **Name/handle**: Danjkboks
- **Location**: Carpentras, France (EU pricing priority)
- **Role**: Sysadmin student, beginner level in AI/automation
- **Budget**: ~50€/month
- **Hardware**:
  - Primary: Gigabyte Aorus 16X ASG — i7, 16GB RAM, RTX 4070 8GB (Windows 11)
  - Secondary: older laptop, dual-boot PikaOS (Linux) + Windows 10 — used as lab/test machine
- **Accounts**: OpenRouter (with credits), GitHub (`Danjkboks`)
- **Preferences**: Self-hosted > cloud · Simple first · EU-first · No overengineering

---

## 2. Core Philosophy

- Start simple. Add complexity only when the simple version breaks or runs out of capability.
- Every piece of the stack must be explainable in one sentence.
- Use OpenRouter as the single LLM gateway — no direct API keys per provider.
- Knowledge lives in `.md` files in GitHub. That is the memory layer.
- Agents read from that memory, not from chat history.

---

## 3. Repository

- **GitHub repo**: `https://github.com/Danjkboks/workflow-lab`
- **GitHub user**: `Danjkboks`
- **Clone command**: `git clone https://github.com/Danjkboks/workflow-lab.git`
- **Local path (Windows)**: `D:\workflow-lab\`

### Repo folder structure

```
D:\workflow-lab\
├── agents\                        ← Python scripts + Docker builds
│   ├── ask.py                     ← Wiki Q&A agent with LLMLingua compression
│   ├── Dockerfile                 ← Docker image definition for ask.py
│   ├── requirements.txt           ← Python dependencies
│   └── .env                       ← API keys + runtime config (NOT committed to git)
├── wiki\                          ← Knowledge base: all .md SOPs and notes
│   ├── SOP_Obsidian_Github_Agent_Memory.md
│   ├── SOP_n8n_Agent_Bridge.md
│   ├── SOP_LLMLingua_Token_Reduction_Docker.md
│   └── MASTER_BUILD.md            ← THIS FILE
└── .gitignore                     ← must include .env
```

> ⚠️ `.env` is NEVER committed. Add it to `.gitignore`. It contains your OpenRouter key.

---

## 4. Stack Overview

| Layer | Tool | Purpose | Status |
|---|---|---|---|
| Knowledge base | Obsidian + GitHub | Persistent memory, versioned SOPs | ✅ Live |
| Sync | Obsidian Git plugin | Auto push/pull every 10 min | ✅ Live |
| Automation | n8n (Docker) | Workflow orchestrator | ✅ Live |
| Network tunnel | Cloudflare Tunnel (cloudflared) | Expose n8n to external agents securely | ✅ Live |
| LLM gateway | OpenRouter | Single API for all models | ✅ Live |
| Token compression | LLMLingua-2 (Docker) | Reduce prompt tokens ~40-60% before LLM call | ✅ Validated |
| Agent memory reader | ask.py | Wiki Q&A with compression + token logging | ✅ Validated |

---

## 5. Module 1 — Obsidian + GitHub Agent Memory

**Goal**: A permanent, version-controlled knowledge base that auto-syncs and serves as long-term memory for AI agents.

### Setup

```powershell
# Clone repo
git clone https://github.com/Danjkboks/workflow-lab.git

# Fix Git identity if needed (run once)
git config --global user.name "Danjkboks"
git config --global user.email "your-github-email@example.com"

# Install Obsidian
winget install Obsidian.Obsidian
```

### Obsidian Git Plugin Config

- Plugin: **Git** by Vinzent (Denis Olehov) — 2.5M+ downloads
- Settings to change:
  - Auto commit-and-sync interval: `10` minutes
  - Auto pull interval: `10` minutes
- Manual trigger: `Ctrl+P` → `Git: Commit-and-sync`

### How it works

```
You write/edit .md in Obsidian
    → Git plugin commits and pushes to GitHub every 10 min
    → AI agents pull the latest SOPs from GitHub
    → Agents always have fresh memory
```

### Key file: `SOP_Obsidian_Github_Agent_Memory.md`

---

## 6. Module 2 — n8n Agent Bridge (Docker + Cloudflare Tunnel)

**Goal**: Self-hosted n8n accessible by external AI agents without exposing your IP or opening router ports.

### Docker Compose (`docker-compose.yml`)

```yaml
version: '3.8'
services:
  n8n:
    image: docker.n8n.io/n8nio/n8n:latest
    restart: unless-stopped
    ports:
      - "5678:5678"
    environment:
      - GENERIC_TIMEZONE=Europe/Paris
    volumes:
      - n8n_data:/home/node/.n8n
volumes:
  n8n_data:
```

### Cloudflare Tunnel

```powershell
# Install cloudflared (Windows)
winget install Cloudflare.cloudflared

# Run tunnel (exposes n8n to internet via Cloudflare)
cloudflared tunnel --url http://localhost:5678

# If localhost fails (Windows loopback block), use LAN IP instead:
cloudflared tunnel --url http://192.168.X.X:5678
```

> ⚠️ Every time you restart cloudflared, you get a NEW tunnel URL. Note it down for agent configs.

### n8n Webhook Node Config

- Node type: **Webhook**
- Method: GET (trigger) or POST (payloads)
- Authentication: **Header Auth**
  - Header name: `x-api-key` (or custom)
  - Value: your secret key
- **Switch from Test URL to Production URL** before going live (`/webhook/` not `/webhook-test/`)
- Click **Publish** to activate persistent listening

### Agent calling n8n (Python — bypasses Cloudflare bot block)

```python
import urllib.request

url = "https://YOUR-TUNNEL-URL.trycloudflare.com/webhook/YOUR-PATH"

req = urllib.request.Request(
    url,
    headers={
        'User-Agent': 'Custom-Agent/1.0',
        'x-api-key': 'YOUR_SECRET_KEY'
    }
)

with urllib.request.urlopen(req, timeout=10) as response:
    print(response.read().decode('utf-8'))
```

> ⚠️ Do NOT use `requests` library or `fetch_url` — Cloudflare blocks headless browsers. Use `urllib` only.

### Key file: `SOP_n8n_Agent_Bridge.md`

---

## 7. Module 3 — LLMLingua Token Reduction (Docker)

**Goal**: Compress prompts before sending to OpenRouter. Reduces billed tokens by ~40-60%.

### How it works

```
wiki .md files
    → loaded into memory
    → LLMLingua-2 compresses the prompt (rate=0.4 = keep 40% of tokens)
    → compressed prompt sent to OpenRouter
    → response + actual billed token count logged
```

### Files in `D:\workflow-lab\agents\`

#### `requirements.txt`
```
llmlingua
openai
python-dotenv
```

#### `Dockerfile`
```dockerfile
FROM python:latest
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
CMD ["python", "ask.py"]
```

#### `.env`
```env
OPENROUTER_API_KEY=your_key_here
QUESTION=Summarize the key topics in this knowledge base.
MODEL=anthropic/claude-3-5-haiku
```

#### `ask.py` (full script)
```python
import os
from llmlingua import PromptCompressor
from openai import OpenAI
from dotenv import load_dotenv

load_dotenv()

compressor = PromptCompressor(
    model_name="microsoft/llmlingua-2-bert-base-multilingual-cased-meetingbank",
    use_llmlingua2=True,
    device_map="cpu"
)

client = OpenAI(
    base_url="https://openrouter.ai/api/v1",
    api_key=os.getenv("OPENROUTER_API_KEY")
)

wiki_dir = "/wiki"
wiki_content = ""
loaded = []

for fname in sorted(os.listdir(wiki_dir)):
    if fname.endswith(".md"):
        with open(os.path.join(wiki_dir, fname), "r", encoding="utf-8") as f:
            wiki_content += f.read() + "\n\n"
            loaded.append(fname)

print(f"Loaded {len(loaded)} wiki files: {loaded}")
print(f"Total chars before compression: {len(wiki_content)}")

question = os.getenv("QUESTION", "Summarize the key topics in this knowledge base.")
prompt = f"Wiki content:\n{wiki_content}\n\nQuestion: {question}"
tokens_before = len(prompt.split())

if len(prompt) > 500:
    compressed = compressor.compress_prompt(prompt, rate=0.4)
    prompt = compressed["compressed_prompt"]

tokens_after = len(prompt.split())
print(f"Tokens before: {tokens_before} | after: {tokens_after} | saved: {round((1 - tokens_after/tokens_before)*100)}%")

response = client.chat.completions.create(
    model=os.getenv("MODEL", "anthropic/claude-3-5-haiku"),
    messages=[
        {"role": "system", "content": "You answer questions strictly based on the provided wiki content."},
        {"role": "user", "content": prompt}
    ],
    max_tokens=800
)

usage = response.usage
print(f"API tokens — prompt: {usage.prompt_tokens} | completion: {usage.completion_tokens} | total: {usage.total_tokens}")
print("\n--- RESPONSE ---")
print(response.choices[0].message.content)
```

### Build & Run commands

```powershell
# Always run from agents/ folder
cd D:\workflow-lab\agents

# Build image (required after any code change)
docker build -t agent-llmlingua .

# Run (mounts wiki folder at runtime)
docker run --rm --env-file .env -v "D:\workflow-lab\wiki:/wiki" agent-llmlingua
```

### Validated results

- Input: `SOP_Agent_Wiki_ScrapeGraphAI.md` (5098 chars)
- Tokens before: 767 → after: 413 → **46% saved**
- Actual billed tokens via OpenRouter response.usage logged ✅

### Compression rate guide

| `rate` | Tokens kept | Use when |
|---|---|---|
| `0.3` | 30% | Very long docs, some quality loss OK |
| `0.4` | 40% | **Default — best balance** |
| `0.5` | 50% | Shorter docs, higher fidelity needed |
| `0.6` | 60% | Near-lossless, minimal savings |

### Common mistakes

| Mistake | Symptom | Fix |
|---|---|---|
| Missing `.` in build command | `requires 1 argument` | `docker build -t agent-llmlingua .` |
| `docker run` pasted in Dockerfile | Build fails silently | Dockerfile = instructions only. Run command goes in PowerShell |
| Build from wrong folder | `no Dockerfile found` | `cd agents` first |
| `--env-file .env` from wrong dir | `cannot find file` | Use full path or `cd agents` first |
| Edited `ask.py`, didn't rebuild | Old code runs | Rebuild after every code change |
| Model gives generic answer | Wiki not mounted | `docker run --rm -v "D:\workflow-lab\wiki:/wiki" python:latest ls /wiki` |
| `[transformers] sequence > 512` | Warning in terminal | Harmless — LLMLingua auto-chunks |

### Key file: `SOP_LLMLingua_Token_Reduction_Docker.md`

---

## 8. Token Reduction in n8n (Not Yet Built — Planned)

LLMLingua does NOT run natively inside n8n. Three options to integrate:

| Option | How | Complexity | Status |
|---|---|---|---|
| **Flask microservice** (recommended) | ask.py becomes an always-on API on port 5001, n8n calls it via HTTP Request node before every OpenRouter call | Low — one service, all workflows benefit | ⬜ Planned |
| **n8n Code node (JS)** | Lightweight prompt cleanup (strip whitespace, markdown noise, truncate) — 10-25% savings, no Python | Zero setup | ⬜ Planned |
| **Execute Command node** | n8n calls Docker directly | Slow (30s per call), only for batch jobs | ⬜ Not recommended for real-time |

> Next step: convert `ask.py` to a Flask API and add one HTTP node in n8n before each OpenRouter call.

---

## 9. OpenRouter Model Reference

| Model string | Speed | Cost | Best for |
|---|---|---|---|
| `anthropic/claude-3-5-haiku` | Fast | Low | Grounded Q&A, SOPs |
| `anthropic/claude-3-7-sonnet` | Medium | Medium | Complex reasoning |
| `openai/gpt-4o-mini` | Fast | Low | General tasks |
| `google/gemini-2.0-flash` | Fast | Very low | Large context |
| `mistralai/mistral-small` | Fast | Low | EU-hosted option |

---

## 10. What AI Should Know When Reading This File

- The user is a beginner — explain clearly, no jargon without definition
- Always prefer the simplest solution first
- All paths are Windows (`D:\workflow-lab\`) unless stated otherwise
- The secondary machine (PikaOS) is for testing only — no production deployments there yet
- Budget is ~20€/month — flag anything that would exceed that
- OpenRouter is the ONLY LLM gateway — do not suggest direct API keys to individual providers
- The repo `Danjkboks/workflow-lab` is the source of truth — always check it before assuming a file exists
- Do NOT suggest cloud-only solutions (AWS, GCP, Azure) — self-hosted first
- Do NOT suggest overly complex stacks — if it needs more than 3 steps to set up, propose a simpler alternative first

---

## 11. What Is NOT Built Yet (Next Steps)

- [ ] Flask compressor microservice (so n8n gets automatic token reduction)
- [ ] n8n workflow that uses wiki Q&A via HTTP call
- [ ] Scheduled wiki health check (weekly cron)
- [ ] ScrapeGraphAI ingestion pipeline (scrape → summarize → write to wiki)
- [ ] Second machine (PikaOS) setup as test/lab environment

---

## 12. Glossary (for future conversations)

| Term | Meaning |
|---|---|
| `workflow-lab` | The GitHub repo and local project folder for all builds |
| `wiki/` | The folder of `.md` files = agent long-term memory |
| `agents/` | Python scripts + Docker builds |
| `ask.py` | The main wiki Q&A script with LLMLingua compression |
| `agent-llmlingua` | Docker image name for ask.py |
| OpenRouter | API gateway for all LLM calls (one key, many models) |
| cloudflared | Cloudflare Tunnel CLI — gives n8n a public HTTPS URL |
| LLMLingua-2 | Microsoft's prompt compression model, runs on CPU |
| `rate=0.4` | LLMLingua compression level — keep 40% of tokens |
| `response.usage` | OpenRouter returns actual billed token counts here |
