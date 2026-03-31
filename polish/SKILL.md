---
name: polish
description: Polish UI and UX — use when the user says things like "make this look better", "the UI feels rough", "improve the design", "this needs some polish", or "clean up the UI". Applies ui-rules and ux-principles to review and improve a specific view or component.
effort: max
---

# Polish UI and UX

**Task:** `$ARGUMENTS`

Focus on the UI referenced in `$ARGUMENTS`. If not specified, focus on the most recently modified view or component.

## Instructions

Invoke the `ui-rules` and `ux-principles` skills using the Skill tool before doing any work. These provide the design rules and UX principles that govern all recommendations.

Present all recommendations at once and wait for the user to approve before implementing anything. Once approved, implement all agreed changes in one pass.

After implementing, check whether tests exist for the affected components. If they do, run them to catch any regressions. If none exist, skip this step.
