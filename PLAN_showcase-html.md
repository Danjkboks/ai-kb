# PLAN — AI Infrastructure Showcase HTML
> Created: 2026-06-09 | Surface: Chat | Execute in: Claude Code at D:\aidirectory\
> Output: D:\aidirectory\projects\showcase\index.html

---

## Context

Single self-contained HTML file. Purpose: show a non-technical potential client the architecture,
thinking, and real value behind a self-hosted AI memory infrastructure. Not a portfolio. A pitch.
No frameworks. No external dependencies except Inter font from Google Fonts. Vanilla JS only.
All content hardcoded.

Design reference: dark product UI (similar to Attio/Linear dark mode).
Image reference: D:\aidirectory\knowledge\3_Dark_Mode_Table.png

---

## Design tokens (CSS variables — define at :root)

```css
--bg-page:        #0f0f11;
--bg-card:        #1a1b1f;
--bg-element:     #25262c;
--bg-hover:       #2e2f38;
--bg-selected:    #1e3a5f;
--accent-blue:    #4478c4;
--accent-purple:  #8b6fd4;
--accent-green:   #3d9e6e;
--accent-amber:   #e8913a;
--accent-red:     #d95c5c;
--accent-teal:    #3abfb8;
--text-primary:   #e2e2e4;
--text-secondary: #8b8c96;
--text-muted:     #5a5b63;
--border:         #2e2f38;
--radius-sm:      6px;
--radius-md:      10px;
--radius-lg:      16px;
```

---

## Global styles

- Font: Inter from Google Fonts (weights 400, 500)
- Body: `background: var(--bg-page)`, `color: var(--text-primary)`, `font-family: Inter, sans-serif`
- Max content width: 1200px, centered, `padding: 0 32px`
- All transitions: `0.2s ease` unless specified
- No box-shadows — depth via background color stepping only
- Borders: `1px solid var(--border)` everywhere

---

## Section 1 — Hero

- Full width, `padding: 80px 0 60px`
- Headline: `"AI that remembers, learns, and improves itself."` — 38px, weight 500, `--text-primary`
- Subheadline: `"A self-hosted intelligence infrastructure built for sovereignty, efficiency, and continuous improvement — without sending your data anywhere."` — 18px, `--text-secondary`, max-width 680px
- Three pill badges row, `margin-top: 24px`, `gap: 12px`:
  - `🔒 Fully local` — bg `--bg-element`, border `--border`
  - `⚡ Token-efficient` — same
  - `🔁 Self-improving` — same
  - Each pill: `padding: 6px 14px`, `border-radius: 20px`, `font-size: 13px`, `--text-secondary`
- Thin `<hr>` divider below: `border: none; border-top: 1px solid var(--border); margin: 48px 0 0`

---

## Section 2 — The interactive map

Section label above: `"HOW IT WORKS"` — 11px, letter-spacing 2px, `--accent-blue`, `--text-muted` mix
Section title: `"Every session becomes structured knowledge."` — 26px, weight 500
Section subtitle: `"A five-stage pipeline captures, compresses, extracts, stores, and feeds back everything learned."` — 15px, `--text-secondary`

### Map container

- `display: flex`, `align-items: flex-start`, `gap: 0`, `margin-top: 40px`
- Horizontally scrollable on small screens: `overflow-x: auto`
- Five zone cards + four arrow connectors between them

### Zone card (default/collapsed state)

- Width: `180px`, fixed. Height: auto.
- `background: var(--bg-card)`, `border: 1px solid var(--border)`, `border-radius: var(--radius-lg)`
- `padding: 20px 16px`
- Top: zone number badge — small circle, accent color bg (per zone), white number, 11px
- Zone label: 11px, letter-spacing 1.5px, uppercase, accent color (per zone), `margin-top: 8px`
- Icon: 28px emoji or SVG, `margin: 12px 0`
- One-line description: 12px, `--text-secondary`
- Bottom: `"↓ explore"` — 11px, `--text-muted`, cursor pointer
- Top border: `3px solid <accent color>` (replaces standard border-top)
- Hover state: `background: var(--bg-hover)`, border accent color brightens slightly

### Zone card (expanded state — class `is-open`)

- Same card, expands below the one-liner to reveal component grid
- `max-height` transitions from `0` to `600px` over `0.3s ease`
- Component grid: `display: grid`, `grid-template-columns: 1fr`, `gap: 8px`, `margin-top: 16px`
- Divider line `1px solid var(--border)` above the component grid

### Component card (inside expanded zone)

