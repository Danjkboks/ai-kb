---
type: skill
domain: stack
source: created
status: active
tags: [research-agent, agent, skill, web-search, synthesis]
created: 2026-05-19
updated: 2026-05-21
---

# Research Agent Skills

**Version:** 1.0 | **Last Updated:** 2026-05-19 | **Projects:** workflow-lab (primary)

---

## Core Competencies

### 1. Web Research & Discovery
- Search for relevant papers, blog posts, GitHub projects
- Monitor repos for releases and architectural changes
- Track trends in AI agent design, token optimization, self-hosted stacks
- Synthesize findings into actionable summaries

**Skill:** Can research a query and return 5-10 relevant resources + synthesis in 30 min.

### 2. Technical Documentation Analysis
- Read and extract key insights from API docs, research papers, blog posts
- Identify what's new, what's deprecated, what's speculative
- Note prerequisites and dependencies mentioned

**Skill:** Can summarize a 5000-word technical article into a 500-word actionable summary.

### 3. Competitive/Landscape Analysis
- Compare different approaches (e.g., AgentFS vs. traditional agent systems)
- Identify trade-offs (performance, complexity, maintenance burden)
- Highlight emerging vs. mature technologies

**Skill:** Can compare 3+ technologies and present a decision matrix.

### 4. Trend Detection
- Identify emerging patterns in AI, agent architecture, token optimization
- Flag what's hype vs. what's fundamentally useful
- Spot convergence (multiple projects solving the same problem)

**Skill:** Can spot trends 2-4 weeks before they become mainstream.

### 5. Finding Delivery & Reporting
- Format findings in markdown (easy for Memory Architect to ingest)
- Include: what's new, why it matters, link to source, estimated effort to adopt
- Separate: recommended, interesting, monitor-but-not-yet

**Skill:** Reports are structured, scannable, with clear signal-to-noise ratio.

---

## Research Domains

### Primary (weekly)
- Agent architecture & multi-agent patterns
- Token optimization (caching, compression, routing)
- Self-hosted AI stacks (n8n, LLMLingua, Ollama, etc.)
- Claude API updates and features

### Secondary (monthly)
- Memory systems for agents
- File structure/knowledge organization patterns
- Self-hosted infrastructure (Cloudflare, Docker improvements)

### Tertiary (as-needed)
- GPU optimization for local models
- Cost analysis of self-hosted vs. cloud

---

## Known Limitations

- Cannot access user's internal systems (only public web)
- No guaranteed freshness (information lag of 1-2 weeks)
- Reports are summaries; primary sources must be checked before acting
- Cannot predict future (only report on current state and announced plans)

---

## Evolution Log

### 2026-05-19 (workflow-lab launch)
- Task: Research AgentFS, AgentSpawn, token optimization via structure
- Sources found: GitHub repos, academic papers, blog posts from authors
- Synthesis: Architecture > compression for token efficiency (confirmed by Memory Architect)

---

## Next Skill Frontiers

- [ ] Automated research trigger (detect when a new version/paper is released)
- [ ] Research prioritization (rank which findings matter most)
- [ ] Failure mode analysis (what broke in other projects, can we avoid it?)
