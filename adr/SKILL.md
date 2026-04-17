---
name: adr
description: Create architectural decision records in MADR format. Use when the user wants to document a technical decision, record an architecture choice, or create an ADR. Also invoke proactively after completing a significant architectural or design decision, even if the user hasn't explicitly asked for an ADR. For non-obvious project conventions (naming, patterns, framework quirks), use a project guidelines skill if one exists — ADRs are for decisions with context, alternatives, and rationale; guidelines are for terse conventions future agents need to follow.
effort: medium
---

# Create an ADR

## Steps

1. **Auto-detect numbering**: Scan `docs/decisions/` for existing ADR files matching `NNNN-*.md`. Use the next sequential number. If the directory doesn't exist, start at `0001`.

2. **Gather context**: Gather context from the current conversation first. Only ask if the decision and considered options aren't already clear.

3. **Review existing ADRs**: Read all existing ADR files and check whether any are related to this new decision:
   - **Superseded**: an existing ADR that this decision fully replaces (e.g. the new decision reverses or obsoletes the old one). Note the ADR number and title — the new ADR will supersede it, and the old one will need its status updated.
   - **Extended**: an existing ADR this decision builds on or narrows without replacing it entirely. Note it for the "Related" field.
   - **Conflicting**: an existing ADR that appears to contradict this decision. Flag it to the user before drafting.

   If no existing ADRs are related, proceed without comment.

4. **Draft the ADR**: Write the full ADR content using the template below and present it to the user as a markdown code block. Populate "Supersedes" and "Related" fields if applicable. Do NOT create any file yet.

5. **Wait for approval**: Ask the user to review the draft and confirm or request changes before proceeding. If any existing ADRs will be superseded or updated, list them explicitly so the user knows what else will change. Do not create or edit any file until the user explicitly approves.

6. **Create the file** at `docs/decisions/NNNN-kebab-case-title.md` only after approval, incorporating any requested changes. Create the directory if it doesn't exist.

7. **Update superseded ADRs**: For each ADR this decision supersedes, edit it to set `Status: superseded` and fill in `Superseded by: NNNN - Title` with the new ADR's number and title. Keeping the superseded chain accurate is what makes the decision graph navigable — a stale status misleads future readers into thinking a replaced decision is still live.

8. **Update the index**: If `docs/decisions/README.md` exists, read it first and append the new ADR in the same style as existing entries.

## Template

```markdown
# NNNN - Title

- **Status**: proposed
- **Supersedes**: <!-- NNNN - Title, only if this ADR replaces a previous one -->
- **Superseded by**: <!-- NNNN - Title, only if this ADR is later replaced -->
- **Related**: <!-- NNNN - Title, only if this ADR extends or relates to another -->
- **Date**: YYYY-MM-DD

## Context and Problem Statement

[2-3 sentences describing the problem or question driving this decision.]

## Considered Options

1. [Option A]
2. [Option B]
3. [Option C]

## Decision Outcome

Chosen option: "[Option X]", because [1-2 sentence justification].

### Positive Consequences

- [benefit 1]
- [benefit 2]

### Negative Consequences

- [tradeoff 1]
- [tradeoff 2]

## Pros and Cons of the Options

### Option A

- Good, because [argument]
- Bad, because [argument]

### Option B

- Good, because [argument]
- Bad, because [argument]

### Option C

- Good, because [argument]
- Bad, because [argument]
```

## Anti-patterns

- Don't fill in sections with vague filler. If something is unknown, mark it as "TBD" or omit the section.
- Don't create ADRs for trivial choices that don't warrant documentation.
- Don't use status values other than: proposed, accepted, deprecated, superseded.
- When marking an ADR as superseded, always fill in "Superseded by" with the number and title of the replacing ADR.
- Don't leave superseded ADRs stale — always update the old ADR's status and "Superseded by" field in the same step as creating the new one.
- Don't mark an ADR as "Related" just because it touches the same area — only use it when there's a meaningful dependency or extension relationship.
