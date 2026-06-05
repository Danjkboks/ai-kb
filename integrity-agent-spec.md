# Integrity Agent — Design File
> type: design | date: 2026-05-24 | status: brainstorming-pending
> Next action: open new Chat, read this file, brainstorm before any implementation

---

## Purpose

System-wide audit agent. Reads the full stack across sessions and surfaces problems
no individual session can see: cross-session drift, inconsistent implementations,
bad patterns, predicted failure modes, recurring issues.

---

## Implementation Target (decided)

- PRIMARY: Claude Code slash command `/integrity`
  Claude reads files directly, reasons natively, writes output. No API call, no cost.
- SECONDARY: `integrity-check.ps1` PS7 script
  Calls OpenRouter, runs scheduled or headless via Task Scheduler.
- NOT n8n. n8n adds no value here over a direct PS7 script.

Output: `D:\aidirectory\knowledge\audits\REVIEW_<date>.md`

---

## Open Design Questions (brainstorm these before building)

### 1. Scope
- What does this agent actually audit? What's in vs out of scope?
- What inputs are genuinely useful vs noise?
- What questions should it be answering?

### 2. Output structure
- What sections make a review actually useful vs a wall of text nobody acts on?
- What severity model? What format?

### 3. Prompt design
- How to get high-signal output without hallucination on a meta-reasoning task?
- How to structure context so the LLM reasons across documents efficiently?
- Known failure modes of this type of task?

### 4. Token efficiency
- Inputs can be large: 20 extracts + 10 audits + scripts + commands
- LLMLingua available at http://host.docker.internal:5001/compress (~46% savings)
- What compression strategy? What must never be compressed?
- What's the right context budget?

### 5. Relevancy
- How to avoid surfacing obvious or stale issues?
- Should it track previous reviews to avoid repetition?

### 6. Triggers and frequency
- When does this run? On-demand only first, then scheduled.
- Right cadence for signal vs noise?

### 7. Integration
- How does output feed back into the workflow?
- Should it update HANDOVER.md [SHARED]? Just write a file?
- Should /handover optionally trigger it?

### 8. Failure modes of the agent itself
- What if it hallucinates a critical issue?
- How to validate its output?
- Could a poorly-scoped agent add noise and make things worse?

---

## Constraints (non-negotiable)
- Token efficiency at every level
- Output must be actionable — if it doesn't change behavior, it's waste
- Build slash command first, schedule later
- No over-engineering

---

## Stack Context
- Surfaces: Claude Code, Chat (Filesystem connector), Cowork
- LLM routing: OpenRouter only | Kimi K2 (coding), DeepSeek V3.2 (extraction), Sonnet 4 (reasoning)
- Memory pipeline: /wrap → EXTRACT → n8n → DeepSeek → ai-kb/extracts/ (GitHub)
- HANDOVER.md: 4 sections [SHARED][CHAT][COWORK][CODE] — each surface owns one section
- Scripts: D:\aidirectory\scripts\ | Knowledge: D:\aidirectory\knowledge\
