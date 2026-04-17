---
name: wrap-up
description: Wraps up completed work — use when the user says things like "ship it", "let's wrap up", "create a PR", "commit and push", or "I'm done". Updates PLAN.md if present, updates AI guidelines and ADR docs where applicable, commits everything, then opens or updates a GitHub PR. Guards against working on main.
effort: high
---

Follow these steps in order. Skip only where a step explicitly permits it.

## 1. Check the current branch

Run `git branch --show-current`.

If the branch is `main` or `master`, stop and tell the user they're on the main branch. Suggest a sensible branch name based on recent changes or the current task, and offer to create the branch before continuing.

## 2. Run the test suite

First, check whether a test run is necessary:

- Run `git status --short` and `git diff HEAD --name-only`.
- If there are no uncommitted changes, or all changes are to documentation or configuration files only (e.g. `.md` files, `PLAN.md`, ADR files, guidelines, lock files), skip this step — the suite was already passing and nothing that could affect tests has changed.
- If there are uncommitted changes to source or test files, invoke the `test` skill. If all tests pass, continue. If any fail, stop and resolve them before proceeding. If no test command can be identified (no config, no test scripts), note this and continue.

## 3. Check for PLAN.md

Use the Glob tool to search recursively for `PLAN.md` or `plan.md` anywhere in the working directory tree (pattern `**/PLAN.md`).

If found:
- Read it
- Run `git diff HEAD` and `git status --short` to understand what changed
- Determine which phases or tasks should be marked completed and what (if anything) needs updating
- Present your proposed changes to the user and wait for approval before editing PLAN.md
- Apply approved changes to PLAN.md before continuing

## 4–6. Post-implementation documentation

Examine `git diff HEAD` once, then check each of these in order:

| Check | Condition | Action |
|-------|-----------|--------|
| Agent rules | `.claude/rules/`, `.cursor/rules/`, `.windsurf/rules/`, `.copilot/rules/` exists | Add/update rules for non-obvious constraints; apply to all rules dirs found |
| Laravel guidelines | `.ai/guidelines/` exists | Invoke `laravel-guidelines` if diff contains non-standard decisions; run `artisan boost:install` after |
| ADR | `docs/decisions/` or `docs/adr/` exists | Invoke `adr` if diff contains an architectural or significant design decision |
| Documentation | Any docs directory exists (`docs/`, `doc/`, `documentation/`, or similar) | Update any documentation files affected by the diff — changelogs, READMEs, API docs, guides. Only update what the diff actually changes; don't rewrite unrelated docs |

If none of the directories exist, skip this section silently. If nothing in the diff qualifies for a given check, skip that row silently.

## 7. Commit uncommitted changes (if any)

Run `git status --short` (never `-uall`).

If there are staged or unstaged changes (including any PLAN.md or ADR files from the steps above):
- Run `git diff HEAD` to understand what changed
- Stage relevant files by name (never `git add -A` or `git add .`); if `git diff HEAD` shows files that seem unrelated to the current task, flag them to the user before staging and ask whether to include them
- Write a clear commit message and commit immediately — no confirmation needed, because the user invoking "ship it" / "wrap up" is the consent
- Commit using the project's default git config (no co-author lines, no Claude attribution)

## 8. Push the branch

Check if the branch has a remote tracking branch and is ahead. Push with `-u origin <branch>` if needed.

## 9. Check for an existing PR

Run `gh pr view --json number,title,url,headRefOid 2>/dev/null`.

Branch on the result (see `references/pr-update.md` for the ancestor check command):
- **No PR exists** → go to step 10
- **PR exists** → run the ancestor check. If `yes`, the PR belongs to this branch's history → go to step 11. If `no`, the branch name was reused → go to step 10.

## 10. Create the PR

Gather context:
- `git log main...HEAD --oneline` — all commits on this branch
- `git diff main...HEAD` — full diff

Write a PR title (under 70 chars) and body using the format in `references/pr-update.md`. Create immediately — no confirmation needed. Then print the PR URL.

## 11. Update the existing PR

Note: only reached when the existing PR's commit was confirmed as an ancestor of the current HEAD. This rewrites the entire PR body. If the existing description contains manually-added notes or reviewer context worth preserving, incorporate them into the rewrite rather than discarding them.

Gather context:
- `git log main...HEAD --oneline`
- `git diff main...HEAD`
- Current PR body: `gh pr view --json body -q .body`

Rewrite the PR description to reflect the current state of the branch (same format as step 10 — see `references/pr-update.md`). Update immediately — no confirmation needed. Then print the PR URL.
