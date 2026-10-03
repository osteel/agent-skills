---
name: test
description: Run the test suite and fix any failures. Use after writing or changing code, when tests are failing, or when the user asks to run tests, check tests, or make tests pass. When the user only asks to "run tests" (not "fix" or "make them pass"), run the suite and report — don't auto-fix without confirmation.
effort: medium
---

# Run Tests and Fix Failures

## Running without a user

The approval gates below assume someone is there to answer. If you're running as a subagent, nobody is — waiting would hang the caller. In that mode: fix the failures whose cause and remedy are unambiguous, leave the ones that need a judgment call, and put both lists in your report. A clear report the caller can act on beats a guess, and beats a stall.

## Notes

- Prefer fixing source code bugs over adjusting tests to pass
- If a test seems wrong but you're unsure, ask the user before changing it — or, as a subagent, leave it and flag it
- For flaky tests, look for race conditions, timing issues, or shared state
- Don't skip or delete tests without user approval
- If tests require environment setup (database, API keys, Docker), inform the user

## Phase 1: Run Tests

1. **Identify the test command** from the project (config files, `package.json` scripts, `composer.json`, etc.)
2. **Run the test suite** and capture the output
3. **If all tests pass**, report success and stop

## Phase 2: Diagnose Failures

For each failing test, start by running just the failing test(s) in isolation — this is faster and gives a cleaner signal than re-running the full suite on every diagnosis loop.

1. **Locate the relevant code**: the test file, the source being tested, and any fixtures or mocks.

2. **Determine the root cause**: is the test wrong (outdated expectations), the code wrong (bug), or the environment wrong (missing setup)?

## Phase 3: Fix Failures

1. **If the user asked only to run the tests**, summarize findings and wait for approval before making changes; if they asked to fix them or make them pass, proceed:
   - What failed and why
   - Whether the fix should be in the test or source code
   - Your proposed approach

   As a subagent, skip the wait: proceed with the unambiguous fixes and report the rest (see "Running without a user").

2. **Implement the fix**:
   - If test is outdated: update the test expectations
   - If code is buggy: fix the source code
   - If setup is missing: add necessary fixtures/mocks

3. **Re-run the specific failing test** to verify, then run the full suite to check for regressions

4. **If still failing**, re-diagnose and repeat

**One green full-suite run is the answer.** Don't re-run it to double-check, and don't run it again "for the record" — a second pass on unchanged code tells you nothing the first didn't, and on a slow suite it's the most expensive way to learn that. If a clean result is surprising, spend the time confirming the tests actually exercise the change (read them, or break the behaviour and watch them fail) rather than repeating the same run.

**If the runner is impact-filtered, report what actually ran.** Pest's Tia, `jest --onlyChanged`, pytest-testmon and friends execute the affected tests and replay the rest from cache — while still printing the *total* count. Trust the verdict; these tools track dependencies precisely, and re-running everything by hand defeats the point of having them. But don't launder a replay into "the full suite passed": say which tests ran, and quote the runner's own summary of what it considered affected. Check project docs for the escape hatch (`--no-tia`, `--ci`, …) and use it only when you have a concrete reason to distrust the graph.

## Phase 4: Complete

1. **Once all tests pass**, summarize:
   - Tests that were fixed
   - Changes made (test vs source code)
   - Any concerns or notes for the user

2. **If some tests cannot be fixed** (e.g., external dependency, requires discussion):
   - Explain what's blocking
   - Suggest next steps
   - Ask the user how to proceed
