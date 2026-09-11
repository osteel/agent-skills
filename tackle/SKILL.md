---
name: tackle
description: Pick up a task from the plan and drive it to completion — context gathering, planning, implementation, and the full post-implementation pipeline. Use when the user says things like "tackle X", "let's work on X", "implement X", "start on X", "pick up X", "work through X", or just "/tackle".
effort: max
---

# Tackle a Task

**Arguments:** `$ARGUMENTS`

Drive a task from identification through to a clean, reviewed, tested and QA'd implementation.

---

## Resuming an in-progress tackle

If the branch already has commits and a plan file exists, you're likely resuming. Read the plan file, check git log for what's done, and pick up from the first incomplete step. Ask the user to confirm before re-running any step that appears already complete.

If the current branch name matches `^\d+-` and `gh` is available, treat the leading number as a GitHub issue reference and fetch it for additional context — both `gh issue view <N> --json number,title,body,labels,state` and `gh issue view <N> --comments`, since the JSON form returns the opening post only and drops the thread silently. If the issue isn't found, ignore silently — the prefix may be coincidental.

---

## Step 1: Identify the task

### If `$ARGUMENTS` is non-empty

1. **GitHub issue check.** If `$ARGUMENTS` is a bare number (e.g. `42`) or starts with `#` (e.g. `#42`), and `gh` is available, treat it as a GitHub issue reference. Run **both**:

   ```
   gh issue view <N> --json number,title,body,labels,state
   gh issue view <N> --comments
   ```

   **The comments are part of the task, not optional colour.** `--json body` returns the opening post only and drops the thread silently, with no hint that anything is missing. Comments routinely carry the parts that change the work: a prerequisite that landed first and changed the shape, a design decision already made, a constraint discovered later, or scope the author explicitly added or withdrew. A body that looks complete is not evidence there is nothing else — check.

   Use the title + body + comments together as the task description. Quote the title so it's clear what was picked up, and summarise anything in the comments that contradicts, narrows, or extends the body. Where a comment and the body disagree, the **later comment wins** — the body is rarely edited to match. Skip the plan-file lookup in this case — the issue is the source of truth.

2. Otherwise, search for a plan file. Look for `PLAN.md`, `plan.md`, or any `*.md` file whose name suggests a project plan (glob `**/PLAN.md`, `**/plan.md`). If found, read it.

