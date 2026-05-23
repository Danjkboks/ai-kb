---
type: ref
domain: llm
source: created
status: active
tags: [haiku, sonnet, opus, model-selection, cost, decision-tree]
created: 2026-05-19
updated: 2026-05-21
---

# Model Selection Framework

**Purpose:** Evaluate task complexity and recommend the optimal Claude model (Haiku, Sonnet, Opus) to minimize debugging and token waste.

**Last Updated:** 2026-05-19 | **Status:** Production

---

## Quick Reference

| Task Type | Complexity | Model | Reason |
|---|---|---|---|
| Read/summarize files | Low | **Haiku** | No reasoning needed |
| Execute script/command | Low | **Haiku** | Straightforward execution |
| Write simple code (<50 lines) | Low | **Haiku** | Template-based, no edge cases |
| List/organize information | Low | **Haiku** | Formatting, not reasoning |
| **Debug script errors** | **Medium** | **Sonnet** | Needs analysis of root cause |
| **Code generation (100+ lines)** | **Medium** | **Sonnet** | Handle edge cases, validation |
| **Architecture decisions** | **Medium-High** | **Sonnet** | Trade-off analysis, patterns |
| **Session analysis/pattern extraction** | **Medium-High** | **Sonnet** | Understand context, find subtle patterns |
| **Complex multi-step workflows** | **High** | **Opus** | Coordination, state management |
| **Novel problem solving** | **High** | **Opus** | Creative reasoning required |
| **Research synthesis** | **Medium** | **Sonnet** | Evaluate sources, judge relevance |

---

## Evaluation Criteria

### 1. **Reasoning Complexity** (Does it need to *think*?)

**Low:** Pattern matching, rule application, straightforward derivation
- Examples: execute script, format data, read file, simple sed/awk
- **→ Haiku**

**Medium:** Analyze context, identify patterns, handle edge cases
- Examples: debug error message, analyze session transcript, create SOP from example
- **→ Sonnet**

**High:** Multi-step reasoning, creative problem-solving, novel approaches
- Examples: redesign architecture, resolve deadlock, invent new pattern
- **→ Opus**

### 2. **Error Consequence** (What happens if it's wrong?)

**Low:** Easy to verify, quick to fix
- Examples: script syntax, file formatting, list ordering
- Debugging cost: < 5 min
- **→ Haiku OK**

**Medium:** Requires investigation, but fixable
- Examples: API call wrong parameters, incomplete code, logic bug
- Debugging cost: 15-60 min
- **→ Sonnet recommended**

**High:** Cascading failures, hard to debug
- Examples: memory corruption, architectural mistake, security vulnerability
- Debugging cost: hours to days
- **→ Opus required**

### 3. **Domain Expertise Needed** (Does it need to know your context?)

**Low:** General task, no project context
- Example: "write a Python function to parse CSV"
- **→ Haiku** (or delegate to Sonnet if complex)

**Medium:** Requires understanding your stack, patterns, or recent decisions
- Example: "create a new n8n workflow following our pattern"
- **→ Sonnet** (knows your context, applies patterns correctly)

**High:** Requires deep understanding of codebase, past decisions, long-term memory
- Example: "should we switch to a different agent architecture? pros/cons vs our current approach?"
- **→ Opus** (or Sonnet with full context loaded)

### 4. **Output Quality Sensitivity** (Does mediocre output cost you?)

**Low:** Output is close enough if correct
- Example: directory listing, file copy, script execution
- **→ Haiku**

**Medium:** Output must be well-reasoned but doesn't need to be perfect
- Example: SOP draft (you'll review/edit), code (you'll test)
- **→ Sonnet**

**High:** Output sets the foundation for future work (technical debt risk)
- Example: architecture decision, memory system design, skill definition
- **→ Opus**

### 5. **Token Efficiency Sensitivity** (Is token waste painful?)

**Low:** Low token cost either way (read-only, simple write)
- **→ Haiku** (save tokens where possible)

**Medium:** Moderate token cost, but output quality matters more
- **→ Sonnet** (justified spend)

**High:** One wrong output means re-running with more context
- **→ Opus** (get it right first time)

---

## Decision Tree

```
Start: Evaluate task complexity
│
├─ Is it straightforward execution/formatting?
│  YES → Haiku
│  NO → Continue
│
├─ Does it require understanding context or patterns?
│  YES → Evaluate error consequence
│  NO → Can it be done with general knowledge?
│      YES → Haiku
│      NO → Sonnet
│
├─ If error, how hard to debug?
│  < 5 min → Haiku OK (if no context needed)
│  15-60 min → Sonnet
│  hours+ → Opus
│
├─ Does it require creative/novel reasoning?
│  YES → Opus
│  NO → Sonnet or Haiku
│
└─ Final: Choose model based on dominant factor
```

---

## Examples

### Example 1: Run a PowerShell script
- Complexity: Low (execute command)
- Error consequence: Low (easy to retry)
- Context needed: Low (self-contained script)
- Quality sensitive: Low (works or fails)
- **→ Haiku** ✓

