# Feature Proposal: Context Intelligence System for Google Antigravity

> **Author:** Montassar Werteni  
> **Date:** March 26, 2026  
> **Target Platform:** Google Antigravity v1.20+  
> **Status:** Proposal

---

## Table of Contents

1. [Problem Statement](#1-problem-statement)
2. [Proposed Solution](#2-proposed-solution)
3. [Technical Implementation](#3-technical-implementation)
4. [User Impact](#4-user-impact)

---

## 1. Problem Statement

### The Core Issue

Modern AI-assisted development tools suffer from a fundamental limitation: **they forget everything between context windows.** Despite advances in context window sizes (100K–1M+ tokens), practical quality degrades significantly past ~100K tokens, and all accumulated project knowledge — including fixed bugs, architectural decisions, and proven patterns — is lost the moment a developer starts a new session.

### Specific Pain Points

#### 1.1 Repeated Errors Across Sessions

When developers begin a new context window, the AI agent has no memory of previously encountered and resolved errors. This leads to a frustrating and wasteful cycle:

- The agent uses a deprecated API syntax that was already corrected two sessions ago.
- A terminal command that previously failed (and was fixed) is re-attempted with the same incorrect flags.
- Library-specific gotchas that were debugged for 20+ minutes are re-introduced from scratch.

**Real-world cost:** Developers report spending 15–30% of their time in each new session re-explaining context and re-fixing known issues.

#### 1.2 No Prescriptive Memory

Antigravity's existing Knowledge Items (KI) system captures high-level *summaries* of past conversations (e.g., "this conversation was about building a REST API"). However, it does **not** capture prescriptive, actionable rules such as:

- *"Never use `require()` in this project — it uses ES Modules."*
- *"The Supabase client must be initialized with `@supabase/ssr`, not `@supabase/auth-helpers`."*
- *"Running `npm run build` before `prisma generate` will fail silently."*

These are the exact details that prevent repeated mistakes. Summaries alone are insufficient.

#### 1.3 Zero Transparency Into Context Consumption

Developers currently have no visibility into:

- How many tokens system prompts, agent policies, and tool definitions consume before the first user message.
- How much of the context window the current conversation has used.
- When they are approaching the quality-degradation threshold.
- What Knowledge Items, rules, or files the agent is actively reading.

This forces developers to rely on intuition — noticing output quality declining — rather than proactively managing their sessions.

#### 1.4 Cold Starts on New Projects

When a developer starts an entirely new project that shares characteristics with a past project (e.g., another Next.js + Supabase app), the agent starts from zero. Every best practice, boilerplate pattern, and pitfall avoidance strategy learned from the previous project must be re-discovered, re-debugged, and re-implemented.

---

## 2. Proposed Solution

### Overview: The Context Intelligence System (CIS)

A three-layer system that **automatically captures**, **persistently stores**, and **proactively applies** development knowledge across context windows and projects.

```
┌─────────────────────────────────────────────────────────┐
│                    LAYER 3: UI/UX                       │
│       Token Counter  ·  Context Inspector Panel         │
├─────────────────────────────────────────────────────────┤
│                  LAYER 2: INTELLIGENCE                  │
│   Project DNA Registry  ·  Error Journal Engine         │
├─────────────────────────────────────────────────────────┤
│                   LAYER 1: CAPTURE                      │
│  Auto Context Files  ·  Milestone Detection  ·  Hooks   │
└─────────────────────────────────────────────────────────┘
```

### 2.1 Layer 1 — Automatic Context Capture

**Auto-Generated Context Files:**  
After the first working version of an application (or any developer-defined milestone), the agent automatically generates and maintains structured context files:

- `context.txt` — Frontend state, tech stack, architecture, component tree, routing, and UI decisions.
- `backend_context.txt` — API structure, database schema, authentication flow, environment configuration, and deployment notes.

These files are **living documents** — automatically updated at key milestones:

| Trigger Event | Action |
|---|---|
| First working app compiles/runs | Generate initial `context.txt` |
| Backend API returns first successful response | Generate initial `backend_context.txt` |
| Error is encountered and fixed | Append to Error Journal (see 2.2) |
| Major feature is completed | Update context files with new state |
| User explicitly requests update | Full refresh of all context files |
| User is about to start a new context window | Final state snapshot + pending work summary |

### 2.2 Layer 2 — Intelligence Engine

**Error Journal (`error_journal.md`):**  
Every error-fix cycle is automatically logged with structured metadata:

```
### ERR-0042 | 2026-03-26

- **Error:** `TypeError: fetch is not a function`
- **Root Cause:** Node.js v16 does not include native `fetch`. Project requires v18+.
- **Fix Applied:** Updated `engines` field in `package.json` and switched to Node v18.
- **Prevention Rule:** Always verify Node.js version ≥ 18 before using native `fetch`.
  If targeting v16, use `node-fetch` package instead.
```

At the start of every new context window, all prevention rules are loaded as hard constraints that the agent must follow before writing any code.

**Project DNA Registry (`~/.gemini/project_dna.md`):**  
A global, cross-project knowledge base organized by project archetype:

```
## Archetype: Next.js + Tailwind + Supabase

### Boilerplate Decisions
- Use App Router (not Pages Router)
- Use Server Components by default; add "use client" only when needed
- Initialize Supabase with @supabase/ssr for cookie-based auth

### Known Pitfalls
- middleware.ts must re-export `config` or auth redirects silently fail
- Tailwind v4 uses CSS-based config, not tailwind.config.js
- Supabase RLS policies must be enabled before any client-side queries

### Recommended Structure
/app        → Routes and layouts
/components → Reusable UI components
/lib        → Supabase client, utilities
/types      → TypeScript type definitions
```

When starting a new project, the agent:
1. Reads the Project DNA Registry.
2. Identifies the closest archetype based on the user's description.
3. Applies all accumulated best practices from day one.
4. If no matching archetype exists, creates a new entry after project completion.

### 2.3 Layer 3 — UI/UX Transparency

**Token Counter Widget:**  
A persistent, non-intrusive UI element in the chat interface:

```
┌──────────────────────────────────────┐
│  Tokens: 47,200 / 200,000           │
│  ██████████░░░░░░░░░░  23.6%        │
│  System: 12.1K · Context: 8.3K      │
│  Conversation: 26.8K                │
│  Quality Zone: 🟢 Optimal           │
└──────────────────────────────────────┘
```

Quality zone thresholds:
- 🟢 **Optimal** (< 50K) — Full capability, highest quality output
- 🟡 **Good** (50K–100K) — Minor quality softening possible
- 🟠 **Caution** (100K–150K) — Recommend starting a new window soon
- 🔴 **Degraded** (> 150K) — Prompt to save context and start fresh

**Context Inspector Panel:**  
An expandable sidebar or modal accessible via a dedicated button:

```
📋 Active Context
├── 📜 System Prompts .................. 11,842 tokens
├── 📏 Global Rules (~/.gemini/GEMINI.md) .. 340 tokens
├── 📏 Workspace Rules (.agent/rules/) .... 520 tokens
├── 🧠 Knowledge Items Loaded .............. 3
│   ├── KI: "API Architecture Patterns" ... 1,200 tokens
│   ├── KI: "Auth Flow Implementation" .... 890 tokens
│   └── KI: "Database Schema v2" .......... 650 tokens
├── 📎 Attached Files ...................... 2
│   ├── context.txt .................... 2,100 tokens
│   └── backend_context.txt ............ 1,800 tokens
├── 🚫 Error Prevention Rules Active ....... 12
└── 🧬 Project DNA Archetype: "Next.js + Supabase"
```

This gives developers full visibility into the AI's "working memory" at any moment.

---

## 3. Technical Implementation

### 3.1 Leveraging Existing Antigravity Architecture

The Context Intelligence System is designed to integrate with Antigravity's existing infrastructure rather than requiring a ground-up rebuild.

#### Agent Policies (Rules Engine)

**Current capability:** Antigravity supports global rules (`~/.gemini/GEMINI.md`) and workspace rules (`.agent/rules/*.md`) that are injected into the agent's system prompt at conversation start.

**CIS extension:**

| Component | Implementation |
|---|---|
| Error Journal ingestion | A new agent policy directive: `@auto-load-error-journal` that reads `.agent/error_journal.md` and converts each prevention rule into a system-level constraint. |
| Context file auto-read | A new agent policy directive: `@auto-load-context` that reads `context.txt` and `backend_context.txt` at session start without user prompting. |
| Project DNA matching | A new agent policy directive: `@match-project-dna` that reads `~/.gemini/project_dna.md`, selects the closest archetype, and applies its rules. |

#### Knowledge Subagent Enhancement

**Current capability:** A background Gemini 2.5 Flash model performs "checkpointing and context summarization" to produce Knowledge Items.

**CIS extension:**

The Knowledge Subagent's scope should be expanded to produce two additional output types:

1. **Prescriptive Rules** — Instead of only summarizing "what happened," extract actionable "do this / don't do that" rules from error-fix cycles.
2. **Milestone Detection** — Monitor conversation flow for milestone events (first successful build, new feature completion, major refactor) and trigger context file updates.

```
Current Pipeline:
  Conversation → Summarize → KI (descriptive)

Proposed Pipeline:
  Conversation → Summarize → KI (descriptive)
               → Extract   → Prevention Rules (prescriptive)
               → Detect    → Milestone Events → Context File Update
```

#### MCP Server Integration

**Current capability:** Antigravity supports custom MCP servers for connecting to databases, APIs, and external tools.

**CIS extension:**

A dedicated `context-intelligence` MCP server could provide:

| MCP Tool | Function |
|---|---|
| `read_error_journal` | Returns all prevention rules for the current project. |
| `append_error_journal` | Adds a new error-fix entry after an error is resolved. |
| `update_context_file` | Regenerates `context.txt` or `backend_context.txt` based on current project state. |
| `query_project_dna` | Searches the global Project DNA registry for matching archetypes. |
| `get_token_usage` | Returns current token consumption breakdown for UI rendering. |

**MCP Resource URIs:**

```
context://project/errors          → Error journal entries
context://project/state           → Current context file contents
context://global/project-dna      → Project DNA registry
context://session/token-usage     → Live token consumption data
```

#### Agent Skills Framework

**Current capability:** Antigravity supports "Skills" — folders of instructions and scripts that extend agent capabilities for specialized tasks.

**CIS extension:**

A `context-management` skill could be created:

```
.agent/skills/context-management/
├── SKILL.md                    # Skill instructions
├── scripts/
│   ├── generate_context.py     # Generates context.txt from project analysis
│   ├── update_error_journal.py # Parses conversation for error-fix cycles
│   └── match_archetype.py      # Matches project to DNA registry archetype
├── templates/
│   ├── context_template.md     # Template for context.txt
│   ├── backend_context_template.md
│   └── error_entry_template.md
└── resources/
    └── default_archetypes.md   # Starter set of common project archetypes
```

### 3.2 Token Counter — Implementation Path

Token counting data already exists internally within the Antigravity platform (it must, for billing and rate-limiting). Surfacing it requires:

1. **Backend:** Expose a lightweight WebSocket or polling endpoint that returns:
   ```json
   {
     "system_tokens": 11842,
     "context_tokens": 8340,
     "conversation_tokens": 26800,
     "total_tokens": 46982,
     "max_tokens": 200000,
     "quality_zone": "optimal"
   }
   ```

2. **Frontend:** A React/Svelte component in the chat UI that subscribes to this endpoint and renders the token bar. This component should be:
   - Non-intrusive (small bar below the input field or in the header)
   - Expandable on click (to show the full breakdown)
   - Color-coded (green → yellow → orange → red)

### 3.3 Context Inspector — Implementation Path

The Context Inspector aggregates data that already exists in different parts of the system:

| Data Source | Current Location | Access Method |
|---|---|---|
| System prompt size | Internal agent configuration | Expose via internal API |
| Loaded rules | `~/.gemini/GEMINI.md`, `.agent/rules/` | File system read |
| Active KIs | `<appDataDir>/knowledge/` | File system read + metadata.json |
| Attached files | Conversation state | Session state API |
| Error prevention rules | `.agent/error_journal.md` | File system read |
| Project DNA archetype | `~/.gemini/project_dna.md` | File system read |

The inspector panel would be a new UI pane (similar to the existing pane system for files and KIs) that renders this aggregated view.

---

## 4. User Impact

### 4.1 Quantitative Benefits

| Metric | Current (Without CIS) | Projected (With CIS) | Improvement |
|---|---|---|---|
| **Repeated errors per project** | 5–15 per project | 0–2 per project | **85–100% reduction** |
| **Tokens wasted on re-explaining context** | ~30,000–50,000 per new window | ~2,000–5,000 per new window | **90% token savings** |
| **Time to start a new context window** | 5–10 minutes (manual file update + prompt) | 0 minutes (automatic) | **100% time saved** |
| **New project cold-start quality** | Low (starts from zero) | High (inherits best practices) | **First-attempt success rate ↑** |
| **Sessions before optimal workflow** | 3–5 per project (learning curve) | 1 (inherits from DNA) | **60–80% fewer iterations** |

### 4.2 Qualitative Benefits

#### For Individual Developers

- **Elimination of "Groundhog Day" debugging.** Errors that were fixed once stay fixed forever — across sessions, across projects.
- **Reduced cognitive load.** Developers no longer need to mentally track what the AI knows and doesn't know, or maintain manual context files.
- **Proactive session management.** The token counter empowers developers to start new windows at the right time, before quality degrades, rather than after noticing degraded output.
- **Confidence in AI consistency.** Knowing the AI carries forward project knowledge builds trust and encourages deeper delegation of complex tasks.

#### For Teams

- **Shared Project DNA.** Team members can share `project_dna.md` and `error_journal.md` files via version control, creating a collective institutional memory that benefits every developer on the team.
- **Onboarding acceleration.** New team members inherit the accumulated best practices and error prevention rules from the entire team's history with the codebase.

#### For the Antigravity Platform

- **Reduced compute costs.** Fewer wasted tokens on repeated errors and re-explanations means lower inference costs per meaningful output.
- **Stronger user retention.** Developers who see their AI assistant getting smarter over time (rather than resetting to zero) are significantly more likely to remain on the platform.
- **Competitive differentiation.** No major AI IDE currently offers transparent token counting, automatic error journaling, or cross-project knowledge transfer. This would be a first-mover advantage.

### 4.3 User Workflow: Before and After

#### Before (Current State)

```
Session 1: Build app → Fix 8 errors → Manually write context.txt → End session
Session 2: Attach context.txt → Remind AI of 3 things it forgot → Fix 2 repeated errors → Update context.txt → End session
Session 3: Attach context.txt → Fix 1 repeated error → Finish feature → Update context.txt → End session
New Project: Start from zero → Re-encounter 5 of the same errors → Repeat cycle
```

#### After (With CIS)

```
Session 1: Build app → 8 errors auto-logged with fixes → Context auto-generated → End session
Session 2: AI auto-loads context + error rules → 0 repeated errors → Continue building → Context auto-updated → End session
Session 3: AI auto-loads context → Finish feature → Context auto-updated → End session
New Project: AI matches to "Type A" archetype → Applies all learned patterns → 0 known errors from day one
```

---

## Appendix: Immediate Workaround for Users

While this proposal awaits platform-level implementation, developers can achieve approximately 60% of the described value today using Antigravity's existing Agent Policies:

**Step 1:** Create `~/.gemini/GEMINI.md` with instructions to auto-read context files and error journals.

**Step 2:** Create `.agent/rules/context.md` in each project workspace with project-specific context management instructions.

**Step 3:** After the first working app, prompt the agent to generate `context.txt` and `backend_context.txt`.

**Step 4:** After fixing errors, prompt the agent to append entries to `.agent/error_journal.md`.

This manual approach validates the concept and demonstrates user demand for the full automated solution.

---

*This proposal is based on real developer workflows and pain points observed during active development with Google Antigravity v1.20+. The features described aim to transform the AI coding assistant from a stateless tool into an intelligent partner that learns and improves with every interaction.*
