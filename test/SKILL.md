---
name: test
description: Run the test suite and fix any failures. Use after writing or changing code, when tests are failing, or when the user asks to run tests, check tests, or make tests pass. When the user only asks to "run tests" (not "fix" or "make them pass"), run the suite and report — don't auto-fix without confirmation.
effort: medium
---

# Run Tests and Fix Failures

## Notes

- Prefer fixing source code bugs over adjusting tests to pass
- If a test seems wrong but you're unsure, ask the user before changing it
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

1. **Summarize findings to the user** and wait for approval before making changes:
   - What failed and why
   - Whether the fix should be in the test or source code
   - Your proposed approach

2. **Implement the fix**:
   - If test is outdated: update the test expectations
   - If code is buggy: fix the source code
   - If setup is missing: add necessary fixtures/mocks

3. **Re-run the specific failing test** to verify, then run the full suite to check for regressions

4. **If still failing**, re-diagnose and repeat

## Phase 4: Complete

1. **Once all tests pass**, summarize:
   - Tests that were fixed
   - Changes made (test vs source code)
   - Any concerns or notes for the user

2. **If some tests cannot be fixed** (e.g., external dependency, requires discussion):
   - Explain what's blocking
   - Suggest next steps
   - Ask the user how to proceed
