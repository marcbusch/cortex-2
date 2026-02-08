# Atlas — Architecture Specification

> v1.0 — 2026-02-07

## Overview

Atlas is a domain expertise acquisition system for Claude Code. It builds structured knowledge through iterative sessions, persisting findings as a tree of markdown files that serve simultaneously as human-readable documents, shareable briefings, and AI agent context.

### Core Principles

1. **The file format is the product.** Every markdown file is simultaneously the data store, the human-readable output, a shareable document, and valid AI agent context. No transformation step, no export.
2. **Write for understanding, not for parsing.** Capture intent and business rules, not code structure. "Orders are enriched with customer attributes for segmentation" survives a refactor; "line 47 joins on customer_id" does not.
3. **Iterative refinement.** Knowledge builds over multiple sessions. The system remembers what it knows, what it doesn't, and what to investigate next.
4. **Sparse human interaction.** The user steers direction; Atlas does the heavy lifting autonomously.
5. **Safe persistence.** Backups before any modification. User confirms before changes are written.

---

## Information Gathering Methods

Atlas gathers knowledge through five methods. Not all apply to every project — the orchestrator selects methods based on domain and session focus.

| Method | Autonomous | Tools | Best For |
|--------|-----------|-------|----------|
| **Web Research** | Yes | WebSearch, WebFetch | Public knowledge, regulations, best practices, API docs, community patterns |
| **Code Review** | Yes | Read, Glob, Grep, Explore agent | Business logic embedded in code, conventions, patterns |
| **Data Exploration** | Yes | BigQuery MCP | Understanding data, validating business rules, schema discovery |
| **User Interview** | No | AskUserQuestion | Tribal knowledge, preferences, constraints, decisions, context |
| **Expert Questions** | No | Output for async use | Generating targeted questions for external stakeholders |

### Method Selection Examples

- Personal migration project → Web Research + User Interview + Expert Questions
- dbt warehouse → Code Review + Data Exploration + Web Research + User Interview
- LookML analytics → Code Review + Data Exploration + User Interview
- Exam preparation → Web Research + User Interview
- Holiday planning → Web Research + User Interview

---

## Project File Structure

Each Atlas project is a tree of markdown files stored in `.local/atlas/<project-name>/`.

### Minimal Structure (New Project)

```
.local/atlas/<project-name>/
  project.md          # Root: brief, scope, area index, session log
  <area-1>.md         # Knowledge area file
  <area-2>.md         # Knowledge area file
  <area-3>.md         # Knowledge area file
  _backups/           # Timestamped snapshots before updates
```

### Grown Structure (After Multiple Sessions)

```
.local/atlas/<project-name>/
  project.md
  visa-and-residence.md
  healthcare-transition.md
  financial-assets/
    _overview.md              # Area overview + index of sub-topics
    exit-tax.md               # Sub-topic deep dive
    broker-transfer.md
    japan-taxation.md
  _backups/
    2026-02-01_1430/          # Full snapshot
    2026-02-03_1530/
```

### Splitting Rule

When an area file exceeds ~200 lines or covers 3+ distinct sub-topics, it splits:
1. The file becomes a directory with the same name
2. An `_overview.md` replaces the original file (summary + links to sub-topics)
3. Sub-topics become individual files within the directory
4. `project.md` area index is updated
5. Cross-references from other files are rewritten

The reverse (merging thin files) is also supported.

---

## File Specifications

### `project.md` — Root File

Stays small regardless of project size. Contains only top-level orientation.

```markdown
# [Project Name]

> Created: [date] | Last session: [date] | Areas: [N]

## Brief

[2-3 sentences: who, what, why. The elevator pitch for this project.]

## Scope

**In scope:**
- [bullet list]

**Out of scope:**
- [bullet list]

## Personal Context

[Only for projects where user-specific circumstances shape the knowledge.
Constraints, preferences, situation — gathered during initial interview.]

- **Key constraint**: [e.g., "FIRE lifestyle — no employment income"]
- **Timeline**: [e.g., "2-3 years until emigration"]

## Areas

- [Area Name](./area-file.md) — one-line summary. *Status: well-understood | early investigation | not started*
- [Area Name](./area-file.md) — one-line summary. *Status: ...*
- [Area Name](./area-file/) — one-line summary. *Status: ...*

## Open Questions (Top Priority)

[Only the 3-5 highest priority open questions across all areas, with links to where they live.]

1. [Question text] → [area-file.md]
2. [Question text] → [area-file.md]

## Session Log

| Date | Summary |
|------|---------|
| 2026-01-31 | Project initialized. 3 areas defined. |
| 2026-02-03 | Investigated financial assets. 15 findings added. Exit tax rules documented. |

## Next Session Focus

[What to tackle next. Updated at end of each session.]
```

