---
name: finalise
description: Clean up completed feature work — remove dead code, false starts, and over-engineering from iteration. Use when the user says things like "clean this up", "tidy this", "remove the dead code", or "let's clean up before the PR".
effort: max
---

# Clean Up Recent Work

Clean up completed feature work.

## Purpose

Clean up the feature or refactor we've been working on in this session, or all uncommitted work. Remove false starts, dead-end approaches, and experimental code now that we have a working solution. This is NOT a general codebase cleanup — it's specifically about consolidating the recent work into a clean, final implementation.

## Instructions

### Step 1: Identify Recent Work Scope

1. **Review the conversation** to understand:
   - What feature/refactor we've been building
   - Which files were created or modified
   - What approaches were tried (including ones that didn't work)

2. **Check git status** to see uncommitted changes and recently modified files:
   ```
   git status
   git diff --stat HEAD~5  # recent commits if applicable
   git diff HEAD  # uncommitted changes
   ```

### Step 2: Identify Cleanup Targets

Look for these patterns in the files we touched:

1. **False starts**: Code that was written as part of an abandoned approach
   - Commented-out alternative implementations
   - Functions/methods that were superseded by a better approach
   - Imports that are no longer needed

2. **Experimental remnants**:
   - Debug logging/console statements added during development
   - Temporary variables or flags used for testing approaches
   - Hardcoded test values that should be parameterised

3. **Duplicated logic**:
   - Multiple ways of doing the same thing where we settled on one
   - Copy-pasted code that can now be consolidated

4. **Naming inconsistencies**:
   - Variables/functions named for the first approach that no longer fit
   - Misleading names from when the implementation was different

5. **Over-engineering from iteration**:
   - Abstractions added "just in case" that aren't needed
   - Configuration options that will never vary
   - Extra parameters that always have the same value

### Step 3: Clean Up

For each file in scope:

1. **Remove dead code** — don't comment it out, delete it
2. **Consolidate** the working approach into clean, readable code
3. **Rename** anything that no longer reflects its purpose
4. **Simplify** — remove unnecessary indirection or abstraction
5. **Ensure consistency** — make sure the final implementation follows project conventions

### Step 4: Verify

**Only if changes were made in Step 3:**

1. **Invoke the `test` skill** to run the suite and fix any failures introduced by the cleanup
2. **Run static analysis and linters if available** — renaming and removing code can introduce unused imports or type errors
3. **Run the build** if applicable
4. **Quick manual test** of the feature if appropriate

### Step 5: Summary

If changes were made, report:
- Files cleaned up
- What was removed/consolidated
- Any code you were uncertain about deleting — flag these explicitly rather than deleting silently

If no changes were needed, say so clearly and stop.

## Important

- **Don't expand scope** — only touch files related to the recent work
- **Preserve the working solution** — this is about cleaning, not reimplementing
- **Ask if uncertain** — if unsure whether something is a false start or intentional, ask
- **Keep it simple** — the goal is clarity and maintainability, not perfection
- **Let `cover` handle test gaps** — this skill is about code cleanliness, not test coverage; if you notice missing tests, note them but don't write them here