---
name: pipeline
description: Run the full post-implementation quality pipeline on a completed set of code changes — simplify, polish, review, test, cover, analyse, finalise — then wrap it into a pull request. Trigger whenever a change is written and the user wants it hardened and shipped — "run the pipeline", "run it through the pipeline", "harden these changes", "get this branch ready to merge", "QA this and open a PR", "polish and ship this", or after finishing an implementation when the next move is the quality gate. This spawns several subagents and ends in a PR — it's the heavyweight "whole quality pass" option. For a single step (just review, just run tests, just open a PR), use that individual skill instead.
effort: max
---

# Post-Implementation Pipeline

Take a completed, working set of code changes and drive it through the full quality pipeline to a clean, reviewed, tested PR. The change is assumed to be already written — by you, the user, or an upstream workflow. This skill refines and ships what exists; it does not write the feature.

## Inputs

Optional **brief** from the user or a calling workflow: the task description, a plan excerpt, or anything explaining what the change is for and why. Use it to ground the review / cover / finalise subagents so they judge against intent, not just the raw diff. With no brief, infer intent from the diff and commit messages.

A caller may also pass **upstream report rows** — e.g. models already used for planning or implementation — to fold into the final report. Treat these as optional; never assume they exist.

## Step 0: Preconditions and scope

1. **Branch guard.** `git branch --show-current`. If on `main`/`master`, stop: say so, suggest a branch name from the diff, and offer to create it first.
2. **Detect the change set.** `git status --short` and `git diff main..HEAD --stat` (fall back to `master`). Scope is the union of uncommitted changes and committed-but-unmerged commits. If there is neither, stop — nothing to pipeline.
3. **Classify the diff** (drives conditional steps): UI/UX (Blade/views, layouts, CSS/Tailwind, JS), backend only, or tests/docs only?

## Pre-flight enumeration

Before invoking anything, write out each step, resolving the tier labels to concrete model names for the models running right now. Doing this up front lets the user catch a mis-binding before any subagent spends tokens:

```
- 1 simplify → subagent: <yes/no>, model: <name or —>, condition: <met / skipped because …>
- 2 polish   → subagent: <yes/no>, model: <name or —>, condition: <met / skipped because …>
- 3 review   → subagent: <yes/no>, model: <name or —>, condition: <met / skipped because …>
- 4 test     → subagent: <yes/no>, model: <name or —>, condition: <met / skipped because …>
- 5 cover    → subagent: <yes/no>, model: <name or —>, condition: <met / skipped because …>
- 6 analyse  → subagent: <yes/no>, model: <name or —>, condition: <met / skipped because …>
- 7 finalise → subagent: <yes/no>, model: <name or —>, condition: <met / skipped because …>
- 8 wrap-up  → subagent: <yes/no>, model: <name or —>, condition: <met / skipped because …>
```

## Reading the table

The Subagent and Model columns map directly onto Agent-tool parameters. They exist so each step runs in isolation at a right-sized model — running a "Yes" row inline pollutes the main context, and dropping the model argument silently changes cost and quality. So:

- **Subagent = Yes** → a real Agent call with `subagent_type` and an explicit `model`. Don't inline it, even if the work feels small.
- **Subagent = No** → main thread, no Agent call.
- **Model = Best available** → the most capable model currently running. **Default tier** → the standard workhorse (not the cheapest). **—** → no model argument (only valid for "No" rows).

Pass each subagent the brief (or, absent one, the diff scope) so its prompt is grounded. If a step's skill isn't available in this environment, note it and skip — don't hand-roll a weaker substitute.

## Pipeline

| Step | Skill | Subagent | Model | Condition | On failure |
|------|-------|----------|-------|-----------|------------|
| 1 | `simplify` | Yes | Default tier | Only if the skill exists | Fix, then continue |
| 2 | `polish` | Yes | Best available | Only if the diff touched UI/UX | Fix, then continue |
| 3 | `review` | Yes | Best available | Always | Fix critical/major findings, then continue |
| 4 | `test` | Yes | Default tier | Always | Fix failures before moving on |
| 5 | `cover` | Yes | Best available | Always | Add missing tests, then continue |
| 6 | `analyse` | Yes | Default tier | If an `/analyse` skill exists, else any linters present; skip if none | Fix, then continue |
| 7 | `finalise` | Yes | Best available | Always | Fix, then continue |
| 8 | `wrap-up` | No | — | Always | Resolve blockers, then finish |

Run sequentially — start a step only if the previous one succeeded. `wrap-up` runs in the main thread because it commits, pushes, and talks to the user about the PR; that interaction doesn't belong in a detached subagent.

A quality step may propose undoing a change the user made deliberately (a subagent sees only the diff, not which edits were intentional). If a step wants to revert something the user clearly chose, restore it and keep going.

## Done

Once `wrap-up` finishes, report briefly: what the change does, any notable decisions, the PR URL, and a per-step model table. Fold in any upstream rows the caller supplied; otherwise the table is just the pipeline steps.

## Hand-off: stop modifying the branch

After the PR URL is reported, the user reviews it manually. Until they explicitly ask for more work on this change: make no further code changes, commits, amends, force-pushes, rebases, branch operations, or PR updates, and don't treat follow-up as implied. You may still answer questions and discuss the diff; if you spot something worth changing, mention it and wait. This overrides any keep-going bias.
