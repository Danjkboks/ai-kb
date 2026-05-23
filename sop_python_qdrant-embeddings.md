---
type: sop
domain: python
source: session
status: active
tags: [qdrant, embeddings, vector-db, sentence-transformers, n8n, wiki, semantic-search]
created: 2026-05-19
updated: 2026-05-21
---

# SOP — Qdrant Embeddings Pipeline

**Purpose:** Index n8n workflows and wiki documents into Qdrant for semantic similarity search.

---

## What's Indexed

| Collection | Content | Count | Model |
|---|---|---|---|
| `workflows` | n8n workflow JSON metadata | 2,061 | `all-MiniLM-L6-v2` (384 dims, COSINE) |
| `wiki` | Obsidian knowledge base chunks | 71 | `all-MiniLM-L6-v2` (384 dims, COSINE) |

---

## Prerequisites

- Qdrant running on port 6333: `docker start qdrant`
- Python 3.9+ with dependencies: `pip install sentence-transformers qdrant-client python-dotenv`
- Workflow JSON files at `D:\n8n-workflows\workflows\`
- Wiki markdown files at `D:\aidirectory\knowledge\`

---

## 1. Index n8n Workflows

```powershell
cd D:\lab\workflow-lab\agents
python qdrant_embeddings.py
```

**What it does:**
1. Reads all 2,061 workflow JSON files from `D:\n8n-workflows\workflows\`
2. Builds text representation: `{name} {integrations} {trigger} {tags} {complexity}`
3. Generates 384-dim embeddings via `all-MiniLM-L6-v2`
4. Upserts into Qdrant `workflows` collection

**Runtime:** ~15-30 minutes for 2,061 workflows on CPU.

---

## 2. Index Wiki Documents

```powershell
cd D:\lab\workflow-lab\agents
python qdrant_wiki_embeddings.py
```

**What it does:**
1. Reads all `.md` files from the wiki/knowledge directory
2. Chunks documents into ~500-word segments
3. Embeds each chunk
4. Upserts into Qdrant `wiki` collection (71 chunks total)

---

## 3. Verify Collections

```powershell
curl http://localhost:6333/collections
```

Expected response includes `workflows` and `wiki` collections with point counts.

---

## 4. Semantic Search (Python)

```python
from sentence_transformers import SentenceTransformer
from qdrant_client import QdrantClient

model = SentenceTransformer("all-MiniLM-L6-v2")
client = QdrantClient(host="localhost", port=6333)

query = "email notification to slack"
vector = model.encode(query).tolist()

results = client.search(
    collection_name="workflows",
    query_vector=vector,
    limit=5
)
for r in results:
    print(f"[{r.score:.3f}] {r.payload.get('name', 'unknown')}")
```

---

## Re-indexing

Re-run scripts when:
- New workflows added to `D:\n8n-workflows\`
- Wiki documents updated
- Collection appears empty or stale

Scripts are idempotent — safe to re-run, will upsert (overwrite) existing points.

---

## Configuration

Environment variables (in `D:\aidirectory\.env`):
```
QDRANT_HOST=localhost
QDRANT_PORT=6333
```

Collection constants in scripts:
- `COLLECTION_NAME = "workflows"` in `qdrant_embeddings.py`
- Model: `all-MiniLM-L6-v2` (384 dims, COSINE distance)

---

## Storage

Qdrant persists data at `D:\lab\qdrant-storage\`. **Do not move.** Docker volume mounts this path.

---

## Troubleshooting

| Problem | Fix |
|---|---|
| `ConnectionRefusedError` | `docker start qdrant`, wait 10s |
| Missing `sentence_transformers` | `pip install sentence-transformers` |
| Empty results | Check collection exists: `curl localhost:6333/collections` |
| Slow indexing | Expected on CPU — `all-MiniLM-L6-v2` is fast but 2K files takes time |
| Model download stalls | First run downloads model from HuggingFace — needs internet |
