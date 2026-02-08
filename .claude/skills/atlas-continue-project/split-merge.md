# Split & Merge Procedures

This file is referenced by atlas-continue-project after persistence. Check each modified area file for split/merge triggers.

## Split Detection + Execution

After writing area files, check each one for split triggers.

### Split Triggers

Either one is sufficient:
1. **Line count:** file exceeds ~200 lines
2. **Sub-topic count:** file has 3 or more `## ` level headings that are substantive sub-topics (ignore structural sections: Summary, Decisions, Open Questions, Sources)

### Proposing a Split

If a split is triggered, propose it to the user:

> **Split proposed: <area-file>.md**
>
> This file has grown to <N> lines with <N> sub-topics. Splitting would improve navigability.
>
> **Proposed structure:**
> ```
> <area-name>/
>   _overview.md     — Summary + links to sub-topics
>   <sub-topic-1>.md — <title>
>   <sub-topic-2>.md — <title>
>   <sub-topic-3>.md — <title>
> ```
>
> Split now?

Use `AskUserQuestion`:
- Options: "Yes, split now", "Not now — keep as single file"

### Executing a Split

If approved:

1. **Create the directory:** `.local/atlas/<project-name>/<area-name>/`

2. **Create `_overview.md`** with:
   ```markdown
   # <Area Name>

   > Last updated: <today> | Status: <carried from original>

   ## Summary

   <Summary section from the original file — kept in full>

   ## Sub-topics

   - [<Sub-topic 1>](./<sub-topic-1>.md) — <one-line summary>
   - [<Sub-topic 2>](./<sub-topic-2>.md) — <one-line summary>
   - [<Sub-topic 3>](./<sub-topic-3>.md) — <one-line summary>

   ## Decisions

   <Decisions section from the original file, if it exists — decisions stay in the overview>

   ## Open Questions

   <Open questions that are general to the area, not specific to a sub-topic.
   Sub-topic-specific questions move to their respective files.>

   ## Sources

   <Sources that are referenced in the overview. Sub-topic-specific sources move to their files.>
   ```

3. **Create sub-topic files** — one per substantive `## ` section from the original:
   ```markdown
   # <Sub-topic Name>

   > Last updated: <today> | Status: <infer from content depth>
   > Parent: [<Area Name>](./_overview.md)

   <All content from the original section, preserved as-is>

   ## Open Questions

   <Questions specific to this sub-topic, moved from the original>

   ## Sources

   <Sources referenced in this sub-topic's content, moved from the original>
   ```

4. **Rewrite cross-references** in all other project files:
   - Scan all `.md` files in the project directory for links to the original file
   - Replace `[text](./<area-name>.md)` → `[text](./<area-name>/)`
   - Replace `[text](./<area-name>.md#section)` → `[text](./<area-name>/<sub-topic>.md)` where the section maps to a sub-topic file
   - Use `Grep` to find all references, then `Edit` to update them

5. **Update `project.md`** area index:
   - Change the area link from `[Area](./<area-name>.md)` to `[Area](./<area-name>/)`

6. **Delete the original flat file** — it's been replaced by the directory. (Back it up first in the backup step if not already backed up.)

7. **Report the result:**
   > Split complete: `<area-name>.md` → `<area-name>/` with <N> sub-topic files.

---

## Merge Detection + Execution

After writing, check for area directories where all sub-topic files are thin.

### Merge Trigger

An area directory where ALL sub-topic files (excluding `_overview.md`) are under ~30 lines each AND the total line count of all files combined is under ~150 lines.

This can happen when sub-topics were split prematurely or when content has been removed/consolidated.

### Proposing a Merge

If a merge is triggered, propose it:

> **Merge proposed: <area-name>/**
>
> All sub-topic files in this directory are thin (<30 lines each, <N> total lines combined). Merging back into a single file would be simpler.
>
> Merge now?

Use `AskUserQuestion`:
- Options: "Yes, merge back", "No — keep as separate files"

### Executing a Merge

If approved:

1. **Read all files** in the directory (`_overview.md` + all sub-topic files)

2. **Compose the merged file** `<area-name>.md`:
   - Start with the Summary from `_overview.md`
   - Add each sub-topic as a `## ` section, in the order they appear in the overview's index
   - Consolidate Open Questions and Sources sections (deduplicate)
   - Carry over Decisions from the overview
   - Set `> Last updated: <today>`

3. **Write the merged file** to `.local/atlas/<project-name>/<area-name>.md`

4. **Rewrite cross-references** in all other project files:
   - Replace `[text](./<area-name>/)` → `[text](./<area-name>.md)`
   - Replace `[text](./<area-name>/_overview.md)` → `[text](./<area-name>.md)`
   - Replace `[text](./<area-name>/<sub-topic>.md)` → `[text](./<area-name>.md#<sub-topic-slug>)`

5. **Update `project.md`** area index link

6. **Delete the directory** and all files within it (already backed up in the backup step)

7. **Report the result:**
   > Merged: `<area-name>/` (<N> files) → `<area-name>.md`
