---
type: extract
date: 2026-06-05
session_id: qdrant-extracts-indexer-search-scripts
surface: claude-code
environment: env1-claude-desktop
topics: [python, qdrant, memory]
source_file: 2026-06-05-215350_348db170-367c-4366-aacc-6700465ae5dc.jsonl
processed_at: 2026-06-07T16:04:17.268Z
---

# Session Extract: qdrant-extracts-indexer-search-scripts

## Decisions
- **Use QdrantClient 1.7 API with query_points and upsert methods instead of deprecated search method**: Discovered during script development that the search method is deprecated; updated to use the correct API to avoid AttributeError
- **Create daily scheduled task for Qdrant extracts reindexing**: To maintain fresh embeddings of session extracts in the vector store without manual intervention

## Problems Solved
- **QdrantClient.search() method caused AttributeError: 'QdrantClient' object has no attribute 'search'**: Updated to use client.query_points() method per Qdrant 1.7 API
- **Need to index 34 existing session extracts into Qdrant for semantic search**: Created qdrant_extracts_index.py script that reads EXTRACT_*.md files, splits into sections, embeds with MiniLM-L6-v2, and upserts to Qdrant
- **Need command-line search capability for session extracts**: Created qdrant_extracts_search.py script that takes a query, encodes it, and returns top matching extracts with scores

## Errors Encountered
- [resolved] AttributeError: 'QdrantClient' object has no attribute 'search' -> Updated to use client.query_points() method from Qdrant 1.7 API
- [resolved] Python syntax error in search script after API change -> Fixed syntax and method calls to match Qdrant 1.7 API

## Patterns Identified
- Qdrant 1.7 API uses query_points() not search() for searching
- Session extracts should be split into logical sections (headers) before embedding for better retrieval
- MiniLM-L6-v2 model (384 dims) is downloaded once and reused for both indexing and querying

## Files Modified
- created: D:\aidirectory\scripts\qdrant_extracts_index.py -- Indexes session extracts from knowledge/extracts/ into Qdrant collection 'extracts' with section-based embeddings
- created: D:\aidirectory\scripts\qdrant_extracts_search.py -- Command-line search tool for Qdrant extracts collection using semantic similarity
- modified: D:\aidirectory\scripts\qdrant_extracts_search.py -- Updated from deprecated client.search() to client.query_points() per Qdrant 1.7 API

## Next Session Must Know
- Qdrant extracts collection exists with 34 extracts indexed as 353 chunks (10.4 sections per extract)
- Search script uses QdrantClient 1.7 API with query_points() method, not search()
- Indexer script handles missing frontmatter with defaults (date: unknown, topics: [], surface: '')
- Daily reindex task needs to be registered via Task Scheduler using pwsh.exe wrapper
- Claude command '/search' should be created in claude-commands/ to wrap the Python search script
- Embedding model MiniLM-L6-v2 is cached locally, no re-download needed
- Qdrant runs on localhost:6333, collection name is 'extracts' with 384-dim vectors
- RTK is installed at /home/gnren-pc/local/bin/rtk and hooks are active for token compression

## Skill Candidates
- qdrant-extracts-search: Semantic search across indexed session extracts using Qdrant vector store
- qdrant-extracts-index: Index new session extracts into Qdrant with automatic section splitting and embedding

## Token Waste Flags
- Repeated verification of Qdrant API methods through multiple Python inspection commands
- RTK hook firing and context evacuation messages appearing multiple times without adding value

## Knowledge Base Updates
- [update] sop_python_qdrant-embeddings.md: Add Qdrant 1.7 API specifics: query_points() instead of search(), upsert() method usage, and session extracts indexing pattern
- [create] runbook_qdrant_extracts-indexing.md: Document the complete workflow for indexing session extracts: file reading, section splitting, embedding, and Qdrant upsert with error handling

## Links
related:: [[_INDEX]]
tags: python, qdrant, memory