- `background: var(--bg-element)`, `border-radius: var(--radius-sm)`, `padding: 10px 12px`
- Component name: 13px, weight 500, `--text-primary`
- Role pill: 10px, colored bg (15% opacity of accent), matching text color, `border-radius: 4px`, `padding: 2px 7px`
- Two-line description: 11px, `--text-secondary`, `margin-top: 4px`
- `position: relative` — tooltip anchored here

### Arrow connectors between zones

- A `<div class="zone-arrow">` between each pair of zones
- Width: `60px`, `display: flex`, `flex-direction: column`, `align-items: center`, `justify-content: center`, `padding-top: 40px`
- SVG arrow: horizontal, `--text-muted` stroke, 1px, simple chevron head
- Label below arrow: 10px, `--text-muted`, text-align center, max 2 lines

Arrow labels (left to right):
1. Between Zone 1→2: `"raw session file (.jsonl)"`
2. Between Zone 2→3: `"base64 payload"`
3. Between Zone 3→4: `"structured YAML extract"`
4. Between Zone 4→5: `"indexed chunks + patterns"`

### Zone definitions

**Zone 1 — The Work** (accent: `--accent-purple`)
- Icon: 💬
- Description: `"Where AI sessions happen"`
- Components:
  - `Claude Code` | role: `Execution` | `Writes code, scripts, agents. Every session auto-saved as a raw file.`
  - `Claude Chat` | role: `Planning` | `Architecture decisions, research, design. Cheaper context, no file writes.`

**Zone 2 — The Capture** (accent: `--accent-amber`)
- Icon: 📡
- Description: `"Nothing escapes"`
- Components:
  - `Session Watcher` | role: `Monitor` | `Background script watching for new session files. Sends them automatically.`
  - `Windows Task Scheduler` | role: `Trigger` | `Runs the watcher on a fixed schedule. Survives restarts.`
  - `Dedup Log` | role: `Guard` | `Tracks processed files. Prevents double-processing on retry.`

**Zone 3 — The Brain** (accent: `--accent-blue`)
- Icon: ⚙️
- Description: `"Compress, extract, structure"`
- Components:
  - `n8n Workflow` | role: `Orchestrator` | `Automation engine. Coordinates every step from receipt to storage.`
  - `LLMLingua` | role: `Compressor` | `Shrinks session transcripts by up to 80% before LLM processing.`
  - `DeepSeek V3` | role: `Extractor` | `Reads compressed session, outputs structured knowledge in YAML.`

**Zone 4 — The Memory** (accent: `--accent-teal`)
- Icon: 🧠
- Description: `"Structured, searchable, local"`
- Components:
  - `Obsidian Vault` | role: `Storage` | `Local folder of knowledge files. Human-readable. Git-tracked.`
  - `Qdrant` | role: `Search` | `Vector search engine. Finds relevant past context in milliseconds.`
  - `Session Extracts` | role: `Records` | `Structured summaries: decisions, errors, patterns, next steps.`
  - `Pattern Index` | role: `Intelligence` | `Mined recurring behaviors across all sessions.`

**Zone 5 — The Loop** (accent: `--accent-green`)
- Icon: 🔁
- Description: `"The system improves itself"`
- Components:
  - `/wrap command` | role: `Capture` | `Run at session end. Writes extract, updates handover for next session.`
  - `/integrity agent` | role: `Audit` | `Reads all past sessions. Finds contradictions, stale rules, repeated mistakes.`
  - `mine_patterns.py` | role: `Mining` | `Scans extracts for recurring themes. Proposes new rules automatically.`
  - `CLAUDE.md` | role: `Rulebook` | `Living instruction set. Updated when patterns are confirmed. Governs every future session.`

---

## Section 3 — Why this matters

Section label: `"THE PROBLEM WE'RE SOLVING"`
Section title: `"Generic AI has a memory problem. This fixes it."` — 26px
Section subtitle: `"Most AI tools forget everything the moment a session ends. Here's what that costs — and what a structured approach changes."` — 15px, `--text-secondary`

Six cards, `display: grid`, `grid-template-columns: repeat(auto-fit, minmax(340px, 1fr))`, `gap: 20px`, `margin-top: 40px`

Each card structure:
- `background: var(--bg-card)`, `border: 1px solid var(--border)`, `border-left: 3px solid <accent>`, `border-radius: var(--radius-lg)`, `padding: 24px`
- Icon: 22px emoji, top left
- Title: 16px, weight 500, `--text-primary`, `margin: 10px 0 8px`
- Body: 14px, `--text-secondary`, line-height 1.7, 3–4 sentences
- Bottom contrast block: `margin-top: 16px`, `padding-top: 14px`, `border-top: 1px solid var(--border)`
  - `"Generic AI →"` label: 11px, `--accent-red`, + problem text in `--text-muted`
  - `"This approach →"` label: 11px, `--accent-green`, + solution text in `--text-secondary`