### Example 2: Create a new SOP from session conversation
- Complexity: Medium (understand context, distill essence)
- Error consequence: Medium (wrong SOP is confusing, takes time to fix)
- Context needed: Medium (understand your patterns)
- Quality sensitive: Medium (you'll review and adjust)
- **→ Sonnet** ✓

### Example 3: Debug a session where ask.py failed mysteriously
- Complexity: Medium-High (trace through layers, understand error context)
- Error consequence: High (needs proper diagnosis)
- Context needed: High (understand LLMLingua, Docker, API flow)
- Quality sensitive: High (wrong diagnosis wastes hours)
- **→ Sonnet or Opus** (Sonnet if straightforward, Opus if subtle)

### Example 4: Should we refactor the agent architecture?
- Complexity: High (trade-off analysis, creative thinking)
- Error consequence: High (wrong decision = technical debt)
- Context needed: High (your stack, your constraints)
- Quality sensitive: High (sets direction for months)
- **→ Opus** ✓

### Example 5: Extract learnings from 10 session transcripts
- Complexity: Medium (pattern recognition, synthesis)
- Error consequence: Medium (missed patterns, but can re-run)
- Context needed: Medium (understand your project)
- Quality sensitive: Medium (quality affects future decisions)
- **→ Sonnet** ✓

### Example 6: List the files in a directory
- Complexity: Low (trivial)
- Error consequence: Low (obvious if wrong)
- Context needed: Low (no context)
- Quality sensitive: Low (binary: right or wrong)
- **→ Haiku** ✓

---

## Special Cases

### When to Upgrade from Haiku to Sonnet
- ✓ Task is "simple" but you've been burned by bugs before
- ✓ Output will be reused/extended (SOP, code template, decision document)
- ✓ Debugging would require deep context (e.g., tracing through your stack)
- ✓ You're uncertain about the correct approach

### When to Downgrade from Sonnet to Haiku
- ✓ Task is clearly straightforward (copy file, rename, execute)
- ✓ You can quickly verify the output
- ✓ Error consequence is truly low
- ✓ You're in a 100-task batch (some tasks are simple filler)

### When to Use Opus
- ✓ Output will set direction for 1+ months of work
- ✓ Debugging would require 2+ hours
- ✓ You're stuck and need creative reasoning
- ✓ High-stakes decision (architecture, security, budget)

---

## Model Capabilities Reference

### Haiku 4.5
- **Strengths:** Fast, cheap, good at straightforward tasks
- **Weaknesses:** Limited reasoning depth, struggles with edge cases
- **Best for:** Execution, data transformation, simple code, summaries
- **Cost:** ~1x (baseline)
- **Speed:** Fastest
- **Reasoning depth:** Shallow (1-2 steps)

### Sonnet 4.6
- **Strengths:** Balanced—good reasoning, reliable code, fast enough
- **Weaknesses:** Sometimes over-thinks simple tasks (wastes tokens)
- **Best for:** Code generation, debugging, pattern analysis, synthesis
- **Cost:** ~2-3x Haiku
- **Speed:** Medium
- **Reasoning depth:** Deep (5-10 steps)

### Opus 4.7
- **Strengths:** Best reasoning, handles complex multi-step, creative
- **Weaknesses:** Slow, expensive
- **Best for:** Architecture, novel problems, creative solutions, complex debugging
- **Cost:** ~4-6x Haiku
- **Speed:** Slowest
- **Reasoning depth:** Deepest (10+ steps, multi-domain)

---

## Applying This Framework

### For Agents (Memory Architect, Research Agent, etc.)

**Specify in MANIFEST.md or agent definition:**
```
Recommended Models:
- Session ingestion: Sonnet (pattern extraction, medium complexity)
- SOP creation: Sonnet (understands context, quality sensitive)
- Memory audit: Haiku (straightforward list/check)
- Research synthesis: Sonnet (evaluate sources, judge relevance)
- Trigger checking: Haiku (simple JSON logic)
```

### For Project Tasks

**At task time, human evaluates:**
```
Task: "Create a routing.json for new project"
→ Complexity: Medium (design a structure that will be reused)
→ Error consequence: Medium (bad routing = wasted context on every task)
→ Recommended model: Sonnet
→ Rationale: Design quality matters; wrong structure cascades
```

### For Claude Code Sessions

**Set model for session based on task:**
```
Session 1: "Deploy the Docker container" → Haiku (execution)
Session 2: "Debug why the API call fails" → Sonnet (analysis)
Session 3: "Should we refactor the memory system?" → Opus (reasoning)
```

---

## Budget Impact

**Rough costs (OpenRouter pricing, 2026-05):**
- Haiku: $0.80 / 1M input tokens, $2.40 / 1M output
- Sonnet: $3 / 1M input, $15 / 1M output
- Opus: $15 / 1M input, $60 / 1M output

**Example workflow (50€/month budget):**
- 80% of tasks = Haiku (cheap, fast)
- 15% of tasks = Sonnet (balanced)
- 5% of tasks = Opus (when needed)
- Result: Opus sessions feel "free" because most work is Haiku

---

## Approval Checklist

Before running a task with a selected model:

- [ ] Complexity level identified
- [ ] Error consequence assessed
- [ ] Model recommended based on framework
- [ ] User approves OR model was pre-agreed
- [ ] (If upgrading from Haiku) Understand why the extra cost is justified
- [ ] (If using Opus) Confirm this is high-stakes enough

---

## Feedback & Refinement

This framework is a living document. Update it as you learn:
- Tasks where Haiku surprised you (good or bad)
- Tasks where you wished you'd used a different model
- New task categories that need their own rule
- Better heuristics for deciding
