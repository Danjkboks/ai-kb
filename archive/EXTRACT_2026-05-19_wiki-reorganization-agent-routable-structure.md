---
type: extract
date: 2026-05-19
session_id: wiki-reorganization-agent-routable-structure
surface: claude-code
environment: env1-claude-desktop
topics: [memory, n8n, docker, python, infra]
source_file: 2026-05-19-232204_24c995e7-0bd5-4d8e-bc39-cbfeaf3f227c.jsonl
processed_at: 2026-05-23T22:53:08.676Z
---

# Session Extract: wiki-reorganization-agent-routable-structure

## Decisions
- **Adopt semantic prefix naming convention for wiki files (sop_, guide_, runbook_, reference_)**: Enables agent discovery via glob patterns and semantic routing; replaces chaotic mixed naming
- **Organize files into agent-routable directories by primary responsibility (claude-code, n8n, docker, python, infra, memory, general, reference)**: Creates predictable structure where agents know where to look for documentation; supports component-based discovery
- **Add YAML frontmatter to all files with agent-type, tags, status, and updated date**: Enables semantic filtering and machine-readable metadata for agent querying; supports status filtering (stable/draft/deprecated)
- **Create central INDEX.md as discovery map with agent routing information**: Provides single source of truth for agents to understand documentation structure; maps topics to files and agent responsibilities

## Problems Solved
- **Chaotic mixed naming conventions in wiki files (SOPs, guides, playbooks without consistent prefixes)**: Reorganized with semantic prefixes (sop_, guide_, runbook_, reference_) and moved to agent-specific directories
- **No agent-friendly discovery mechanism for documentation**: Created INDEX.md with agent routing map, YAML frontmatter on all files, and directory structure organized by agent responsibility
- **Duplicate and outdated files cluttering wiki root**: Moved 16 old files to archive/ directory, keeping only reorganized files with new naming convention

## Errors Encountered
- [resolved] Bash mv commands failed with 'command not found' due to PowerShell environment -> Used PowerShell-compatible Move-Item with -ErrorAction SilentlyContinue parameter
- [resolved] Directory names with spaces caused glob pattern issues -> Used quoted paths and proper PowerShell escaping

## Patterns Identified
- Agent-first documentation requires both semantic naming AND directory structure
- YAML frontmatter enables both human and machine readability of documentation metadata
- Flat hierarchy with prefixes is more agent-friendly than deep nested structures
- PowerShell requires different syntax than bash for file operations in Windows environment

## Files Modified
- created: D:\lab\wiki\claude-code\sop_windows-full-setup.md -- Claude Code + OpenRouter Windows setup SOP with YAML frontmatter
- created: D:\lab\wiki\claude-code\sop_vscode-openrouter-windows.md -- French version of VS Code + OpenRouter setup with YAML frontmatter
- created: D:\lab\wiki\n8n\sop_webhook-agent-bridge.md -- n8n webhook bridge with Cloudflare tunnel setup, YAML frontmatter for n8n-workflow agent
- created: D:\lab\wiki\memory\sop_obsidian-github-agent-memory.md -- Obsidian + GitHub sync for agent memory with YAML frontmatter
- created: D:\lab\wiki\python\sop_scrapegraphai-agent-wiki.md -- ScrapeGraphAI ingestion pipeline with YAML frontmatter for python-ops agent
- created: D:\lab\wiki\docker\runbook_master-build.md -- MASTER_BUILD converted to runbook format with YAML frontmatter for docker-ops agent
- created: D:\lab\wiki\general\reference_master-build.md -- Reference version of MASTER_BUILD for memory-agent
- created: D:\lab\wiki\memory\guide_ai-memory-architecture.md -- AI memory architecture guide with Qdrant + Ollama setup
- created: D:\lab\wiki\reference\reference_ai-agent-models-2026.md -- AI model recommendations 2026 with YAML frontmatter
- created: D:\lab\wiki\claude-code\sop_environment-setup-kilo-2026.md -- Kilo Code + Qdrant + Ollama environment setup 2026
- created: D:\lab\wiki\INDEX.md -- Central discovery map with agent routing information and file catalog
- created: D:\lab\wiki\archive\ -- Directory for old unorganized files (16 files moved here)

## Next Session Must Know
- Wiki reorganized into agent-routable structure at D:\lab\wiki\ with directories: claude-code, n8n, docker, python, infra, memory, general, reference, archive
- All files now have YAML frontmatter with agent-type, tags, status, updated date
- INDEX.md at D:\lab\wiki\INDEX.md is central discovery map for agents
- Naming convention: sop_*, guide_*, runbook_*, reference_* prefixes
- Old files moved to archive/ directory (16 files)
- PowerShell file operations require Move-Item not mv, with -ErrorAction SilentlyContinue
- infra/ directory created but empty - needs Cloudflare tunnel docs
- obsidian_guide.md (French second brain guide) not reorganized - still in root

## Skill Candidates
- wiki-reorganize-agent-structure: Reorganizes flat wiki into agent-routable directories with semantic naming and YAML frontmatter
- wiki-create-index-map: Generates INDEX.md discovery map from existing wiki structure with agent routing

## Token Waste Flags
- Repeated reading of same file contents multiple times during reorganization
- Generated full file contents in tool calls instead of just metadata
- Multiple directory listing commands with similar parameters

## Knowledge Base Updates
- [create] sop_memory_wiki-reorganization.md: Document the agent-routable wiki structure pattern for future reference and replication
- [update] ref_memory_agent-context.md: Add wiki discovery patterns and INDEX.md usage for memory agent context loading
- [create] guide_wiki_yaml-frontmatter.md: Document YAML frontmatter schema for agent discoverability and status tracking