**Card 1 — The context problem** (accent: `--accent-blue`)
- Icon: 🔄
- Title: `"AI forgets everything between sessions"`
- Body: `"Every time you open a new conversation, the AI starts from zero. It has no memory of what you built yesterday, what failed last week, or what decision you made three months ago. For complex, evolving work this is crippling — you spend more time re-explaining than doing."`
- Generic: `"Starts every session blind. Re-explaining context wastes 20–30% of every session."`
- This approach: `"Every session is captured, structured, and indexed. The next session starts informed."`

**Card 2 — Hallucination & coherence** (accent: `--accent-purple`)
- Icon: 🌀
- Title: `"Larger datasets make AI less reliable, not more"`
- Body: `"When an AI model has too much context to process, it starts confusing, contradicting, and hallucinating. The problem gets worse as your project grows. Most tools have no answer for this — they just hope the model handles it. It doesn't."`
- Generic: `"Incoherence grows with dataset size. No mechanism to detect or correct it."`
- This approach: `"Integrity agent detects contradictions across sessions. Grounded responses from real history."`

**Card 3 — Data sovereignty** (accent: `--accent-teal`)
- Icon: 🔒
- Title: `"Your knowledge is your competitive edge"`
- Body: `"Every session you have with an AI is generating valuable intellectual property — decisions, solutions, patterns, architecture. When that runs on someone else's cloud, you don't control it, you don't own it, and you can't audit what happens to it."`
- Generic: `"Data processed on third-party servers. No auditability. Potential training data contribution."`
- This approach: `"Everything runs locally. No data leaves your machine. Full audit trail in plain files."`

**Card 4 — Efficiency & cost** (accent: `--accent-amber`)
- Icon: ⚡
- Title: `"Token bloat is silent and expensive"`
- Body: `"Loading entire session histories into every AI call is wasteful. A raw session export can be 2–3MB. Sending that to an LLM for every query is slow, expensive, and counterproductive — you're mostly paying to process noise."`
- Generic: `"Full context loaded every time. Costs scale linearly with project size."`
- This approach: `"LLMLingua compresses by up to 80%. Qdrant returns only relevant chunks. Costs stay flat."`

**Card 5 — Self-improvement** (accent: `--accent-green`)
- Icon: 📈
- Title: `"Most AI setups don't learn from mistakes"`
- Body: `"When an AI makes the same error twice, that's a process failure. When it makes it ten times, that's a structural problem. This system mines its own history for recurring failures and automatically writes new rules to prevent them. It gets measurably better with every session."`
- Generic: `"No feedback loop. Same mistakes repeat indefinitely. No mechanism for improvement."`
- This approach: `"Pattern mining → rule proposals → CLAUDE.md update → better next session. Closed loop."`

**Card 6 — Future-ready architecture** (accent: `--accent-red`)
- Icon: 🚀
- Title: `"Built to scale, not to be replaced"`
- Body: `"This isn't a tool — it's a foundation. Every component is modular, local, and replaceable. As better models arrive, as your data grows, as your needs evolve — the architecture adapts. You're not locked into any vendor, any model, or any format."`
- Generic: `"Vendor-locked. Migration means starting over. Data stuck in proprietary formats."`
- This approach: `"Open formats, local storage, model-agnostic. Swap any component without losing history."`

---

## Section 4 — The agents

Section label: `"THE TEAM"`
Section title: `"Seven specialist agents. Each with a role, permissions, and hard limits."` — 26px
Section subtitle: `"Agents don't do everything — they do one thing well, and nothing outside their boundary."` — 15px

Horizontal flex row, `overflow-x: auto`, `gap: 16px`, `padding-bottom: 16px`, `margin-top: 40px`

Each agent card:
- `min-width: 200px`, `background: var(--bg-card)`, `border: 1px solid var(--border)`, `border-left: 3px solid <accent>`, `border-radius: var(--radius-lg)`, `padding: 20px`
- Agent name: 14px, weight 500
- Role sentence: 12px, `--text-secondary`, `margin: 6px 0 12px`
- Two tags row:
  - Domain tag: colored pill
  - `"Cannot:"` tag: red-tinted pill with what it's blocked from

Agents and accent colors:
1. `Memory Architect` — `--accent-purple` — domain: `memory/audit` — cannot: `modify code files`
2. `n8n Workflow` — `--accent-blue` — domain: `automation` — cannot: `run arbitrary commands`
3. `Docker Ops` — `--accent-teal` — domain: `containers` — cannot: `access outside aidirectory`
4. `Python Ops` — `--accent-amber` — domain: `scripts/compression` — cannot: `store API keys in logs`
5. `Git Ops` — `--accent-green` — domain: `version control` — cannot: `push without diff review`
6. `Infra Ops` — `--accent-red` — domain: `network/tunnel` — cannot: `modify system settings`
7. `Research Agent` — `--text-secondary` accent — domain: `knowledge/web` — cannot: `write to vault directly`