### Area File — Standard Template

Each area file follows this structure. All sections are optional except Summary.

```markdown
# [Area Name]

> Last updated: [date] | Status: [well-understood / early investigation / not started]

## Summary

[2-3 paragraphs. Executive summary of current understanding — what is known,
what is uncertain, what actions are needed. A reader should get 80% of the
value from this section alone.]

## [Sub-topic Heading]

[Knowledge as concise prose, organized by sub-topic. Each paragraph or bullet
makes a claim with inline confidence and source markers.]

[Claim text.] [high confidence · s:source-name]

[Another claim.] [medium confidence · s:source-name, s:other-source]

> **Hypothesis:** [Unverified claim that needs checking. Include verification
> approach.] [low confidence]

## [Another Sub-topic Heading]

[More knowledge...]

## Decisions

### [Decision Title]
- **Chosen:** [what was chosen]
- **Alternatives considered:** [what was rejected and why, briefly]
- **Rationale:** [why]
- **Date:** [when decided]
- **Revisit if:** [conditions that would reopen this decision]

## Open Questions

- **[Question text]** (priority: high/medium/low) — [context, what's been tried]
- **[Question text]** (priority: high/medium/low) — [context]

## Sources

- **[s:source-id]** [type: url/code/data/user/document] — [title or description]. [location/URL]. Reliability: [high/medium/low].
```

### Inline Conventions

Instead of YAML fields, use lightweight inline markers at the end of paragraphs or bullets:

| Concept | Convention | Example |
|---------|-----------|---------|
| Confidence | `[high confidence]`, `[medium confidence]`, `[low confidence]` | The COE is free. [high confidence · s:immigration-bureau] |
| Sources | `· s:source-id` after confidence | ...applies to holdings above EUR 500k per fund. [high confidence · s:bmf-2025, s:tax-advisor-blog] |
| Hypothesis | Blockquote with `**Hypothesis:**` prefix | > **Hypothesis:** The 5-year window may reset if... [medium confidence] |
| Open question | Bold text in `## Open Questions` section | **What format is required for brokerage statements?** |
| Cross-reference | Standard markdown link | See [Japan Taxation](./financial-assets/japan-taxation.md) |
| Code/artifact ref | Backtick inline reference | The logic lives in `fct_orders.sql` |

### No Fact IDs

Facts do not have IDs. A fact is a paragraph in a section of a specific file. To reference a fact, reference the file + section heading. This eliminates ID management, renumbering, and the overhead of maintaining a parallel identifier system.

Hypotheses and open questions are called out explicitly because they represent uncertainty — the most actionable part of any knowledge base.

---

## Domain-Specific Extensions

The core system works for any domain. These optional extensions activate when the project's domain warrants them.

### For Code-Linked Projects (dbt, LookML, Application Code)

**Conventions file.** Projects with "how we do things here" patterns get a top-level `conventions.md`:

```markdown
# Conventions

## Naming
- Staging models: `stg_{source}__{entity}`
- Mart models: `fct_{entity}` or `dim_{entity}`
...

## Patterns
- [Reusable patterns, standard approaches]

## Anti-Patterns
- [Things to avoid and why]
```

**Glossary.** Projects where business terms map to code artifacts get a `glossary.md`:

```markdown
# Glossary

### Revenue (Net)
**Definition:** Gross order value minus discounts, returns, and taxes.
**Implemented in:** `fct_orders.net_revenue` (mart layer)
**Not to be confused with:** Gross Revenue, Recognized Revenue
```

**Drift awareness.** When resuming a session, the orchestrator should note if referenced code may have changed since the knowledge was last verified. Area files include a `Last updated` timestamp for this purpose. The system does not duplicate code-level details — it captures the business-level understanding that Claude Code can validate against the actual code at query time.

### For Planning/Personal Projects

**Timeline section.** Projects with temporal dependencies get a `## Timeline` in `project.md` or relevant area files:

