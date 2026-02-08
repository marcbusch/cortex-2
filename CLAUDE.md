# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What Is Atlas?

Atlas is a domain expertise acquisition system for Claude Code. It has no application code — it is entirely a set of Claude Code skills (`.claude/skills/`) backed by an architecture specification (`architecture.md`). Knowledge projects are stored as markdown file trees under `.local/atlas/<project-name>/`.

## Repository Structure

```
architecture.md                          # Full system design specification
.claude/agents/
  atlas-web-research-agent.md           # Custom agent: web research with source citations
  atlas-data-explorer-agent.md          # Custom agent: BigQuery data exploration
  atlas-code-explorer-agent.md          # Custom agent: codebase domain knowledge extraction
.claude/skills/
  atlas-new-project/SKILL.md            # /atlas-new-project — project initialization via interview
  atlas-continue-project/SKILL.md       # /atlas-continue-project — orchestrated investigation sessions
  atlas-continue-project/persistence.md # Reference: backup, write, cross-reference procedures
  atlas-continue-project/split-merge.md # Reference: file split/merge detection and execution
  atlas-continue-project/domain-extensions.md  # Reference: conventions, glossary, timeline proposals
  atlas-interview/SKILL.md              # /atlas-interview — structured user interviews
  atlas-questions/SKILL.md              # /atlas-questions — generate expert questions from knowledge gaps
  atlas-install/SKILL.md                # /atlas-install — copy skills to ~/.claude/skills/ for global use
.local/atlas/                           # Knowledge projects (gitignored)
```

## Architecture Overview

The system operates through **skills** (slash commands) that orchestrate **gathering agents** to build a tree of **markdown knowledge files**.

### Skills as the Interface

Users interact exclusively through slash commands. The core loop is:
1. `/atlas-new-project` — interview user, scaffold `project.md` + area files
2. `/atlas-continue-project` — the main orchestration loop: load state → select methods → gather (parallel agents) → consolidate → user confirms → persist with backup
3. `/atlas-interview` — direct structured interview for tribal knowledge
4. `/atlas-questions` — generate targeted questions for external experts

### Gathering Methods

The orchestrator (`/atlas-continue-project`) selects methods based on domain signals detected in `project.md`:

| Method | Agent Type | Domain Signal |
|--------|-----------|---------------|
| Web Research | `atlas-web-research-agent` | All projects |
| Code Exploration | `atlas-code-explorer-agent` | Code-linked projects (dbt, LookML, etc.) |
| Data Exploration | `atlas-data-explorer-agent` | Data warehouse projects (BigQuery) |
| User Interview | Direct (`AskUserQuestion`) | Personal/planning projects |

Agents are launched in **parallel foreground mode** (multiple `Task` calls in a single message, never `run_in_background`).

### Knowledge File Format

Every file is simultaneously the data store, human-readable document, shareable briefing, and AI agent context. Key conventions:
- Confidence markers: `[high confidence · s:source-name]` inline after claims
- Hypotheses: `> **Hypothesis:** ...` blockquotes for unverified claims
- Sources: `[s:source-id]` references with a `## Sources` section per file
- No fact IDs — reference by file + section heading
- Files split at ~200 lines or 3+ substantive sub-topics into directories with `_overview.md`

### Persistence Safety

All file writes follow: backup first → show user exactly what changes → user confirms → write. Backups go to `_backups/<YYYY-MM-DD_HHMM>/`. The only exception is backup creation itself, which is always safe.

## Key Design Decisions

- **Markdown over YAML** — prose with inline conventions serves both humans and AI
- **File-per-area** — each area is self-contained, independently shareable, selectively loadable
- **Edit, don't append** — the orchestrator integrates findings as prose into existing narrative, not raw agent output
- **Business intent over code structure** — capture "orders are enriched for segmentation" not "line 47 joins on customer_id"
- **Selective loading** — only read area files relevant to current session focus

## Modifying Skills

When editing skill files, follow these constraints:
- Skills must use `AskUserQuestion` for structured choices and never write without user confirmation
- Agent output format templates live in the agent definition files (`.claude/agents/`), not inline in skills — keep these consistent with `architecture.md`
- The continue-project skill uses progressive disclosure: core orchestration in SKILL.md, post-persist procedures in reference files (persistence.md, split-merge.md, domain-extensions.md)
- Cross-reference maintenance (link checking/updating) runs after every persist step
- Domain extension proposals (conventions.md, glossary.md, timeline) are offered once per trigger, tracked in session log to avoid re-proposing
