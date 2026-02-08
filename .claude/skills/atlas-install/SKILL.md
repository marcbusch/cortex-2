# /atlas-install

> Copy Atlas skills and agents to ~/.claude/ for global availability

You are the Atlas installer. Your job is to copy the Atlas skill files and custom agent definitions from this repository to the user's global Claude Code directory so they're available in any project.

## Step 1: Discover Skills and Agents

Use `Glob` to find all Atlas assets in the current repository:
- Skills: `.claude/skills/atlas-*/SKILL.md`
- Agents: `.claude/agents/atlas-*.md`

Exclude `atlas-install` from skills — it only makes sense from the repo, not globally.

List what was found.

## Step 2: Check Destination

Check if `~/.claude/skills/` and `~/.claude/agents/` exist. On Windows, this is `C:\Users\<username>\.claude\`.

Use `Glob` to check for existing Atlas assets:
- Skills: `atlas-*/SKILL.md` in `~/.claude/skills/`
- Agents: `atlas-*.md` in `~/.claude/agents/`

If existing files are found, note which ones would be overwritten.

## Step 3: Show Plan

Present to the user:

> **Atlas Install Plan**
>
> **Source:** `<repo-path>/.claude/`
> **Destination:** `~/.claude/`
>
> **Skills to copy:**
> - `skills/atlas-new-project/` (new / overwrite existing)
> - `skills/atlas-continue-project/` (new / overwrite existing)
> - `skills/atlas-interview/` (new / overwrite existing)
> - `skills/atlas-questions/` (new / overwrite existing)
>
> **Agents to copy:**
> - `agents/atlas-web-research-agent.md` (new / overwrite existing)
> - `agents/atlas-data-explorer-agent.md` (new / overwrite existing)
>
> <If overwriting:> **Note:** This will overwrite <N> existing Atlas files.

## Step 4: Confirm

Use `AskUserQuestion`:

- "Proceed with installation?"
- Options: "Yes, install skills", "Cancel"

## Step 5: Copy Files

If confirmed:

1. For each skill directory found in step 1:
   - Create the matching directory under `~/.claude/skills/` if it doesn't exist (use `Bash` with `mkdir -p`)
   - Read the `SKILL.md` file from the repository using `Read`
   - Write it to `~/.claude/skills/<skill-name>/SKILL.md` using `Write`

2. For each agent file found in step 1:
   - Create `~/.claude/agents/` if it doesn't exist (use `Bash` with `mkdir -p`)
   - Read the agent `.md` file from the repository using `Read`
   - Write it to `~/.claude/agents/<agent-name>.md` using `Write`

## Step 6: Confirm Success

Verify the files were written by using `Glob` to check:
- `~/.claude/skills/atlas-*/SKILL.md`
- `~/.claude/agents/atlas-*.md`

Report:

> **Atlas installed successfully.**
>
> Installed <N> skills to `~/.claude/skills/`:
> - `/atlas-new-project` — Initialize a new knowledge project
> - `/atlas-continue-project` — Run an investigation session
> - `/atlas-interview` — Structured user interview
> - `/atlas-questions` — Generate expert questions
>
> Installed <N> agents to `~/.claude/agents/`:
> - `atlas-web-research-agent` — Web research with source citations
> - `atlas-data-explorer-agent` — BigQuery data exploration
>
> These are now available globally in Claude Code. Start with `/atlas-new-project` in any directory.

## Important Rules

- **Always confirm before overwriting.** The user may have customized their installed skills.
- **Copy ALL atlas-* skill directories except atlas-install**, not just a hardcoded list — this future-proofs for new skills.
- **Copy ALL atlas-* agent files** from `.claude/agents/` — this ensures agent definitions travel with the skills.
- **Do NOT copy atlas-install** — it only works from the repo and isn't useful globally.
- **Do NOT modify the source files.** This is a one-way copy from repo to global.
- **Do NOT copy non-atlas files** from the repo's `.claude/skills/` or `.claude/agents/` directories.
