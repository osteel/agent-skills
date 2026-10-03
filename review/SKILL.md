---
name: review
description: Deep code review. Use whenever the user asks to "review", "check", "audit", or "critique" recent code changes — uncommitted work, a feature branch, or an open PR. Also trigger proactively after implementing a non-trivial feature when quality assurance would be valuable. Reviews the diff against a full rubric, validates each finding before acting, then applies the fixes that hold up. Works on uncommitted work, branch commits, and open PRs.
effort: max
---

# Deep Code Review

Review recent changes thoroughly, then apply the findings that make sense.

Do the review yourself, in this context — don't delegate it to a subagent. A reviewer needs the surrounding code, the plan, and the project's conventions to tell a real defect from a false positive, and every hand-off costs some of that. If a caller wants this isolated, the caller runs the whole skill in a subagent.

## Step 1: Determine scope and gather context

Run in parallel:

```bash
git status --short
git diff HEAD
git branch --show-current
gh pr view --json number,title,headRefName,baseRefName,body 2>/dev/null || true
find . -maxdepth 2 \( -name "PLAN.md" -o -name "plan.md" \) 2>/dev/null | head -5
```

Also read any `CLAUDE.md` files in the root and directories touched by the diff.

Determine the review scope using this priority order:

1. **Open PR on current branch** — if `gh pr view` returned a PR, the scope is `git diff <base>...HEAD`
2. **Uncommitted changes** — if `git diff HEAD` is non-empty, review those
3. **Recent branch commits** — if on a non-main branch with no uncommitted changes, review all commits since branching: `git log main..HEAD --oneline` and `git diff main...HEAD`

If none of the above yields any changes, stop and tell the user there is nothing to review.

If a plan file exists, read it — it provides crucial context on what was *intended*, helping the reviewer assess whether the implementation matches the design.

## Step 2: Review the diff

Read `references/reviewer-prompt.md` and work through its rubric against the full diff, with the CLAUDE.md contents, PR description, and plan file in hand. Look past the obvious defects; this step is what the skill is for.

Produce a findings list before you change anything. Separating "what's wrong" from "what to do about it" is what makes Step 3 possible; if you fix as you read, you lose the chance to check your own work.

## Step 3: Validate findings

Go back over your own findings with a skeptical eye — you wrote them while forming a picture of the code, and some won't survive contact with the surrounding context. For each finding:

- **Critical/Major**: Evaluate whether it's a genuine issue in context. Look at surrounding code if needed. Accept it if it holds up; discard it if it's a false positive (explain why).
- **Minor**: Use judgment — apply obvious wins, skip borderline nitpicks unless clearly correct.

Don't apply findings that contradict the plan or that require architectural decisions the user hasn't made.

## Step 4: Apply fixes

For each accepted finding, apply the fix directly. After all edits:

- Check whether a test suite exists (look for `tests/`, `spec/`, `__tests__/`, or equivalent — or check `package.json`/`composer.json`/`Makefile` for test scripts). If one is found, invoke the `test` skill.
- If a fix causes test failures, either fix the test (if the test was wrong) or reconsider the fix.

If the caller owns test execution (as `pipeline` does), skip the full-suite run and note it in your report — the caller's run covers what targeted runs miss.

## Step 5: Report

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
