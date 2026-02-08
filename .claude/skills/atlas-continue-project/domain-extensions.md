# Domain Extension Proposals

This file is referenced by atlas-continue-project after all file operations are complete. Check whether the project's growing knowledge warrants domain-specific extension files. These are proposed once — when conditions are first met — not every session.

## Conventions File (`conventions.md`)

### When to Propose

- The project has code-linked domain signals AND
- Area files contain 3+ observations about naming patterns, coding standards, or reusable approaches AND
- No `conventions.md` exists yet

### Proposal

> **Suggestion:** Your project has accumulated several coding conventions and patterns. Would you like me to create a `conventions.md` to consolidate them?
>
> This would extract convention-related findings from area files into a dedicated reference document covering naming, patterns, and anti-patterns.

### Template (if accepted)

```markdown
# Conventions

> Last updated: <today> | Extracted from: <list of source area files>

## Naming

<Extract naming conventions from area files. Each convention as a bullet with example.>

- <Pattern>: `<example>` [high confidence · s:<source>]

## Patterns

<Extract reusable patterns and standard approaches.>

### <Pattern Name>

<Description of when and how to use this pattern.>

## Anti-Patterns

<Extract things to avoid and why.>

- **<Anti-pattern>:** <why to avoid> [<confidence> · s:<source>]
```

Add it to `project.md`'s area index. Do NOT remove the convention observations from the original area files — `conventions.md` is a consolidated view, not a replacement.

---

## Glossary File (`glossary.md`)

### When to Propose

- Area files define 5+ domain-specific terms, business concepts, or entity names AND
- No `glossary.md` exists yet

### Proposal

> **Suggestion:** Your project uses several domain-specific terms. Would you like me to create a `glossary.md` to define them in one place?

### Template (if accepted)

```markdown
# Glossary

> Last updated: <today>

### <Term>
**Definition:** <clear definition>
**Implemented in:** <code reference, if applicable>
**Not to be confused with:** <similar terms, if applicable>
**Source:** [<confidence> · s:<source>]

### <Term>
...
```

---

## Timeline Section (in `project.md`)

### When to Propose

- The project has personal/planning domain signals AND
- Area files mention dates, deadlines, sequences, or dependencies between actions AND
- No `## Timeline` section exists yet in `project.md`

### Proposal

> **Suggestion:** Your project involves time-dependent actions. Would you like me to add a Timeline section to track milestones and dependencies?

### Template (if accepted)

Add a `## Timeline` section to `project.md` (before `## Session Log`):

```markdown
## Timeline

Milestone: <Key Event> (target: <date or range>)

| When | Action | Status | Depends On | Area |
|------|--------|--------|------------|------|
| <relative time> | <action> | TODO | <dependency> | [<area>](./<area>.md) |
```

Use relative timelines (T-minus from key milestone) rather than absolute dates — personal timelines shift.

---

## Proposal Rules

- Only propose one extension per session. If multiple are warranted, propose the most impactful one and note the others for next session.
- Use `AskUserQuestion` for the proposal — don't just create files.
- If the user declines, note it in the session log so you don't re-propose next session: `| <date> | ... Conventions file declined. |`
- Extension files follow the same backup rules as all other files.