3. Determine whether `$ARGUMENTS` refers to something in the plan:
   - If it matches a specific phase, task, or item in the plan (by ID, heading, or description), use that plan entry as the task description. Quote the relevant section so it's clear what was matched.
   - If it doesn't match anything in the plan but reads as a clear, self-contained description (e.g. "add email validation to the registration form"), treat it as the task description directly.
   - If it's ambiguous — matches nothing and isn't self-explanatory — tell the user what you found (or didn't find) and ask them to clarify.

### If `$ARGUMENTS` is empty

Ask the user: "What task would you like to tackle? (You can also pass a GitHub issue number, e.g. `/tackle 42`.)" Wait for their response, then treat it as the task description and continue from the issue/plan lookup above.

---

## Blocker check

Now that the task is identified, work out whether it depends on something that isn't done yet. Starting on a blocked task tends to produce work that has to be redone once the prerequisite lands — cheaper to catch it now than after implementation.

Look for two kinds of signal:

- **Explicit** — the task text says so: "blocked by", "depends on", "after X", "once Y is merged", a linked issue, a `Depends on #12` line, or a plan file where this item sits under a later phase whose earlier phases aren't complete.
- **Inferred** — nothing says it outright, but the task assumes something that doesn't exist yet: it extends a model, endpoint, or component that isn't in the codebase; it consumes an API or migration another plan item is meant to introduce; a sibling task in the plan clearly has to land first.

Check cheaply — read the plan file around this task, grep for the things the task assumes exist, and if the task came from a GitHub issue, look at what it references (`gh issue view <N>` output already includes the body; follow up on linked issues only if the body points at one).

If it looks blocked, say what you found and let the user decide:

> This looks like it depends on **<blocker>** — <one line of evidence, e.g. "PLAN.md phase 2 introduces the `Subscription` model this task extends, and it doesn't exist yet">. Do you want to tackle that first, or proceed anyway?

Wait for their answer. They may know the blocker is already handled elsewhere, or want to proceed regardless — either is fine, just don't decide it silently. If nothing suggests a dependency, move on without comment; don't manufacture doubt.

---

## Already-planned check

Before running Steps 2 and 3, check whether the task already has a plan:

- Does `PLAN.md` (or `plan.md`) exist and contain a section that covers this task with concrete implementation steps?
- Or does the task description itself (from arguments or conversation context) already contain a structured implementation plan?

If yes, **skip Steps 2 and 3** and go directly to Step 4. Present a brief summary of the plan you found so the user can confirm it's the right one before continuing.

---

## Step 2: Context gathering and planning

### 2.1 Context gathering (subagent)

Delegate context gathering to a subagent using the Agent tool. Use the latest Opus model — accurate context gathering directly shapes plan quality. Pass it:

- The task description
- **The full issue comment thread verbatim, if the task came from an issue** — not your summary of it. The subagent needs to reconcile what the comments say against what the code actually looks like now, which it can't do from a paraphrase.
- The plan file contents (if found)
- Any conversation context needed to understand the task

The subagent should read everything needed to understand the task fully:

- Relevant ADRs — scan `docs/decisions/` or `docs/adr/` filenames, read the ones related to the task
- Relevant spec/product docs — any `SPEC.md`, `PRD.md`, or equivalent
- Source files directly related to the task (models, controllers, services, tests, views — whatever applies)

The subagent should return the gathered context — key excerpts, file paths, and any observations relevant to planning — but not produce a plan itself.

Wait for the subagent to return before continuing.

### 2.2 Planning (main agent, /grill-me)

Using the gathered context, invoke `/grill-me` to stress-test the approach with the user before writing a plan. Use the latest Opus model.

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

Derive a branch name from the approved plan and task description (kebab-case, concise, e.g. `add-email-validation`). If the task came from a GitHub issue, prefix the slug with the issue number (e.g. `42-add-email-validation`) — `wrap-up` uses this prefix to add `Fixes #N` to the PR body.

Create it yourself with `git checkout -b <branch>` and say what you named it. **Do not ask for approval of the branch name** — plan approval is approval to start work, and a branch name is trivially renameable.

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

Delegate the full implementation to a subagent using the Agent tool. Use the latest Sonnet model for this subagent — implementation does not require a planning-grade model. Pass it:

- The approved implementation plan
- The task description
- All relevant context gathered in Step 2 (plan file, ADRs, spec excerpts, key source files)
- The contents of `CLAUDE.md`
- The reporting contract below, verbatim

### The reporting contract

Put both rules in the subagent's prompt. Each exists because an implementation agent broke it and the cost landed here.

1. **Finish before you stop.** A turn that ends with "I'll wait for the test run and report back" ends the agent — nothing wakes it, and the work lands on disk unverified with no report. If a check is slow, run it in the foreground and wait for it, or poll it to completion in the same turn. Never park on a background job.
2. **Report only what you observed.** Every result you state must come from output you actually read. If a check never returned, say that. An invented pass, or a claim that other agents corroborated something they never sent you, is worse than no report at all.

Wait for the subagent to report back, then check the tree yourself before continuing: `git status --short` and `git diff --stat`. If it reports failures or blockers, resolve them. If it stopped without reporting at all, the edits are still on disk and still unverified — verify them yourself rather than re-running the agent over a tree it has already changed.

---

## Step 6: Post-implementation pipeline

Invoke the `pipeline` skill (Skill tool, name `pipeline`) to run the full quality pipeline through to a PR. Pass it a brief so its subagents judge against intent and its final report is complete:

- the task description and the approved plan,
- the context gathered in Step 2 (ADRs, spec excerpts, key files),
- the models already used upstream — planning (latest Opus model, Steps 2.1/2.2) and implementation (latest Sonnet model, Step 5) — as upstream report rows to fold into the pipeline's model table.

`pipeline` owns the quality pipeline, its final report, and the post-PR hand-off. When it returns, the task is complete — do not add further steps here.
