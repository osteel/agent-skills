---
name: test-audit
description: Audit the existing test suite for structural and quality issues — wrong-layer tests, redundant browser tests, infrastructure overuse, flaky/unreliable patterns, rotted skipped tests, unclear test names, dataset opportunities, duplicate tests, folder drift — then fix the approved issues. Use when the user says things like "audit the tests", "review the test suite", "our tests are slow", "our tests are flaky", "improve test distribution", "too many end-to-end tests", "tests are taking too long", "are our tests in the right layer", or "clean up the test suite". Also invoke proactively after a major feature milestone when the suite has grown significantly. Distinct from `cover` (which adds tests for recent code changes) — this skill audits what's already there and never adds net-new tests for uncovered code. Use the `test` skill to run the suite during the baseline and verification phases.
effort: max
---

# Test Suite Audit

Survey the existing test suite, identify issues across several dimensions, present a prioritised report, get approval, then execute the changes.

## What this skill is for

Test suites drift over time. Tests accumulate in the wrong layer, slow setup spreads to tests that don't need it, near-identical tests pile up where a single parameterised one would do, flaky patterns leak in, skipped tests rot. This skill is a structural review of the suite as it stands today.

The guiding question across every dimension: is each test pulling its weight, in the right place, in a way that won't betray you later?

This skill **never adds net-new tests for uncovered code** — that's what the `cover` skill is for. It only edits, moves, consolidates, deletes, or renames tests that already exist (and adds dataset cases that consolidate existing tests).

## Phase 1: Survey

Explore the suite before forming opinions.

1. Find all test files and group them by layer (e.g. unit / integration / feature / browser / e2e — whatever the project uses).
2. Understand what setup each layer gets automatically — base classes, shared fixtures, traits, helpers, any auto-applied database/HTTP/browser setup.
3. Get a baseline runtime per layer using the `test` skill. If layers can be run independently, record each separately. Otherwise record the total. This baseline is what you'll compare against at the end to quantify the improvement.

## Phase 2: Analyse

Work through the dimensions below systematically. The goal is to find work that is genuinely worth doing — not to move things for the sake of it. When a test's situation is ambiguous, flag it rather than guessing.

### Distribution: tests in the wrong layer

For each test file, ask: does this test actually need what its layer provides?

When reading tests, look for the real dependencies:
- Does it persist data or query a real database?
- Does it make real HTTP, browser, or filesystem calls?
- Does it need the application container, service locator, or framework boot?
- Or does it only work with plain objects, mocks, and value objects?

If a test doesn't need its layer's infrastructure, it belongs somewhere faster.

### Distribution: redundant browser/end-to-end tests

A browser/e2e test is redundant if a faster test already verifies the same behaviour. Keep browser tests only when they exercise something that genuinely requires a real browser or full stack:
- JavaScript interactions (form toggles, modal open/close, dropdowns, reactive UI)
- DOM state not visible at the component level (disabled attributes, CSS classes, aria attributes)
- Cross-component reactivity or timing-sensitive behaviour

When in doubt, keep the browser test. A false-positive removal is worse than a redundant one. Never propose removing a browser test without naming the specific lower-level test that already covers the same behaviour.

### Infrastructure overuse

A test can be in the right layer and still pull in setup it doesn't need — a unit test that resets the database, a feature test that boots a full browser context, fixtures applied globally that only a few tests actually use. Look for setup running for every test in a file when only one or two tests actually depend on it; that's a candidate for narrower scoping or splitting the file.

### Reliability: flaky patterns

Look for patterns that produce intermittent failures or that will silently lie when the world changes:
- Real `sleep` or arbitrary delays used to wait for state
- Time-based assertions without a frozen clock (current date/time, "now", relative dates)
- Randomness without a seeded source
- Real network calls (or unmocked outbound HTTP) in tests below the e2e layer
- Tests that read or write shared state (global singletons, the real filesystem outside a temp directory, a shared cache) without resetting it
- Tests that pass only when run in a particular order

Flag each pattern with the file and a short description. Fixing flakiness usually requires a small targeted edit — freezing time, seeding randomness, replacing a sleep with a polling helper, mocking the network call.

### Reliability: rotted skipped tests

Find tests marked skipped, todo, or otherwise disabled. For each, determine why. If the reason still applies (a known limitation, an external dependency intentionally stubbed out), leave it. If the reason is gone, the underlying code has moved on, or the skip has been there long enough that nobody can explain it, propose deletion. Skipped tests that nobody is going to revive are noise — they imply coverage that doesn't exist.

