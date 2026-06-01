---
name: tackle
description: Pick up a task from the plan and drive it to completion — context gathering, planning, implementation, and the full post-implementation pipeline. Use when the user says things like "tackle X", "let's work on X", "implement X", "start on X", "pick up X", "work through X", or just "/tackle".
effort: max
---

# Tackle a Task

**Arguments:** `$ARGUMENTS`

Drive a task from identification through to a clean, reviewed, tested implementation.

---

## Resuming an in-progress tackle

If the branch already has commits and a plan file exists, you're likely resuming. Read the plan file, check git log for what's done, and pick up from the first incomplete step. Ask the user to confirm before re-running any step that appears already complete.

If the current branch name matches `^\d+-` and `gh` is available, treat the leading number as a GitHub issue reference and fetch it (`gh issue view <N> --json number,title,body,labels,state`) for additional context. If the issue isn't found, ignore silently — the prefix may be coincidental.

---

## Step 1: Identify the task

### If `$ARGUMENTS` is non-empty

1. **GitHub issue check.** If `$ARGUMENTS` is a bare number (e.g. `42`) or starts with `#` (e.g. `#42`), and `gh` is available, treat it as a GitHub issue reference. Run `gh issue view <N> --json number,title,body,labels,state` and use the issue title + body as the task description. Quote the title so it's clear what was picked up. Skip the plan-file lookup in this case — the issue is the source of truth.

2. Otherwise, search for a plan file. Look for `PLAN.md`, `plan.md`, or any `*.md` file whose name suggests a project plan (glob `**/PLAN.md`, `**/plan.md`). If found, read it.

3. Determine whether `$ARGUMENTS` refers to something in the plan:
   - If it matches a specific phase, task, or item in the plan (by ID, heading, or description), use that plan entry as the task description. Quote the relevant section so it's clear what was matched.
   - If it doesn't match anything in the plan but reads as a clear, self-contained description (e.g. "add email validation to the registration form"), treat it as the task description directly.
   - If it's ambiguous — matches nothing and isn't self-explanatory — tell the user what you found (or didn't find) and ask them to clarify.

### If `$ARGUMENTS` is empty

Ask the user: "What task would you like to tackle? (You can also pass a GitHub issue number, e.g. `/tackle 42`.)" Wait for their response, then treat it as the task description and continue from the issue/plan lookup above.

---

## Already-planned check

Before running Steps 2 and 3, check whether the task already has a plan:

- Does `PLAN.md` (or `plan.md`) exist and contain a section that covers this task with concrete implementation steps?
- Or does the task description itself (from arguments or conversation context) already contain a structured implementation plan?

If yes, **skip Steps 2 and 3** and go directly to Step 4. Present a brief summary of the plan you found so the user can confirm it's the right one before continuing.

---

## Step 2: Context gathering and planning

### 2.1 Context gathering (subagent)

Delegate context gathering to a subagent using the Agent tool. Use the best available model — accurate context gathering directly shapes plan quality. Pass it:

- The task description
- The plan file contents (if found)
- Any conversation context needed to understand the task

The subagent should read everything needed to understand the task fully:

- Relevant ADRs — scan `docs/decisions/` or `docs/adr/` filenames, read the ones related to the task
- Relevant spec/product docs — any `SPEC.md`, `PRD.md`, or equivalent
- Source files directly related to the task (models, controllers, services, tests, views — whatever applies)

The subagent should return the gathered context — key excerpts, file paths, and any observations relevant to planning — but not produce a plan itself.

Wait for the subagent to return before continuing.

### 2.2 Planning (main agent, /grill-me)

Using the gathered context, invoke `/grill-me` to stress-test the approach with the user before writing a plan. Use the best available model.

The interview should surface:

- What will be created or changed (files, classes, DB schema, documentation, etc.)
- Key decisions or trade-offs
- Any risks or unknowns
- Any clarifying questions that must be answered before implementation can begin

Once `/grill-me` reaches shared understanding, synthesise the outcomes into a concise implementation plan.

---

## Step 3: Refine and approve the plan

Present the plan and any questions from the planning subagent to the user. Answer questions and incorporate feedback. If the user requests changes, relay them to the subagent (or revise directly if minor) and re-present.

Repeat until the user approves the plan.

---

## Step 4: Branch setup

Run `git branch --show-current`.

### If on `main` or `master`

Suggest a branch name derived from the approved plan and task description (kebab-case, concise, e.g. `add-email-validation`). If the task came from a GitHub issue, prefix the slug with the issue number (e.g. `42-add-email-validation`) — `wrap-up` uses this prefix to add `Fixes #N` to the PR body. Then ask:

> I suggest the branch name `<name>`. Shall I create it? If you'd prefer a different name, enter it below and I'll create that instead.

Wait for the user's response, then run `git checkout -b <branch>` with the confirmed or provided name before continuing.

### If on a feature branch

Run the following checks in parallel:

1. **Committed work** — `git log main..HEAD --oneline` (or `master..HEAD`). Any output means commits exist on this branch.
2. **Open PR** — `gh pr list --head <branch> --state open`.
3. **Closed/merged PR** — `gh pr list --head <branch> --state closed`.

If any check returns results, warn the user with a summary of what was found, e.g.:

> This branch already has 3 commits, an open PR (#42), and a previously closed PR (#17). Proceeding will add implementation work on top of the existing state. Are you sure you want to continue?

Wait for explicit confirmation before proceeding. If the user declines, stop and let them sort out the branch situation first.

---

## Step 5: Implementation (subagent)

Delegate the full implementation to a subagent using the Agent tool. Use the default-tier model for this subagent — implementation does not require a planning-grade model. Pass it:

- The approved implementation plan
- The task description
- All relevant context gathered in Step 2 (plan file, ADRs, spec excerpts, key source files)
- The contents of `CLAUDE.md`

Wait for the subagent to complete and report back. If it reports failures or blockers, resolve them before continuing.

---

## Step 6: Post-implementation pipeline

Invoke the `pipeline` skill (Skill tool, name `pipeline`) to run the full quality pipeline through to a PR. Pass it a brief so its subagents judge against intent and its final report is complete:

- the task description and the approved plan,
- the context gathered in Step 2 (ADRs, spec excerpts, key files),
- the models already used upstream — planning (best available, Steps 2.1/2.2) and implementation (default tier, Step 5) — as upstream report rows to fold into the pipeline's model table.

`pipeline` owns the quality pipeline, its final report, and the post-PR hand-off. When it returns, the task is complete — do not add further steps here.
