---
type: decision
date: 2026-07-13
status: DECIDED — migrate
---

# Decision — Internal memory pipeline architecture

## Context
Current chain (8 hops, internal session memory only — not the client product):
session-watcher.ps1 -> n8n webhook (Cloudflare tunnel) -> LLMLingua -> OpenRouter/DeepSeek -> Obsidian md
-> Qdrant -> mine_patterns.py -> CLAUDE.md

Note: pipeline was not "broken" by a bug — Docker Desktop simply wasn't running while AJ was heads-down
on the ITSM project. Containers are live again; the existing chain can resume as-is. This decision is
about whether resuming as-is is the right foundation, not about fixing a defect.

## Decision: migrate to memsearch (github.com/zilliztech/memsearch)

New chain (2-3 hops): Claude Code hooks -> memsearch (local ONNX embed + LLM summarize) -> markdown +
Milvus (small self-hosted container, not Milvus Lite, for concurrent-session support).

## Why
- pip/uv install, WSL2-native, no npm — no conflict with CLAUDE.md Non-Negotiables.
- Existing `knowledge\extracts\*.md` are indexable as-is — no migration needed for past work.
- Removes: session-watcher.ps1 scheduled task, n8n webhook + Cloudflare tunnel, LLMLingua container,
  Qdrant. Removes the exact failure surface that let the pipeline sit unnoticed (async webhook,
  tunnel URL rotation, container babysitting).
- n8n and LLMLingua are NOT removed from the project — they stay for the client product (Phase C/D).
  This decision only affects AJ's internal session-memory tooling.

## Implementation outcome (2026-07-14) — vector store revised to Milvus Lite + write-lock

The "small self-hosted Milvus container" assumed above proved unviable in this environment:
- **Milvus 2.5.x** (only line supporting single-container embedded-etcd): pymilvus 3.0.0 (bundled by
  memsearch 0.4.14) creates the collection fine but its inserts silently no-op — server `row_count`
  stays 0. Confirmed client↔server version mismatch (memsearch uses Milvus-3.0-generation features:
  server-side BM25 `Function`, `SPARSE_FLOAT_VECTOR` + analyzer, `hybrid_search` + RRF).
- **Milvus 2.6.x**: dropped single-container embedded etcd — panics `embedded etcd can not be used
  under distributed mode`; needs the full 3-container (etcd+minio+milvus) topology.
- **Milvus 3.0.x**: the version pymilvus 3.0.0 officially matches, but it is beta and also 3-container.

Decision (AJ, 2026-07-14): use the bundled **Milvus Lite** (verified working: index + search) and
serialize the SessionEnd hook's index step behind a Windows named mutex (`Global\memsearch-index-lock`)
to cover the concurrent-session concern that originally ruled Lite out. Zero containers, no beta, no
version fragility. Downgrading pymilvus is not viable — memsearch requires the modern server features.
Lite DB lives on WSL ext4 at `~/.memsearch/milvus.db` (a rebuildable index; source of truth is the
`.md` extracts in the vault).

## Required follow-up work (not "install and done")
1. Custom `prompts.summarize` template — memsearch's default output is generic third-person notes,
   not the `patterns.good / patterns.bad / errors / env_friction` YAML schema `mine_patterns.py`
   depends on. Must write a custom summarize prompt that preserves this schema, or rule-mining quality
   degrades silently.
2. Use a small self-hosted Milvus container (not Milvus Lite) — Milvus Lite is single-process only;
   AJ runs concurrent Claude Code sessions.
3. Repoint `mine_patterns.py` at memsearch's markdown output path once (1) is done.
4. Decide backfill path for the June 11 - July 5 raw `.jsonl` sessions never extracted (via
   memsearch's transcript parser, once installed).

## Not affected by this decision
mine_patterns.py -> CLAUDE.md rule promotion (kept, repointed), integrity agent, graphify — all
orthogonal to the capture/store substrate.
