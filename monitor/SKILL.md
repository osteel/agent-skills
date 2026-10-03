---
name: monitor
description: Watch CI for the current branch's PR, auto-fix failures, and stop once every check is green. Use when the user says things like "monitor CI", "babysit CI", "watch CI", "keep an eye on the checks", or after opening a PR when the user wants CI monitored to green. Observes CI only — review comments and merge state are out of scope. Designed to be driven by /loop (e.g. "/loop /monitor"); invoking it authorises committing and pushing CI fixes to the PR branch without further confirmation.
effort: medium
---

# Monitor CI

Each invocation is one iteration of a loop: inspect CI, act, then wait or stop. Pair with `/loop` so iterations repeat until green.

## Scope

- CI check runs only. Ignore review comments, merge conflicts, and mergeability — never merge.
- Auto-fix is pre-authorised: commit and push fixes to the PR branch without asking. Never force-push, never rewrite history, never push to the default branch.
- Run project commands via the project's own tooling (check CLAUDE.md / rules for wrappers before running `npm`/`composer`/etc. directly).

## Iteration

1. **Find the target**: `gh pr view --json number,url,headRefName,statusCheckRollup`. No PR for the current branch → report that and stop the loop.
2. **Read CI state**: `gh pr checks` (or `gh run list --branch <branch>`).
3. **Branch on state**:
   - **All green** → report success and stop the loop (`ScheduleWakeup` with `stop: true` when running under `/loop`).
   - **Runs pending/in progress** → schedule the next check, with the delay matched to how long this repo's CI usually takes; don't poll faster than results can change.
   - **Failures** → fix (below), then schedule the next check to observe the re-run.

## Fixing failures

1. `gh run view <run-id> --log-failed` for each failed run; diagnose from the logs.
2. Reproduce locally with the project's equivalent command (linter, static analysis, tests) before changing anything.
3. Fix the root cause, verify locally, commit with a plain message describing the fix, push to the PR branch.
4. Looks flaky (network blip, runner death, timeout with no code signal)? `gh run rerun <run-id> --failed` once. If it fails again, treat as real.
5. **Unfixable** (missing secrets, infra outage, fix would need a design decision, or the same check failed after two fix attempts) → stop the loop and report what's blocking. Don't thrash.

## Reporting

On stop, summarise: checks watched, fixes pushed (commit SHAs), reruns triggered, final CI state.
