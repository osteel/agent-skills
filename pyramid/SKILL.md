---
name: pyramid
description: Audit the test suite against test pyramid principles and fix the issues found. Use when the user says things like "review our test pyramid", "our tests are slow", "audit the test suite", "improve test distribution", "too many end-to-end tests", "tests are taking too long", or "are our tests in the right layer". Also invoke proactively after a major feature milestone when the test suite has grown significantly. This skill identifies mislabeled tests, dataset opportunities, and redundant tests — then executes the approved fixes. Distinct from `cover` (which adds missing tests for recent changes) — pyramid audits the full suite for structural/distribution issues. Use the `test` skill to run the suite during Phase 1 and Phase 4. If the project has no distinct test layers, note this and skip this audit.
effort: max
---

# Test Pyramid Audit

Analyse the test suite, identify structural problems, present a prioritised report, get approval, then execute the changes.

## What this skill is for

A healthy test pyramid has many fast unit tests at the base, fewer integration tests in the middle, and a small number of end-to-end (browser/UI) tests at the top. Over time suites drift: tests accumulate in the wrong layer, slow tests duplicate faster ones, and groups of similar tests that could share parameterisation don't.

This skill corrects that drift.

The core principle: a test should *belong* to its layer, not just *sit* there. A test that boots a database or real browser but doesn't need to is paying a tax it didn't earn.

## Phase 1: Survey

Explore the full test suite before forming any opinions.

1. Find all test files and group them by layer (e.g. unit / integration / feature / browser / e2e — whatever the project uses).
2. Understand what setup each layer gets automatically (test base classes, database traits, shared fixtures).
3. Get a baseline runtime per layer. Run the suite (or each layer separately) and record how long each layer takes. If the project has a way to run layers independently (e.g. separate test commands or filter flags), use that. If not, run the full suite and note the total. This baseline is what you'll compare against at the end to quantify the improvement.

## Phase 2: Analyse

Work through each layer systematically. The goal is to find work that is genuinely worth doing — not to move things for the sake of it.

### Tests in the wrong layer or redundant at a higher layer

For each test file, ask: does this test actually need what its layer provides?

When reading tests, look for the real dependencies:
- Does it persist data or query a real database?
- Does it make real HTTP or browser requests?
- Does it need the application container or service locator?
- Or does it only work with plain objects, mocks, and value objects?

If a test doesn't need its layer's infrastructure, it belongs somewhere faster. Equally, if a browser/e2e test only verifies something already asserted by a faster test (content appearing, validation firing, behaviour fully covered at a lower level), it's redundant.

Keep browser tests only when they test something that genuinely requires a real browser or full stack:
- JavaScript interactions (form toggles, modal open/close, dropdowns, reactive UI)
- DOM state not visible at the component level (disabled attributes, CSS classes, aria attributes)
- Cross-component reactivity or timing-sensitive behaviour

When in doubt, keep the browser test. A false positive removal is worse than a redundant one.

### Dataset (parameterisation) opportunities

Look for groups of 3 or more tests in the same file that:
- Test the same logical behaviour with different input values
- Have nearly identical structure (same setup, same assertion shape, only the data differs)

These are strong candidates for parameterised/data-driven tests. Two near-identical tests is borderline — flag but don't insist.

### Duplicate tests across files

Check whether the same behaviour is tested in multiple files at the same layer. Flag these — keep the one with the more precise assertions, remove the other.

### Folder structure

Compare the test folder structure to the application source structure.

**When the app has a clear folder structure** (e.g. `app/Services/`, `src/Domain/`, `app/Http/Controllers/`): test files should mirror it. A test for `app/Services/PaymentService.php` should live at `tests/Unit/Services/PaymentServiceTest.php`, not at `tests/Unit/PaymentServiceTest.php` or scattered elsewhere. Flag any test file whose location doesn't reflect its subject's location in the app.

**When the app has no obvious structure to mirror**: check instead that tests are organised coherently — grouped by domain concept, layer, or feature rather than dumped in a flat directory. Flag files that seem arbitrarily placed or that would be hard to find given their subject matter.

Don't flag files that follow a legitimate project convention even if it differs from the app structure — look for evidence of intent before calling something a problem.

## Phase 3: Report

Present findings before touching anything. Use the report template in `references/report-template.md`.

Ask: "Should I proceed with all of these, or are there any you'd like to skip or discuss first?"

Wait for a response before touching any files.

## Phase 4: Execute

Apply only the approved changes. Work in this order to keep the suite green throughout:

1. **Move tests to a faster layer** — create files in the new location, delete the originals. Adjust any infrastructure dependencies (remove database setup traits if the test no longer needs them, adjust imports).
2. **Reorganise folder structure** — move files to their correct paths, creating subdirectories as needed. Update any references (autoloading, imports, CI config) that pointed to the old paths.
3. **Add datasets** — consolidate repeated tests into parameterised equivalents.
4. **Remove redundant browser/e2e tests** — edit files to remove the identified tests. Delete the file if it becomes empty.

After each change, run the affected tests to confirm they still pass. Run the full suite at the end.

## Phase 5: Wrap up

Run any project linters or formatters on changed files. Then re-run the suite (or each layer) and record the new runtime. Report:

- Files changed / deleted
- Net test count change per layer
- Runtime before → after (from the baseline recorded in Phase 1)
- Any approved changes that couldn't be executed cleanly (and why)

## Important constraints

- **Never remove a browser/e2e test without naming the specific lower-level test that already covers the same behaviour.** The report must be explicit.
- **Don't move a test just because it *could* run without its layer's infrastructure.** If the project has a strong convention (e.g. "unit tests are pure, no framework") respect it — flag the test as DB-free if useful, but don't force a move that would break the convention.
- **Don't consolidate datasets across files** — keep each file self-contained.
- **Don't expand scope** — only audit the test suite, not the application code.
- **Don't reorganise for its own sake** — only flag folder structure mismatches where the current location is genuinely confusing or inconsistent with the rest of the suite. A flat structure is fine if the project is small and the suite is easy to navigate.
- **Flag rather than guess** — if a test's dependencies are ambiguous, say so in the report instead of making an assumption.
