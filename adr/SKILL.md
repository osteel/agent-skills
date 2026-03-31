---
name: adr
description: Create architectural decision records in MADR format. Use when the user wants to document a technical decision, record an architecture choice, or create an ADR. Also invoke proactively after completing a significant architectural or design decision, even if the user hasn't explicitly asked for an ADR.
effort: medium
---

# Create an ADR

## Steps

1. **Auto-detect numbering**: Scan `docs/decisions/` for existing ADR files matching `NNNN-*.md`. Use the next sequential number. If the directory doesn't exist, start at `0001`.

2. **Gather context**: Gather context from the current conversation first. Only ask if the decision and considered options aren't already clear.

3. **Draft the ADR**: Write the full ADR content using the template below and present it to the user as a markdown code block. Do NOT create any file yet.

4. **Wait for approval**: Ask the user to review the draft and confirm or request changes before proceeding. Do not create the file until the user explicitly approves.

5. **Create the file** at `docs/decisions/NNNN-kebab-case-title.md` only after approval, incorporating any requested changes. Create the directory if it doesn't exist.

6. **Update the index**: If `docs/decisions/README.md` exists, read it first and append the new ADR in the same style as existing entries.

## Template

```markdown
# NNNN - Title

- **Status**: proposed
- **Superseded by**: <!-- only if status is superseded -->
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
