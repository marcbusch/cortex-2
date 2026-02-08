# /atlas-install

> Copy Atlas skills to ~/.claude/skills/ for global availability

You are the Atlas installer. Your job is to copy the Atlas skill files from this repository to the user's global Claude Code skills directory so they're available in any project.

## Step 1: Discover Skills

Use `Glob` to find all Atlas skill directories in the current repository:
- Pattern: `.claude/skills/atlas-*/SKILL.md`

Exclude `atlas-install` — it only makes sense from the repo, not globally.

List what was found.

## Step 2: Check Destination

Check if `~/.claude/skills/` exists. On Windows, this is `C:\Users\<username>\.claude\skills\`.

Use `Glob` to check for existing Atlas skill directories there:
- Pattern: `atlas-*/SKILL.md` in the `~/.claude/skills/` directory

If existing skills are found, note which ones would be overwritten.

## Step 3: Show Plan

Present to the user:

> **Atlas Install Plan**
>
> **Source:** `<repo-path>/.claude/skills/`
> **Destination:** `~/.claude/skills/`
>
> **Skills to copy:**
> - `atlas-new-project/` (new / overwrite existing)
> - `atlas-continue-project/` (new / overwrite existing)
> - `atlas-interview/` (new / overwrite existing)
> - `atlas-questions/` (new / overwrite existing)
>
> <If overwriting:> **Note:** This will overwrite <N> existing Atlas skills.

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

## Step 6: Confirm Success

Verify the files were written by using `Glob` to check `~/.claude/skills/atlas-*/SKILL.md`.

Report:

> **Atlas skills installed successfully.**
>
> Installed <N> skills to `~/.claude/skills/`:
> - `/atlas-new-project` — Initialize a new knowledge project
> - `/atlas-continue-project` — Run an investigation session
> - `/atlas-interview` — Structured user interview
> - `/atlas-questions` — Generate expert questions
>
> These skills are now available globally in Claude Code. Start with `/atlas-new-project` in any directory.

## Important Rules

- **Always confirm before overwriting.** The user may have customized their installed skills.
- **Copy ALL atlas-* skill directories except atlas-install**, not just a hardcoded list — this future-proofs for new skills.
- **Do NOT copy atlas-install** — it only works from the repo and isn't useful globally.
- **Do NOT modify the source files.** This is a one-way copy from repo to global.
- **Do NOT copy non-atlas skill directories** from the repo's `.claude/skills/` directory.
