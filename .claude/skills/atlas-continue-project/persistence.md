# Persistence Procedure

This file is referenced by the atlas-continue-project and atlas-interview skills. Execute these steps in order after the user confirms changes.

## Step A: Backup

1. Get the current date/time formatted as `YYYY-MM-DD_HHMM`
2. Create directory: `.local/atlas/<project-name>/_backups/<timestamp>/`
3. Copy each file that will be modified into the backup directory using `Read` then `Write` (read the current content, write it to the backup location)

## Step B: Write Area Files

For each area file being updated:
1. Read the current file content
2. **Rewrite the file with updates integrated** — you are an editor, not an appender. Weave new findings into existing narrative:
   - Update the `## Summary` to reflect the new state of knowledge
   - Add or update sub-topic sections with new findings, using inline confidence and source markers
   - Add new entries to `## Open Questions`
   - Remove questions that have been answered (the answers are now in the body)
   - Add new entries to `## Sources`
   - Update the `> Last updated:` date and status
3. Write the updated file

## Step C: Update project.md

1. Read current project.md
2. Update:
   - `Last session:` date in the header
   - Area status in `## Areas` section
   - `## Open Questions (Top Priority)` — refresh with current top priorities
   - `## Session Log` — add new entry row
   - `## Next Session Focus` — write recommendation for next session based on what was learned and what gaps remain
3. Write updated project.md

## Step D: Cross-Reference Maintenance

After writing all files (area files + project.md), scan for broken or stale cross-references:

1. **Collect all internal links** across modified files. Internal links look like `[text](./<path>)` or `[text](./<path>#<section>)`.

2. **Check each link target:**
   - Does the target file exist? (Use `Glob`)
   - If the link includes a `#section` anchor, does that heading exist in the target file? (Use `Grep` for the heading text)

3. **If broken links are found**, fix them silently:
   - File was renamed/moved → update the link path (search for a file with similar name)
   - Section heading was reworded → update the anchor to match the new heading
   - File was deleted → remove the link and replace with plain text + a note: `(link removed — <file> no longer exists)`

4. **If section headings in the modified files changed**, proactively scan other project files for links pointing to the old heading:
   - Use `Grep` to search all `.md` files in the project for `#<old-heading-slug>`
   - Update any found references to `#<new-heading-slug>`

This step should be quick — only check files that were modified in this session and their inbound references. Do not scan the entire tree unless a rename occurred.
