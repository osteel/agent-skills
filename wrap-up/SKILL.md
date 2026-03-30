---
name: wrap-up
description: Wraps up completed work — use when the user says things like "ship it", "let's wrap up", "create a PR", "commit and push", or "I'm done". Updates PLAN.md if present, updates AI guidelines and ADR docs where applicable, commits everything, then opens or updates a GitHub PR. Guards against working on main.
effort: high
user-invocable: true
disable-model-invocation: true
---

Follow these steps in order. Do not skip steps.

## 1. Check the current branch

Run `git branch --show-current`.

If the branch is `main` or `master`, stop and tell the user they're on the main branch. Suggest a sensible branch name based on recent changes or the current task, and offer to create the branch before continuing.

## 2. Run the test suite

Invoke the `test` skill. If all tests pass, continue. If any fail, stop and resolve them before proceeding.

## 3. Check for PLAN.md

Use the Glob tool to search recursively for `PLAN.md` or `plan.md` anywhere in the working directory tree (pattern `**/PLAN.md`).

If found:
- Read it
- Run `git diff HEAD` and `git status --short` to understand what changed
- Determine which phases or tasks should be marked completed and what (if anything) needs updating
- Present your proposed changes to the user and wait for approval before editing PLAN.md
- Apply approved changes to PLAN.md before continuing

## 4. Update AI guidelines (Laravel projects only)

Check if an `.ai/guidelines/` directory exists in the project root. If it does not exist, skip this step silently.

If it exists, examine the uncommitted diff (`git diff HEAD`) for any non-obvious project-specific decisions that future agents would not infer from standard Laravel patterns or existing guidelines. If you find anything worth recording, invoke the `laravel-guidelines` skill (that skill may ask you questions — answer them and continue), then run `artisan boost:install` to regenerate `CLAUDE.md`. If nothing qualifies, skip silently.

## 5. Create an ADR (if applicable)

Check if a directory for ADRs exists in the project (look for `docs/decisions/`, `docs/adr/`, or similar). If no such directory exists, skip this step silently.

If it exists, examine the uncommitted diff (`git diff HEAD`) for architectural or significant design decisions that warrant an ADR. If you identify one, invoke the `adr` skill (that skill may ask you questions — answer them and continue). If nothing warrants an ADR, skip silently.

## 6. Commit uncommitted changes (if any)

Run `git status --short` (never `-uall`).

If there are staged or unstaged changes (including any PLAN.md or ADR files from the steps above):
- Run `git diff HEAD` to understand what changed
- Stage relevant files by name (never `git add -A` or `git add .`); if `git diff HEAD` shows files that seem unrelated to the current task, flag them to the user before staging and ask whether to include them
- Write a clear commit message and commit immediately (no confirmation needed)
- Commit using the project's default git config (no co-author lines, no Claude attribution)

## 7. Push the branch

Check if the branch has a remote tracking branch and is ahead. Push with `-u origin <branch>` if needed.

## 8. Check for an existing PR

Run `gh pr view --json number,title,url 2>/dev/null`.

Branch on the result:
- **No PR exists** → go to step 9
- **PR exists** → go to step 10

## 9. Create the PR

Gather context:
- `git log main...HEAD --oneline` — all commits on this branch
- `git diff main...HEAD` — full diff

Write a PR title (under 70 chars) and body using this format:

```
## Summary
- <bullet points covering what changed and why>

## Test plan
- <bulleted checklist of what to verify>
```

Do not add "Generated with Claude Code" or any mention of Claude/AI in the PR description. Create with (no confirmation needed):

```
gh pr create --title "..." --body "$(cat <<'EOF'
...
EOF
)"
```

Then print the PR URL.

## 10. Update the existing PR

Note: this rewrites the entire PR body. If the existing description contains manually-added notes or reviewer context worth preserving, incorporate them into the rewrite rather than discarding them.

Gather context:
- `git log main...HEAD --oneline`
- `git diff main...HEAD`
- Current PR body: `gh pr view --json body -q .body`

Rewrite the PR description to reflect the current state of the branch (same format as step 9). Update with (no confirmation needed):

```
gh pr edit --body "$(cat <<'EOF'
...
EOF
)"
```

Then print the PR URL.