---

## Section 5 — Future implementations

Section label: `"WHAT'S NEXT"`
Section title: `"The roadmap is already in the system."` — 26px
Section subtitle: `"These aren't wishlist items. They're identified gaps from integrity agent findings and pattern analysis."` — 15px

Vertical timeline list, `margin-top: 40px`:
- Left border line: `2px solid var(--border)`, `margin-left: 16px`
- Each item: circle dot on the line (8px, `--bg-element`, `border: 2px solid <accent>`), then card to the right

Six items:

1. (accent-blue) `"n8n execution poller"` — `"Surface pipeline failures automatically. Today, a failed extraction is invisible unless you check manually. A poller fixes that."`
2. (accent-green) `"Scheduled integrity runs"` — `"Auto-trigger /integrity every 10 sessions. Drift detection becomes proactive, not reactive."`
3. (accent-teal) `"Qdrant knowledge synthesis"` — `"Auto-generate summary documents when the vault hits size thresholds. Keeps search fast as data grows."`
4. (accent-amber) `"Multi-surface token dashboard"` — `"Real-time view of token spend across Chat, Code, and Cowork. Today this is estimated manually."`
5. (accent-purple) `"Automated session quality scoring"` — `"Score each session on efficiency, error rate, and goal completion. Feed scores back into pattern mining."`
6. (accent-red) `"SME workflow library"` — `"Tested, documented n8n workflow templates ready for client deployment. With runbooks and handoff SOPs."`

---

## Section 6 — Footer

`padding: 60px 0 40px`, `border-top: 1px solid var(--border)`, `margin-top: 80px`
`text-align: center`

Line 1 (16px, `--text-secondary`): `"Built on a Windows 11 laptop. No cloud. No external APIs except one LLM gateway. Everything else: yours."`
Line 2 (13px, `--text-muted`, margin-top 12px): `"Self-hosted · Local-first · Model-agnostic · Open formats"`

---

## Tooltip system (JS)

Single `<div id="tooltip">` appended to body, `position: fixed`, `z-index: 1000`.

Styles:
```css
#tooltip {
  background: var(--bg-element);
  border: 1px solid var(--border);
  border-radius: var(--radius-md);
  padding: 12px 14px;
  max-width: 260px;
  font-size: 12px;
  line-height: 1.6;
  pointer-events: none;
  opacity: 0;
  transition: opacity 0.15s ease;
}
#tooltip.visible { opacity: 1; }
#tooltip .tt-title { font-size: 13px; font-weight: 500; color: var(--text-primary); margin-bottom: 4px; }
#tooltip .tt-body  { color: var(--text-secondary); }
#tooltip .tt-why   { color: var(--text-muted); margin-top: 6px; font-size: 11px; border-top: 1px solid var(--border); padding-top: 6px; }
```

JS behavior:
- Every element with `data-tooltip-title`, `data-tooltip-body`, `data-tooltip-why` attributes triggers the tooltip
- On `mouseenter`: populate tooltip divs, position near cursor (clamp to viewport), add class `visible`
- On `mousemove`: reposition
- On `mouseleave`: remove class `visible`

---

## Zone expand/collapse (JS)

- Every zone card has `data-zone="1"` etc. and a toggle button
- Click toggles class `is-open` on the zone card
- CSS: `.zone-card .zone-components { max-height: 0; overflow: hidden; opacity: 0; transition: max-height 0.3s ease, opacity 0.2s ease; }`
- CSS: `.zone-card.is-open .zone-components { max-height: 600px; opacity: 1; }`
- Toggle button text: `"↓ explore"` → `"↑ collapse"` on open

---

## Output

Write final file to: `D:\aidirectory\projects\showcase\index.html`
Create the `showcase` directory if it doesn't exist.

---

## Prompt for Claude Code session

```
Read D:\aidirectory\knowledge\PLAN_showcase-html.md in full before writing a single line.

Build index.html exactly to the spec. Rules:
- Single file, no frameworks, vanilla JS only
- All CSS variables defined at :root as specified
- Inter font from Google Fonts
- All content hardcoded — no fetch, no external data
- Build section by section in order: global styles → hero → map → why → agents → roadmap → footer
- Write the complete file, do not truncate
- Output path: D:\aidirectory\projects\showcase\index.html (create directory if missing)
```
