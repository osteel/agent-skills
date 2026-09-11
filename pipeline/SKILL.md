---
name: pipeline
description: Run the full post-implementation quality pipeline on a completed set of code changes — simplify, polish, review, analyse, cover, qa — then wrap it into a pull request and monitor CI until it's green. Trigger whenever a change is written and the user wants it hardened and shipped — "run the pipeline", "run it through the pipeline", "harden these changes", "get this branch ready to merge", "QA this and open a PR", "polish and ship this", or after finishing an implementation when the next move is the quality gate. This spawns several subagents and ends in a PR with green CI — it's the heavyweight "whole quality pass" option. For a single step (just review, just run tests, just open a PR), use that individual skill instead.
effort: max
---

# Post-Implementation Pipeline

Take a completed, working set of code changes and drive it through the full quality pipeline to a clean, reviewed, tested PR. The change is assumed to be already written — by you, the user, or an upstream workflow. This skill refines and ships what exists; it does not write the feature.

## Inputs

Optional **brief** from the user or a calling workflow: the task description, a plan excerpt, or anything explaining what the change is for and why. Use it to ground the review / cover subagents so they judge against intent, not just the raw diff. With no brief, infer intent from the diff and commit messages.

A caller may also pass **upstream report rows** — e.g. models already used for planning or implementation — to fold into the final report. Treat these as optional; never assume they exist.

## Step 0: Preconditions and scope

1. **Branch guard.** `git branch --show-current`. If on `main`/`master`, stop: say so, suggest a branch name from the diff, and offer to create it first.
2. **Detect the change set.** `git status --short` and `git diff main..HEAD --stat` (fall back to `master`). Scope is the union of uncommitted changes and committed-but-unmerged commits. If there is neither, stop — nothing to pipeline.
3. **Classify the diff** (drives conditional steps): UI/UX (Blade/views, layouts, CSS/Tailwind, JS), backend only, or tests/docs only?

## Pre-flight enumeration

Before invoking anything, write out each step, resolving the Opus/Sonnet labels to concrete model names for the models running right now. Doing this up front lets the user catch a mis-binding before any subagent spends tokens:

```
- 1 simplify → subagent: <yes/no>, model: <name or —>, condition: <met / skipped because …>
- 2 polish   → subagent: <yes/no>, model: <name or —>, condition: <met / skipped because …>
- 3 review   → subagent: <yes/no>, model: <name or —>, condition: <met / skipped because …>
- 4 analyse  → subagent: <yes/no>, model: <name or —>, condition: <met / skipped because …>
- 5 cover    → subagent: <yes/no>, model: <name or —>, condition: <met / skipped because …>
- 6 qa       → subagent: <yes/no>, model: <name or —>, condition: <met / skipped because …>
- 7 wrap-up  → subagent: <yes/no>, model: <name or —>, condition: <met / skipped because …>
- 8 monitor  → subagent: <yes/no>, model: <name or —>, condition: <met / skipped because …>
- 9 show-me  → subagent: no, model: —, condition: <artifact: UI/UX met / skipped because …; explanation: always>
```

## Reading the table

The Subagent and Model columns map directly onto Agent-tool parameters. They exist so each step runs in isolation at a right-sized model — running a "Yes" row inline pollutes the main context, and dropping the model argument silently changes cost and quality. So:

- **Subagent = Yes** → a real Agent call with `subagent_type` and an explicit `model`. Don't inline it, even if the work feels small.
- **Subagent = No** → main thread, no Agent call.
- **Model = Latest Opus model** → the latest Opus model (the most capable). **Latest Sonnet model** → the latest Sonnet model (the standard workhorse). **—** → no model argument (only valid for "No" rows).

Isolation isn't free, so it isn't uniform. A subagent gets a clean, undistracted read of the diff — worth a lot for `review` and `cover`, where a fresh pair of eyes is the whole point. `analyse` gets nothing from fresh eyes: it runs the project's own linters and formatters and reports what they say, so it runs in the main thread rather than paying for a subagent round-trip on what amounts to a shell command.

Pass each subagent the brief (or, absent one, the diff scope) so its prompt is grounded. If a step's skill isn't available in this environment, note it and skip — don't hand-roll a weaker substitute.

**`wrap-up` owns the full-suite run — make every step aware of it.** Put a line to that effect in each subagent prompt, and hold to it yourself on the inline steps. Left alone, the member skills run the suite themselves — `review` after applying fixes, `cover` after adding tests — which on a slow suite means repeat full runs to reach the same verdict `wrap-up` reaches once. Targeted runs are still fine and expected: verify the tests you just wrote, or the files you just touched. What no step before `wrap-up` should do is invoke the `test` skill for a full-suite pass.

**Tell `wrap-up` not to re-run for confirmation.** Say so explicitly: its one guarded suite run is the gate, and a second "just to be sure" pass proves nothing the first didn't. Same for the other gates — one clean run each is the answer.

**No step may report a result it did not observe.** Put this in every subagent prompt and hold to it inline. A step that fans out to its own review agents reports what those agents actually sent back; if one never replied, that is the finding. Attributing a conclusion to agents that never reported, or counting votes that were never cast, turns a single unreviewed opinion into false consensus and the pipeline's whole value is that the opinions are real. The same applies to gates: quote the output you read.

**Batch fixes, then gate once.** When a step returns findings, apply *all* of them before running any gate, rather than gating after each fix. Linters, type-checkers and test suites are usually the slowest thing in the pipeline, and re-running the set after every individual fix is the easiest way to turn a ten-minute pipeline into an hour. Targeted verification of the file you just edited is cheap and encouraged; the full gate belongs at the end of the batch. If one fix might invalidate another, order the batch so the risky one lands first — don't gate in between to find out.

