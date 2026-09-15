---
name: steps
description: Guide the user through a task they must do themselves, one step at a time, revealing the next step only once they confirm the current one is done. Use when the user says "/steps", "walk me through it", "one step at a time", "guide me through this", "break this down for me", or otherwise asks to be led through manual work — dashboard setup, account or DNS configuration, a deploy, a migration they run by hand, a GitHub issue they're picking up. Accepts an optional issue number or task description.
---

Lead the user through a task **they** carry out, one step at a time. The point is focus: they only ever see the step in front of them, so never reveal, list, or hint at later steps.

## 1. Work out the task

In this order:

1. **Issue number in the arguments** (`123`, `#123`) — fetch it with `gh issue view <n> --comments` and use it as the task.
2. **Task description in the arguments** — use it.
3. **Current conversation** — the most common case. The task is usually whatever manual work was just discussed (a setup Claude can't do, a fix the user has to apply, a service to configure).
4. **Still unclear** — ask one short question: what task should be broken down. Don't guess between several candidates; name them as options.

## 2. Plan silently

Before the first step, build the full breakdown privately. Check the facts each step depends on — read the relevant code, config, or docs — so no step has to be retracted halfway through. Don't show the plan.

Good steps:

- **One action each** — something done in a single sitting without needing to scroll back.
- **Concrete** — exact commands, file paths, values, menu names, URLs. No "configure X appropriately".
- **Verifiable** — the user can tell when it's done.

## 3. Reveal one step

Open with a one-line headline of the task and a rough size, then the first step. Each step message:

```
**Step 2 of ~6 — <short title>**

<what to do, with exact commands/values>

**Done when:** <observable result>
```

Keep the count approximate (`~6`) — plans shift. Then stop and wait. End with nothing that previews what comes next.

## 4. Between steps

- **User confirms** — if the result is cheaply checkable (a file exists, a command succeeds, a config value is set), check it before moving on. Then reveal the next step.
- **User hits a problem or pastes output** — stay on the current step and help resolve it. Don't advance until it's done.
- **Something changes the plan** (unexpected state, user chose differently) — re-plan the remaining steps silently and carry on. Adjust the count.
- **User asks what's coming** — then, and only then, give a brief outline.
- **User asks you to do a step** — do it if you can, then continue.

Don't perform steps unprompted: the task is theirs.

## 5. Finish

After the last step is confirmed, say it's done in one line, plus anything that genuinely needs follow-up (e.g. a secret to rotate later). No recap of the steps.
