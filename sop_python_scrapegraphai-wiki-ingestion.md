---
type: sop
domain: python
source: imported
status: active
tags: [scrapegraphai, wiki, obsidian, ingestion, openrouter]
created: 2026-05-12
updated: 2026-05-21
---

SOP: Autonomous Agent Wiki + ScrapeGraphAI Ingestion
(scope: Obsidian + Git repo you already use, Windows / PikaOS, OpenRouter, 20 €/mo)

---

## 0. Goal & Architecture

- Build a self-hosted “agent wiki” where an LLM:
  - ingests raw sources → Obsidian markdown knowledge base
  - answers questions against that wiki
  - gradually cleans and extends it
- Feed the wiki using ScrapeGraphAI for structured scraping of websites and docs.

---

## 1. Baseline Prereqs

- Complete your existing Obsidian + GitHub Agent Memory SOP so you have:
  - a repo cloned locally
  - Obsidian vault pointing at that repo
  - Git auto-sync working every few minutes.
- Install:
  - Python 3.11+ on both Windows and PikaOS
  - pipx or venv for isolation (recommended)
- Have ready:
  - OpenRouter API key
  - A personal “playground” branch in your repo (e.g. `agents-lab`).

---

## 2. Repo / Folder Layout

Inside your Obsidian/Git repo, standardize this layout:

- `/raw/` – unprocessed input:
  - `/raw/web/` (HTML, JSON, scraped artifacts)
  - `/raw/papers/` (PDFs)
  - `/raw/notes/` (dumped text, logs)
- `/wiki/` – LLM-generated `.md` knowledge base (Karpathy-style).
- `/agents/` – scripts and config:
  - `/agents/config/` – YAML/JSON for models and scrap flows
  - `/agents/scripts/` – Python entrypoints

---

## 3. LLM Access Config (OpenRouter)

- In `/agents/config/model.yaml` define:
  - provider: `openrouter`
  - base_url: their OpenAI-compatible endpoint
  - model: pick 1 reasoning-strong model you can afford.
- Store API key:
  - in `.env` (not committed) with `OPENROUTER_API_KEY=...`
  - load via `python-dotenv` in scripts.
- Rule: one model first, do not multi-model until workflows are stable.

---

## 4. Autonomous Agent Wiki Loop

### 4.1 Ingest: raw → draft wiki notes

- Script name: `/agents/scripts/ingest_raw_to_wiki.py`.
- Per run:
  - Scan `/raw/**` for new/changed files since last run (track in a small JSON state file).
  - For each new file:
    - Extract plain text (PDF → text, HTML → stripped text, etc.).
    - Chunk text (e.g. 1–3k tokens per chunk).
    - Call LLM with a prompt like:
      - “Summarize this chunk into Obsidian markdown, extract key concepts, tags, and link candidates; do NOT hallucinate sources.”
    - Write to `/wiki/` as `YYYYMMDD_slug.md` with:
      - YAML frontmatter: title, source_path, date_ingested, tags, source_url (if any).

### 4.2 Q&A over the wiki

- Script: `/agents/scripts/ask_wiki.py`.
- Behavior:
  - Input: question string (CLI arg or simple prompt).
  - Search:
    - run keyword search over `/wiki/*.md` (grep/whoosh/simple TF-IDF).
    - select top N candidate files (e.g. 5–10).
  - Build LLM prompt:
    - include question + concatenated relevant markdown excerpts.
    - ask LLM to answer strictly grounded in excerpts and list which notes it referenced.
  - Output to:
    - console, plus
    - optional `/outputs/` as `Q_DATE_TIME.md` (question, answer, references).

### 4.3 “Linting” / health checks

- Script: `/agents/scripts/wiki_health_check.py`.
- Behavior (run weekly / on demand):
  - Sample or iterate over wiki notes.
  - Ask LLM to:
    - flag inconsistent facts, missing data, duplicate concepts
    - suggest new article ideas / missing links.
  - Write a `HealthCheck_YYYYMMDD.md` report into `/wiki/`.

---

## 5. ScrapeGraphAI Ingestion Pipeline

### 5.1 Install & configure

- In your Python env:
  - `pip install scrapegraphai`.
- Create `/agents/config/scrape.yaml`:
  - default LLM provider: OpenAI-compatible, pointing to OpenRouter endpoint
  - temperature, max_tokens tuned low to control cost
  - output format: JSON or markdown.

### 5.2 Define scraping jobs

- Script: `/agents/scripts/scrape_to_raw.py`.
- For each job, define in YAML:
  - `job_name` (e.g. `vendor_status_pages`, `tool_docs`)
  - list of URLs
  - extraction template (fields you want: title, main_text, table_data, metadata).
- In Python, for each job:
  - instantiate a ScrapeGraphAI graph component
  - run graph → get structured result.
  - Write result to `/raw/web/{job_name}/{slug}.json` and optionally `.txt` render.

### 5.3 Connect ScrapeGraphAI → wiki loop

- Adjust `ingest_raw_to_wiki.py` to:
  - detect ScrapeGraph JSON files,
  - flatten fields (e.g. `main_text`) into a single text blob,
  - feed that into the same summarization routine as PDFs/HTML.

---

## 6. Automation & Operations

- Scheduling:
  - Use cron (Linux) / Task Scheduler (Windows) to run:
    - `scrape_to_raw.py` daily/weekly per source.
    - `ingest_raw_to_wiki.py` hourly.
    - `wiki_health_check.py` weekly.
- Cost control:
  - Cap max tokens per run.
  - Limit number of URLs per job and depth of scraping.
  - Log token usage per run in a simple CSV.
- Git hygiene:
  - Keep `/raw/` small; archive/rotate large raw objects elsewhere.
  - Commit scripts and config, but ignore `.env` and heavy data via `.gitignore`.

---

## 7. Role in the Overall Project

- Autonomous Agent Wiki = long-term memory and knowledge backend for your future agents.
- ScrapeGraphAI = ingestion layer pulling external web knowledge into that memory.
