---
type: sop
domain: python
source: session
status: active
tags: [flask, llmlingua, compression, api, langfuse, openrouter, docker, prompt-caching]
created: 2026-05-18
updated: 2026-05-21
---

# SOP — Flask LLMLingua API (agent-llmlingua)

**Purpose:** Deploy and use the Flask compression + LLM API service. Handles prompt compression via LLMLingua-2, routes to OpenRouter, traces via Langfuse.

---

## What It Does

`api_with_langfuse.py` is the core microservice:
1. Accepts a question + context via `POST /ask`
2. Compresses the context with LLMLingua-2 (~43% token savings)
3. Sends compressed prompt to OpenRouter (default: `anthropic/claude-haiku-4.5`)
4. Returns answer + token stats
5. Traces everything to Langfuse

**Runs as:** Docker container `agent-llmlingua` on port `5001`.

---

## Endpoints

| Endpoint | Method | Purpose |
|---|---|---|
| `/ask` | POST | Compress + query LLM |
| `/health` | GET | Health check |
| `/cache-stats` | GET | Prompt cache hit/miss stats |

---

## POST /ask — Request Format

```json
{
  "question": "What are the key concepts in my knowledge base?",
  "context": "...long document text..."
}
```

**Response:**
```json
{
  "answer": "...",
  "tokens_original": 1200,
  "tokens_compressed": 690,
  "compression_ratio": 0.425,
  "cached_tokens": 300,
  "cost_usd": 0.005
}
```

---

## Start the Service

```powershell
docker start agent-llmlingua
```

⚠️ **Allow 2-3 minutes** on first start — LLMLingua-2 model downloads from HuggingFace.

Check readiness:
```powershell
curl http://localhost:5001/health
```

---

## Test the Endpoint

```powershell
curl -X POST http://localhost:5001/ask `
  -H "Content-Type: application/json" `
  -d '{"question": "What is n8n?", "context": "n8n is a workflow automation tool..."}'
```

---

## Prompt Caching

The API implements Anthropic's prompt caching via OpenRouter:
- **Cache hit:** `$0.005/query` (7x cheaper)
- **Cache miss:** `$0.036/query`
- Cache tracked via `prompt_tokens_details.cached_tokens` in OpenRouter response
- Stats available at `GET /cache-stats`

---

## Configuration

Environment variables (from `D:\aidirectory\.env`):
```
OPENROUTER_API_KEY=sk-or-v1-...
MODEL=anthropic/claude-haiku-4.5
LANGFUSE_PUBLIC_KEY=pk-lf-...
LANGFUSE_SECRET_KEY=sk-lf-...
LANGFUSE_HOST=http://localhost:3000
LANGFUSE_ENABLED=true
DAILY_BUDGET_USD=1.50
```

---

## Rebuild After Code Changes

Dockerfile location: `D:\lab\workflow-lab\agents\Dockerfile`
Source: `D:\lab\workflow-lab\agents\api_with_langfuse.py`
Also copied to: `D:\aidirectory\projects\workflow-lab\agents\api_with_langfuse.py`

```powershell
cd D:\lab\workflow-lab\agents
docker build -t agent-llmlingua:latest .
docker stop agent-llmlingua && docker rm agent-llmlingua
docker run -d --name agent-llmlingua -p 5001:5001 --env-file .env agent-llmlingua:latest
```

---

## LLMLingua-2 Details

- **Model:** `microsoft/llmlingua-2-bert-base-multilingual-cased-meetingbank`
- **Device:** CPU (RTX 4070 not yet exposed to Docker)
- **`[transformers] sequence > 512`** warning is harmless — model auto-chunks
- **Compression rate:** `rate=0.4` keeps 40% of tokens (best balance)
- **Typical savings:** 43-46% tokens vs uncompressed

---

## Integration with n8n

Call the API from an n8n **HTTP Request** node:
- URL: `http://localhost:5001/ask`
- Method: POST
- Body: `{"question": "{{$json.question}}", "context": "{{$json.context}}"}`

This wires n8n → LLMLingua → OpenRouter automatically on every workflow execution.

---

## Troubleshooting

| Problem | Fix |
|---|---|
| Container not ready | Wait 2-3 min, model is downloading |
| `curl: connection refused` | `docker start agent-llmlingua` |
| High cost per query | Check cache stats — low cache hit rate means context isn't cacheable |
| `LANGFUSE_ENABLED=true` error | Add Langfuse keys to `.env` or set `LANGFUSE_ENABLED=false` |
| Sequence >512 warning | Harmless, ignore |