```markdown
## Timeline

Milestone: [Key Event] (target: [date or range])

| When | Action | Status | Depends On | Area |
|------|--------|--------|------------|------|
| T-12m | Start COE application | TODO | Document prep | [visa](./visa.md) |
| T-9m | Apply for visa | TODO | COE received | [visa](./visa.md) |
```

Use relative timelines (T-minus from key event) rather than absolute dates — personal project timelines shift.

**Decision tracking** is particularly important for personal projects where the user makes choices between alternatives. The `## Decisions` section is part of the standard template but becomes a primary section for planning projects.

---

## Orchestration Flow

The `/atlas-continue-project` skill orchestrates investigation sessions.

```
USER: /atlas-continue-project

  1. LOAD STATE
     Read project.md + area files relevant to planned focus.
     (Not all files — only what's needed for this session.)

  2. ORIENT USER
     Summarize current state: areas, progress, open questions.
     Show what was planned for this session.

  3. CONFIRM FOCUS
     Ask user: Is this still the right focus?
     Allow adjustment of priorities or scope.

  4. DETERMINE METHODS
     Based on domain and focus, decide which gathering methods to use.
     Prepare focused instructions for each agent/method.

  5. GATHER (parallel where possible)
     ┌──────────────┐  ┌──────────────┐  ┌──────────────┐
     │ Web Research  │  │    Code      │  │    Data      │
     │    Agent      │  │   Explore    │  │  Explorer    │
     └──────────────┘  └──────────────┘  └──────────────┘

  6. CONSOLIDATE
     Gather agent outputs.
     Check for conflicts with existing knowledge.
     Assess confidence levels.
     Identify new questions raised.

  7. CHECK IN WITH USER
     Present findings.
     Highlight conflicts or uncertainties.
     Ask: Sufficient depth? Continue investigating?
     If more needed → return to step 4.

  8. PREPARE UPDATES
     Draft the specific text changes to area files.
     Show user exactly what will be added/modified/removed.
     Propose new open questions, mark answered ones.

  9. USER CONFIRMS
     User reviews proposed changes.
     Can adjust, reject, or approve.

 10. PERSIST
     Create timestamped backup of all modified files.
     Write changes to area files.
     Update project.md (session log, area status, next focus).
     Handle file splits if any area file has grown too large.
```

### Key Behaviors

- The orchestrator loads only the files relevant to the session focus, not the entire tree. This keeps context lean.
- Agents receive focused instructions: the question to investigate, relevant existing knowledge from the area file, and any constraints.
- The orchestrator is an **editor** — it integrates findings as prose into the right sections of the right area files, weaving new knowledge into existing narrative.
- File splits are proposed during step 10 if an area file has grown past the threshold.

---

## Skills

| Skill | Purpose | Invoked By |
|-------|---------|------------|
| `/atlas-new-project` | Initialize project via interview | User |
| `/atlas-continue-project` | Full investigation session: load → investigate → confirm → persist | User |
| `/atlas-interview` | Structured interview to gather user knowledge | User or orchestrator |
| `/atlas-questions` | Generate questions for external experts/stakeholders | User |

### `/atlas-new-project`

Interviews the user to establish:
1. Domain/topic and motivation
2. 2-3 initial areas to explore
3. What's out of scope
4. Personal context and constraints (if relevant)
5. Burning questions to start with
6. Known sources

Creates the project directory with `project.md` and initial area files (mostly empty, with open questions seeded from the interview).

### `/atlas-continue-project`

The core orchestration loop (see Orchestration Flow above). Handles the full cycle from loading state through investigation to persisted updates.

### `/atlas-interview`

Structured gathering of knowledge from the user. Two modes:
1. **Direct interview** — systematic questions using AskUserQuestion, focused on specific areas or open questions
2. **Expert prep** — redirects to `/atlas-questions` for generating questions to send to others

Output: proposed text additions to area files, presented for confirmation.

### `/atlas-questions`

Generates targeted questions for external experts based on open questions and knowledge gaps. Output is a markdown document suitable for copy/paste — organized by topic, with context for each question.

---

## Agents

Custom agents are defined in `.claude/agents/` with tool restrictions and system instructions. The orchestrator spawns them via the `Task` tool using their `subagent_type` name.

