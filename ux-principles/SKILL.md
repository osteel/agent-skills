---
name: ux-principles
description: Applies strict usability, clarity, and accessibility rules when creating or improving interfaces, flows, layouts, and UX copy, including when critiquing an existing flow. Shapes the result; never produces a standalone UX audit or report. Covers how the UI *works* — for how it *looks* (colour, spacing, typography, depth), also invoke `ui-rules`.
user-invocable: false
---

# Basic UX Principles

When **creating or improving** interfaces, flows, layouts, or UX copy, ensure the result is immediately understandable, decision-light, and usable by an average or below-average user **without instructions**.

This skill is **generative and prescriptive**, not analytical.
It exists to **shape outputs at creation time**, not to produce UX audits or reports.

---

## Operating assumptions (hard constraints)

Design **as if all of the following are true**:

- Users do not read carefully
- Users do not explore or experiment
- Users do not infer intent
- Users do not want to make decisions
- Any uncertainty increases cognitive load
- If something needs explaining, the design has already failed

---

## Design posture

While generating designs or interfaces, the agent must:

- Bias toward **obviousness over elegance**
- Prefer **effectiveness over conceptual understanding**
- Remove choices whenever a sensible default exists
- Follow conventions unless breaking them adds clear usability gains
- Treat every screen as a **billboard**, not a document

---

## Generative rules by domain

Read `references/ux-domains.md` for the full rule set (10 domains: core usability, clarity, content/copy, visual layout, navigation, home page, search, forms, mobile, accessibility).

---

## Output expectations

When this skill is active, the agent should:

- **Silently apply** these rules while generating designs or interfaces
- Make defaults explicit and remove unnecessary choices
- Avoid adding explanatory text where design can do the work
- Only surface rationale if the user explicitly asks “why”

---

## Success criteria

The output is successful if:

- A first-time user can complete the task without thinking
- The interface explains itself without instructions
- No unnecessary decisions are left to the user
- Nothing requires guessing, interpretation, or exploration

If these conditions cannot be met, the agent must simplify the design further rather than explain it.