The skill is empowered to delete these after approval, in the same execute phase as the rest.

### Clarity: test names

Test names should describe the behaviour being verified, not the method being called. `it_persists_a_new_user` is clearer than `it_calls_save`; `rejects_an_unverified_email` is clearer than `test_handle_throws`. Flag names that read like implementation traces, are too vague to predict what the test does, or contradict what the test actually asserts.

Don't flag every name that could be marginally tighter — focus on names that would mislead a reader skimming the file.

### Structure: dataset (parameterisation) opportunities

Look for groups of three or more tests in the same file that:
- Test the same logical behaviour with different input values
- Have nearly identical structure (same setup, same assertion shape, only the data differs)

These are strong candidates for parameterised/data-driven tests. Two near-identical tests is borderline — flag but don't insist. Don't consolidate datasets across files; keep each file self-contained.

### Structure: duplicate tests across files

Check whether the same behaviour is tested in multiple files at the same layer. Flag these — keep the one with the more precise assertions, remove the other.

### Structure: folder layout

Compare the test folder structure to the application source structure.

**When the app has a clear folder structure**: test files should mirror it. A test for a service at `<app>/Services/PaymentService` should live at `<tests>/Unit/Services/PaymentServiceTest`, not at `<tests>/Unit/PaymentServiceTest` or scattered elsewhere. Flag any test file whose location doesn't reflect its subject's location in the app.

**When the app has no obvious structure to mirror**: check instead that tests are organised coherently — grouped by domain concept, layer, or feature rather than dumped in a flat directory. Flag files that seem arbitrarily placed or that would be hard to find given their subject matter.

Don't flag files that follow a legitimate project convention even if it differs from the app structure — look for evidence of intent before calling something a problem.

## Phase 3: Report

Present findings before touching anything. Use the report template in `references/report-template.md`.

Ask: "Should I proceed with all of these, or are there any you'd like to skip or discuss first?"

Wait for a response before touching any files.

## Phase 4: Execute

Apply only the approved changes. Work in this order to keep the suite green throughout:

1. **Move tests to a faster layer** — create files in the new location, delete the originals. Adjust any infrastructure dependencies (remove database/browser setup if the test no longer needs them, adjust imports).
2. **Reorganise folder layout** — move files to their correct paths, creating subdirectories as needed. Update any references (autoloading, imports, CI config) that pointed to the old paths.
3. **Trim infrastructure overuse** — narrow setup to where it's actually needed; split files where a small group of tests is dragging in heavy setup for the rest.
4. **Fix reliability issues** — freeze time, seed randomness, replace sleeps with polling, mock network calls, stop relying on shared state.
5. **Rename tests for clarity** — apply the approved rename suggestions.
6. **Add datasets** — consolidate repeated tests into parameterised equivalents.
7. **Delete rotted skipped tests** — remove the approved ones.
8. **Remove redundant browser/e2e tests** — edit files to remove the identified tests. Delete the file if it becomes empty.

After each change, run only the affected test files directly; the full run happens once, in Phase 5.

## Phase 5: Wrap up

Run any project linters or formatters on changed files. Then re-run the suite (or each layer) and record the new runtime. Report:

- Files changed / deleted
- Net test count change per layer
- Runtime before → after (from the baseline recorded in Phase 1)
- Any approved changes that couldn't be executed cleanly (and why)

## Important constraints

- **Never add a net-new test for uncovered code.** That's the `cover` skill's job. Editing, moving, consolidating, renaming, and deleting existing tests is fine; adding dataset cases that consolidate existing tests is fine; writing a test for behaviour that wasn't previously tested is not.
- **Never remove a browser/e2e test without naming the specific lower-level test that already covers the same behaviour.** The report must be explicit about the substitute.
- **Don't move a test just because it *could* run without its layer's infrastructure.** If the project has a strong convention (e.g. "unit tests are pure, no framework") respect it — flag the test as infrastructure-free if useful, but don't force a move that would break the convention.
- **Don't consolidate datasets across files** — keep each file self-contained.
- **Don't expand scope beyond the test suite.** Application code is not in scope. Style nits (formatting, import ordering) are linter territory, not audit territory.
- **Don't reorganise for its own sake** — only flag folder layout mismatches where the current location is genuinely confusing or inconsistent with the rest of the suite. A flat structure is fine if the project is small and the suite is easy to navigate.
- **Flag rather than guess** — if a test's dependencies, intent, or skip reason are ambiguous, say so in the report instead of making an assumption.
