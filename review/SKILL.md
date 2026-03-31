---
name: review
description: Deep code review using the best available model. Use whenever the user asks to "review", "check", "audit", or "critique" recent code changes — uncommitted work, a feature branch, or an open PR. Also trigger proactively after implementing a non-trivial feature when quality assurance would be valuable. Spawns a dedicated reviewer sub-agent (Opus, extended thinking), then the main agent validates findings and applies sensible fixes.
effort: max
---

# Deep Code Review

Review recent changes thoroughly using a sub-agent, then apply the findings that make sense.

## Step 1: Determine what to review

Run these in parallel to understand the current state:

```bash
git status --short
git diff HEAD
git branch --show-current
gh pr view --json number,title,headRefName,baseRefName 2>/dev/null || true
```

Determine the review scope using this priority order:

1. **Open PR on current branch** — if `gh pr view` returned a PR, the scope is `git diff <base>...HEAD`
2. **Uncommitted changes** — if `git diff HEAD` is non-empty, review those
3. **Recent branch commits** — if on a non-main branch with no uncommitted changes, review all commits since branching: `git log main..HEAD --oneline` and `git diff main...HEAD`

If none of the above yields any changes, stop and tell the user there is nothing to review. (If they're on main with a clean tree, suggest checking out the relevant branch or stashing in-progress work first.)

### Check for a local plan file

Look for a plan file that may describe the intended implementation:

```bash
find . -maxdepth 2 \( -name "PLAN.md" -o -name "plan.md" \) 2>/dev/null | head -5
```

If a plan file exists, read it. It provides crucial context on what was *intended*, helping the reviewer assess whether the implementation matches the design.

## Step 2: Gather supporting context

Run in parallel:

- Read any `CLAUDE.md` files in the root and in directories touched by the diff
- If on a PR, read PR description: `gh pr view --json title,body`

## Step 3: Spawn the reviewer sub-agent

Launch a **single sub-agent** using the Agent tool with `model: "opus"` (this always resolves to the latest Opus). Set `thinking` budget high — at least 10,000 tokens if the tool supports it.

Provide the sub-agent with the full diff, CLAUDE.md contents, PR description, and plan file (if any). Use this prompt:

---

**Sub-agent prompt:**

You are a senior engineer performing a thorough code review. Your job is to find real problems — not to nitpick or pad the review.

Review the following diff across these dimensions, in order of severity:

1. **Logic & correctness** — Does the code do what it's supposed to? Off-by-one errors, wrong conditions, missing edge cases, incorrect assumptions about data.

2. **Security** — Injection vulnerabilities (SQL, command, XSS), insecure defaults, secrets in code, improper input validation at system boundaries, auth/authz issues, unsafe deserialization.

3. **Simplicity & over-engineering** — Is anything needlessly complex? Premature abstractions, unnecessary indirection, dead code, redundant parameters. Could this be simpler without loss of correctness?

4. **Good practices** — SOLID principles where applicable, appropriate error handling, no magic values, no swallowed exceptions, no silent failures. Check CLAUDE.md for project-specific conventions and flag any violations.

5. **Performance** — N+1 queries, unnecessary allocations in hot paths, missing indexes (if schema changes present), O(n²) where O(n) is trivial.

6. **Test coverage** — Are new code paths tested? Are edge cases from the logic review covered? Are tests testing behaviour, not implementation?

If a plan or design doc was provided: does the implementation match what was planned? Flag deviations — they may be intentional improvements or accidental omissions.

For each finding, provide:
- Severity: `critical` / `major` / `minor`
- File and line reference
- What the problem is and why it matters
- A concrete suggestion for fixing it (code snippet if helpful)

Exclude:
- Pre-existing issues not touched by the diff
- Things a linter/compiler/CI would catch automatically
- Pedantic style issues not mentioned in CLAUDE.md
- Hypothetical future problems with no concrete evidence

Output format:

```
## Summary
<2-3 sentences on overall quality and the most important concern>

## Findings

### [CRITICAL/MAJOR/MINOR] <short title>
**File:** `path/to/file.ext:line`
**Issue:** <what's wrong>
**Fix:** <concrete suggestion>
```

If there are no real findings, say so clearly.

---

## Step 4: Validate findings

Read the sub-agent's output. For each finding:

- **Critical/Major**: Evaluate whether it's a genuine issue in context. Look at surrounding code if needed. Accept it if it holds up; discard it if it's a false positive (explain why).
- **Minor**: Use judgment — apply obvious wins, skip borderline nitpicks unless clearly correct.

Don't apply findings that contradict the plan or that require architectural decisions the user hasn't made.

## Step 5: Apply fixes

For each accepted finding, apply the fix directly. After all edits:

- Check whether a test suite exists (look for `tests/`, `spec/`, `__tests__/`, or equivalent — or check `package.json`/`composer.json`/`Makefile` for test scripts). If one is found, invoke the `test` skill.
- If a fix causes test failures, either fix the test (if the test was wrong) or reconsider the fix.

## Step 6: Report

```
## Review complete

**Scope:** <uncommitted changes / branch commits / PR #N>

### Applied (<N>)
- `file:line` — <one-line description>

### Noted but not applied (<N>)
- <finding> — <reason not applied>

### No issues found
- <dimension, e.g. "Security">
```

If nothing was found, say so and stop.