| Agent | Definition | Purpose | Tools |
|-------|-----------|---------|-------|
| `atlas-web-research-agent` | `.claude/agents/atlas-web-research-agent.md` | Search and synthesize web sources with citations | WebSearch, WebFetch, Read |
| `atlas-data-explorer-agent` | `.claude/agents/atlas-data-explorer-agent.md` | Query and analyze data via BigQuery MCP | BigQuery MCP tools, Read |
| `Explore` (built-in) | Claude Code built-in | Extract domain knowledge from codebases | Glob, Grep, Read |

### Agent Output Format

Agents return findings as prose with inline markers, structured for direct integration into area files:

```markdown
## Findings: [focus topic]

Agent: web-research | Date: 2026-02-07

### [Sub-topic]

[Finding as prose paragraph.] [high confidence · s:source-name]

[Another finding.] [medium confidence · s:source-name]

> **Hypothesis:** [Unverified claim.] [low confidence]

### New Questions Raised

- **[Question text]** (priority) — [context]

### Sources Used

- **[s:source-id]** [type] — [title]. [URL/location]. Reliability: [high/medium/low]. Accessed: [date].

### Notes

[Observations, caveats, suggestions for follow-up.]
```

The orchestrator takes this output and integrates it into the appropriate area file(s) — it does not copy the agent output verbatim.

---

## Backup and Safety

### Backup Invariant

Before any file modification during step 10 (persist), the system creates a timestamped backup of all files that will be changed:

```
_backups/
  2026-02-07_1430/
    project.md
    financial-assets.md
    visa-and-residence.md
```

Only modified files are backed up (not the entire tree).

### No Writes Without Confirmation

The system never writes to project files without showing the user exactly what will change and receiving explicit approval. The only exception is backup creation, which is always safe.

---

## How Output Gets Used

The markdown files are directly usable in multiple contexts without transformation:

1. **AI agent context.** Drop `project.md` + relevant area files into any agent's prompt. The agent now has structured domain expertise. Example: a coding agent building a new dbt model receives `conventions.md` + the relevant business logic area file.

2. **Decision support.** Point an agent at specific area files and ask domain-specific questions grounded in researched knowledge with confidence levels.

3. **Handover to professionals.** Share area files with lawyers, tax advisors, analysts, or other experts as-is. The format is professional and self-contained.

4. **Onboarding.** Read the tree top-down from `project.md` as a structured learning path for anyone new to the domain.

5. **RAG source material.** Each area file is a well-scoped chunk — natural chunking for retrieval-augmented generation.

6. **Living documentation.** For technical domains, the Atlas output evolves as the system evolves — updated through investigation sessions rather than manual writing.

---

## Design Decisions

| Decision | Resolution | Rationale |
|----------|-----------|-----------|
| Markdown over YAML | All knowledge as prose with inline conventions | AI reads prose as well as YAML; humans read it far better. One format serves both. |
| No fact IDs | Reference by file + section heading | Eliminates ID management overhead. Paragraphs are the atomic unit. |
| File-per-area over monolith | Each area is a self-contained document | Scales naturally. Only load what's needed. Each file is independently shareable. |
| Organic splitting | Split at ~200 lines or 3+ sub-topics | No pre-planned tree. Structure emerges from content. |
| Decisions as first-class sections | Optional `## Decisions` in area files | Knowledge without decisions is trivia. Decisions are what users act on. |
| No code duplication | Capture business intent, not code structure | Claude Code sees the code directly. Knowledge files add the "why" layer. |
| Domain extensions over modes | Optional sections activated by domain, not config | Keeps the core simple. Extensions are just section templates. |
| Selective file loading | Orchestrator loads only session-relevant files | Preserves context window for investigation work. |

---

## Implementation Roadmap

### Phase 1: Foundation
- [x] `/atlas-new-project` skill — interview and project scaffolding
- [x] `/atlas-continue-project` skill — orchestration loop with web research
- [x] Backup-before-write mechanism
- [x] Area file template and inline conventions

### Phase 2: Full Gathering
- [x] `atlas-data-explorer-agent` (custom agent definition)
- [x] Code exploration via built-in `Explore` agent
- [x] `/atlas-interview` skill
- [x] `/atlas-questions` skill

### Phase 3: File Management
- [x] Automatic split detection and proposal
- [x] Split execution (directory creation, link rewriting)
- [x] Merge support for thin files

### Phase 4: Polish
- [x] Domain-specific extension templates (conventions, glossary, timeline)
- [x] Session-start drift awareness for code-linked projects
- [x] Improved cross-referencing and navigation

---

*This specification is the design foundation for Atlas. Update as implementation reveals new insights.*
