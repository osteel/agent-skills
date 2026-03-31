---
name: tackle
description: Pick up a task from the plan and drive it to completion — context gathering, planning, implementation, and the full post-implementation pipeline. Use when the user says things like "tackle X", "let's work on X", "implement X", or just "/tackle".
effort: max
---

# Tackle a Task

**Arguments:** `$ARGUMENTS`

Drive a task from identification through to a clean, reviewed, tested implementation.

---

## Step 1: Branch guard

Run `git branch --show-current`.

If the branch is `main` or `master`, stop immediately. Tell the user they must create a branch before tackling any task. Suggest a sensible branch name based on `$ARGUMENTS` or recent context, and wait for them to act before continuing.

---

## Step 2: Identify the task

### If `$ARGUMENTS` is non-empty

1. Search for a plan file. Look for `PLAN.md`, `plan.md`, or any `*.md` file whose name suggests a project plan (glob `**/PLAN.md`, `**/plan.md`). If found, read it.

2. Determine whether `$ARGUMENTS` refers to something in the plan:
   - If it matches a specific phase, task, or item in the plan (by ID, heading, or description), use that plan entry as the task description. Quote the relevant section so it's clear what was matched.
   - If it doesn't match anything in the plan but reads as a clear, self-contained description (e.g. "add email validation to the registration form"), treat it as the task description directly.
   - If it's ambiguous — matches nothing and isn't self-explanatory — tell the user what you found (or didn't find) and ask them to clarify.

### If `$ARGUMENTS` is empty

Ask the user: "What task would you like to tackle?" Wait for their response, then treat it as the task description and continue from the plan lookup above.

---

## Step 3: Context gathering (plan mode)

Stay in plan mode throughout this step. Gather everything needed to understand the task fully before proposing anything.

Read the following, using judgment about which are relevant:

- The plan file (if not already read)
- Relevant ADRs — scan `docs/decisions/` or `docs/adr/` filenames, read the ones related to the task
- Relevant spec/product docs — any `SPEC.md`, `PRD.md`, or equivalent
- Source files directly related to the task (models, controllers, services, tests, views — whatever applies)

Ask the user any clarifying questions you need answered before you can write a solid implementation plan. Keep asking until you have everything you need. Batch questions where possible — don't ask one at a time if you can help it.

---

## Step 4: Propose an implementation plan (plan mode)

Write a concise implementation plan covering:

- What will be created or changed (files, classes, DB schema, etc.)
- Key decisions or trade-offs
- Any risks or unknowns

Keep it scannable — bullets over prose. The user should be able to approve or push back quickly.

Wait for the user to approve the plan. If they request changes, revise and re-present. Repeat until approved.

Once the plan is approved, exit plan mode.

---

## Step 5: Implementation (subagent)

Delegate the full implementation to a subagent using the Agent tool. Use the fastest capable model available for this subagent — implementation does not require a planning-grade model. Pass it:

- The approved implementation plan
- The task description
- All relevant context gathered in Step 3 (plan file, ADRs, spec excerpts, key source files)
- The contents of `CLAUDE.md`

Wait for the subagent to complete and report back. If it reports failures or blockers, resolve them before continuing.

---

## Step 6: Post-implementation pipeline

Run the following skills in order, each as a subagent. Run them sequentially — only start the next if the previous succeeded. If a step fails, resolve the failure before continuing.

Skip steps that have explicit conditions, as noted.

### 6.1 `/simplify`

**Condition:** Only run if the `simplify` skill is available (global skill or project skill).

Invoke the `simplify` skill. Wait for it to complete.

### 6.2 `/polish`

**Condition:** Only run if the implementation involved UI, design, or UX changes (new views, updated layouts, component changes, CSS/Tailwind).

Invoke the `polish` skill. Wait for it to complete.

### 6.3 `/review`

Invoke the `review` skill. Wait for it to complete.

### 6.4 `/test`

Invoke the `test` skill. Wait for it to complete. Fix any failures before moving on.

### 6.5 `/cover`

Invoke the `cover` skill. Wait for it to complete.

### 6.6 `/analyse`

Invoke the `analyse` skill. Wait for it to complete. Fix any issues before moving on.

### 6.7 `/finalise`

Invoke the `finalise` skill. Wait for it to complete.

### 6.8 `/wrap-up`

Invoke the `wrap-up` skill. Wait for it to complete.

---

## Done

Once `/wrap-up` completes, report back to the user with a brief summary:

- What was implemented
- Any notable decisions made
- PR URL (from `/wrap-up`)
- A table of which model was used for each step (planning, implementation, and each pipeline skill)
