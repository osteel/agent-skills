---
name: cover
description: Identify and add tests for recent code changes. Use when the user says things like "add some tests", "make sure this is tested", "we need tests for this", or after completing a feature or fix. Invoke proactively when significant new behavior has been added without tests. Distinct from `test-audit` (which audits the full test suite for structural issues) — cover is scoped to what was just changed.
effort: medium
---

# Test Coverage for Recent Work

Identify and add tests for recent code changes.

## Purpose

Work out what tests the recent changes are missing. This is NOT about achieving 100% coverage — it's about ensuring the recent work has appropriate test coverage for the behavior that matters.

## Instructions

### Step 1: Identify Recent Work Scope

1. **Establish what was built and why.** Take it from whichever of these you actually have:
   - A **brief** from the caller (task description, plan excerpt) — use it; it states the intent the tests should hold the code to.
   - Otherwise the **conversation**, if you're in a session that has one.
   - Otherwise the **diff and commit messages** alone. That's enough to work from — read them for intent, not just mechanics.

   Note which of these you used; if you're inferring intent from the diff alone, say so in your report, because a reader may want to check your reading of it.

2. **Check git status** to see the changes:
   ```
   git status
   git diff --stat HEAD~5  # recent commits if applicable
   git diff HEAD  # uncommitted changes
   ```

### Step 2: Analyze Test Coverage Needs

For each modified file, consider what matters vs. what doesn't:

**Test**: new/changed logic and code paths, error handling, edge cases, integration points, complex conditionals, data transformations.

**Skip**: simple getters/setters, framework-provided functionality, already-covered code, trivial one-liners.

### Step 3: Check Existing Tests

1. **Find related test files**:
   - Look for existing test files for the modified code
   - Check what's already covered
   - Understand the testing patterns used in the project
   - If no test file exists yet for a modified file, look at sibling test files in the same directory for the pattern to follow

2. **Identify gaps**:
   - New code without any tests
   - New behavior not exercised by existing tests
   - Edge cases that aren't covered

### Step 4: Write Tests

For each gap identified:

1. **Follow project conventions** — match existing test file structure, naming, and patterns. If a skill specific to the project's testing framework is available, invoke it when writing tests.
2. **Use parameterized tests** (datasets, data providers, or the framework equivalent) when testing the same logic with multiple inputs — prefer a single parameterized test over several near-identical tests that differ only in values

### Step 5: Verify

1. **Run the new tests** to ensure they pass
2. **Run the affected test files** — the ones you touched plus their close neighbours, not the whole suite
3. **Intentionally break the code** to verify tests catch it (then revert). This mutation check is the only reliable way to confirm a test actually covers the code path it claims to — a passing test that doesn't catch breakage is worse than no test.

Those three are the whole verification story here. Don't reach for a full-suite run: this skill adds tests rather than changing source, so there's no regression for a wider run to find, and the mutation check is stronger evidence than a green suite anyway. Whoever runs the suite next — `pipeline`, `wrap-up`, or the user — will cover it.

### Step 6: Summary

Report back with:
- Tests added (file and test names)
- What behavior is now covered
- Any gaps that couldn't be easily tested (and why)

## Important

- **Don't over-test** — focus on behavior that matters, not coverage metrics
- **Don't expand scope** — only test the recent work, not the whole codebase
- **Match project style** — use the same test framework, patterns, and conventions
- **Ask if uncertain** — if unsure what testing approach to use, ask. As a subagent there's no one to ask: follow the closest existing test in the project and note the call you made in your report.
- **Tests should be maintainable** — avoid brittle tests that break on implementation changes