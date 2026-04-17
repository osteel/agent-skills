---
name: laravel-guidelines
description: Manages project-specific AI guidelines in `.ai/guidelines/` for Laravel projects. Use when the user says things like "record a convention", "note this for future agents", "document this pattern", or wants to record, update, or remove a non-obvious project convention, or tricky detail that agents wouldn't infer from standard Laravel patterns or CLAUDE.md. Also activate automatically after completing any Laravel implementation task that introduced new models, services, jobs, observers, or non-standard behaviour — to capture decisions future agents wouldn't easily infer. For formal architectural decisions with context, alternatives, and rationale, use the `adr` skill instead — guidelines are for terse, agent-facing conventions, not decision records.
effort: medium
---

# Laravel Guidelines Manager

After completing a Laravel implementation task that introduced something non-standard, or when explicitly asked, review whether any non-obvious decisions were made that future agents would not easily infer. Apply the three-criteria filter below strictly — most tasks produce nothing worth recording. If nothing qualifies, do nothing and briefly say so.

## What belongs in guidelines

Only record details that are **all three** of:
- Project-specific (not standard Laravel convention)
- Not already covered in an existing `.ai/guidelines/` file, and not already covered in any `CLAUDE.md` content that is *not* sourced from `.ai/guidelines/`
- Something an agent would only discover through deep codebase analysis — not something guessable

**Important:** `CLAUDE.md` is partially generated from `.ai/guidelines/` via `artisan boost:install`, but may also contain hand-written content. Do not treat `.ai/guidelines/` content appearing in `CLAUDE.md` as a reason to remove it — that duplication is expected. Only avoid recording something that is already covered in the hand-written portion of `CLAUDE.md` (content not sourced from `.ai/guidelines/`).

Examples of what qualifies:
- Non-standard file locations that deviate from Laravel convention
- Invariants enforced at an unusual layer (e.g. business logic intentionally on a model)
- Tricky sequencing rules in jobs, observers, or event listeners
- Constraints that must never be violated and aren't obvious from the code structure

Examples of what does NOT qualify:
- Standard Eloquent patterns, relationship types, casting
- Anything already in `.ai/guidelines/` or in the hand-written sections of `CLAUDE.md`
- Implementation details that are self-documenting in the code

## Process

1. Check whether `.ai/guidelines/` exists in the project root. If not, and guidelines are warranted, create it.
2. Read all existing `.md` files in `.ai/guidelines/`.
3. Decide: does the new information belong in an existing file, warrant a new file, or require removing/amending a stale entry? If nothing qualifies, stop here.
4. Make the minimum necessary change — add, amend, or delete. Do not rewrite files wholesale.
5. Keep every entry as short as possible. One line per rule where possible.
6. Remind the user to run `artisan boost:install` to regenerate `CLAUDE.md` with the updated guidelines.

## File organisation

Name files by concern (e.g. `scheduler.md`, `notifications.md`, `auth.md`). A single `conventions.md` is fine for small projects. Each file should cover one concern end-to-end — don't split a single concern across files. Never create a file just to have one — only create when there is content that genuinely belongs there.
