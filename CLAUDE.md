# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What Is Cortex 2?

Cortex is a domain expertise acquisition system for Claude Code. It has no application code — it is entirely a set of Claude Code skills (`.claude/skills/`) backed by an architecture specification (`architecture.md`). Knowledge projects are stored as markdown file trees under `.local/cortex/<project-name>/`.

## Repository Structure

```
architecture.md                          # Full system design specification
.claude/skills/
  cortex-new-project/SKILL.md            # /cortex-new-project — project initialization via interview
  cortex-continue-project/SKILL.md       # /cortex-continue-project — orchestrated investigation sessions
  cortex-interview/SKILL.md              # /cortex-interview — structured user interviews
  cortex-questions/SKILL.md              # /cortex-questions — generate expert questions from knowledge gaps
  cortex-install/SKILL.md                # /cortex-install — copy skills to ~/.claude/skills/ for global use
.local/cortex/                           # Knowledge projects (gitignored)
```

## Architecture Overview

The system operates through **skills** (slash commands) that orchestrate **gathering agents** to build a tree of **markdown knowledge files**.

### Skills as the Interface

Users interact exclusively through slash commands. The core loop is:
1. `/cortex-new-project` — interview user, scaffold `project.md` + area files
2. `/cortex-continue-project` — the main orchestration loop: load state → select methods → gather (parallel agents) → consolidate → user confirms → persist with backup
3. `/cortex-interview` — direct structured interview for tribal knowledge
4. `/cortex-questions` — generate targeted questions for external experts

### Gathering Methods

The orchestrator (`/cortex-continue-project`) selects methods based on domain signals detected in `project.md`:

| Method | Agent Type | Domain Signal |
|--------|-----------|---------------|
| Web Research | `web-research-agent` | All projects |
| Code Exploration | `Explore` agent | Code-linked projects (dbt, LookML, etc.) |
| Data Exploration | `data-explorer-agent` | Data warehouse projects (BigQuery) |
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
- Agent prompts include the exact output format template — keep these consistent with `architecture.md`
- Cross-reference maintenance (link checking/updating) runs after every persist step
- Domain extension proposals (conventions.md, glossary.md, timeline) are offered once per trigger, tracked in session log to avoid re-proposing