## Pipeline

| Step | Skill | Subagent | Model | Condition | On failure |
|------|-------|----------|-------|-----------|------------|
| 1 | `simplify` | Yes | Latest Opus model | Always | Fix, then continue |
| 2 | `polish` | Yes | Latest Opus model | Only if the diff touched UI/UX | Fix, then continue |
| 3 | `review` | Yes | Latest Opus model | Always | Fix critical/major findings, then continue |
| 4 | `analyse` | No | — | The project's own static-analysis skill if it has one, else any linters present; skip if neither | Fix, then continue |
| 5 | `cover` | Yes | Latest Sonnet model | Always | Add missing tests, then continue |
| 6 | `qa` | Yes | Latest Opus model | Only if the diff touched rendered output or client-side behaviour, AND browser automation is available, AND the app can be run locally | Fix, re-verify that path, then continue |
| 7 | `wrap-up` | No | — | Always | Resolve blockers, then finish |
| 8 | `monitor` | No | — | Only if wrap-up opened or updated a PR | Per the skill: fix and push; stop and report if unfixable |

Run sequentially — start a step only if the previous one succeeded.

**There is one test gate, and it lives inside `wrap-up`.** Every step before it verifies narrowly — the tests covering the files it touched, the linters over the diff — and `wrap-up`'s own guarded run is where the change has to pass before anything is committed. Don't add a test step in front of it: on a green branch that run has nothing to find that the targeted runs missed, and it doubles the slowest thing in the pipeline.

**The gate is the project's own default test command, not a forced full run.** Where a project uses test impact analysis (Pest's Tia, Jest `--onlyChanged`, `go test` caching, and friends), its default command already replays what your changes can't have affected, and its CI is the full-suite backstop that decides merge-green. Overriding that to force every test — `--ci`, `--no-tia`, `--runInBand` — is duplicated work on the slowest thing in the loop, and needs a concrete reason: a replayed result you don't believe, or a changed input the tool's graph cannot see. Check the project's own conventions before assuming a full run is the gate, and pass that instruction to every step.

`qa` sits immediately before `wrap-up` for two reasons. Anything it finds gets fixed while the branch is still private, rather than as follow-up commits on an open PR — and because the gate now lives in `wrap-up`, a QA fix lands *before* the full suite runs rather than after it. QA's job is to find what tests structurally can't reach, so don't let it re-tread ground the targeted runs already cover.

It gets a subagent at the top model tier despite looking mechanical. Driving a browser is easy; deciding what is worth exercising, reading a screenshot correctly, and noticing that something is subtly wrong rather than absent is not. A misjudged check here produces a *false green*, which is worse than skipping the step — so this is judgement work closer to `review` than to `test`. Expect it to skip often: most changes have no rendered surface, and the skill is written to say so and stop rather than perform QA theatre. It also cannot proceed without working browser automation, so treat an unavailable browser as a skip with a stated reason, not a failure.

`analyse` is a project-scoped skill in most repos — it belongs to the project because it wraps that project's linters and formatters. Look for it in the project's own `.claude/skills/`, not just the global library, and fall back to running the linters directly if there's no skill wrapping them. It is the linters only — it does not run the test suite.

`wrap-up` runs in the main thread because it commits, pushes, and talks to the user about the PR; that interaction doesn't belong in a detached subagent. `monitor` also runs in the main thread: drive it via `/loop` (dynamic pacing) so it re-checks CI across wakeups until every check is green. Its CI-fix commits are part of the pipeline's mandate, not post-PR follow-up.

A quality step may propose undoing a change the user made deliberately — a subagent sees the diff, not which edits were intentional. If a step wants to revert something the user clearly chose, restore it and keep going. Step 4 runs in the main thread partly for this reason: it can still see the conversation that would tell it a choice was deliberate.

## Done

Once `wrap-up` finishes, report briefly: what the change does, any notable decisions, the PR URL, and a per-step model table. Fold in any upstream rows the caller supplied; otherwise the table is just the pipeline steps. Then start `monitor` — when it stops, append its summary (fixes pushed, final CI state).

Finally, invoke `show-me` (Skill tool, main thread) to present the finished work visually — the presentation layer of this report, not a new quality gate:

- **Always** consider one `show-me` visual to carry the explanation when the change has a shape worth drawing (a flow, a refactor's file movement, a state transition). Prose-only is fine for a change that has none — don't manufacture a diagram.
- **When the diff touched UI/UX** (the same Step 0.3 classification that gated `polish` and `qa`), have `show-me` publish a focused HTML **Artifact** walking through the visual change, and give the user the link.

This step is read-only with respect to the branch — publishing an Artifact and writing an explanation are not branch modifications, so it does not breach the hand-off below. Run it after `monitor` stops; skip the Artifact (not the explanation) when the diff had no rendered surface.

## Run once, at the end

The pipeline runs once per change, not once per revision of it. When the user is reviewing the work and asks for a change — during manual review of the diff, or on an open PR — make that change and stop. Don't re-run the pipeline, or any subset of its steps, on each revision. The pipeline runs again only when the user says so, at the end of the review, the same way committing waits for their word.

## Hand-off: stop modifying the branch

Once `monitor` stops (or is skipped), the user reviews the PR manually. Until they explicitly ask for more work on this change: make no further code changes, commits, amends, force-pushes, rebases, branch operations, or PR updates, and don't treat follow-up as implied. You may still answer questions and discuss the diff; if you spot something worth changing, mention it and wait. This overrides any keep-going bias.
