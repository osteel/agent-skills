---
name: ui-rules
description: Apply high-signal visual design principles for UI work. Use by default when designing UI from scratch or when reviewing, refining, or polishing existing interfaces, including colour, layout, spacing, typography, depth, and components. Covers how the UI *looks* — for how it *works* (usability, accessibility, interaction flows), also invoke `ux-principles`.
user-invocable: false
---

# Basic UI Design Rules

A conservative, low-risk ruleset for producing coherent, professional user interfaces. Follow by default; deviate only with a clear, explicit reason.

---

## When to use this Skill

Use this skill automatically for **any task involving visual UI decisions**, including:

### Designing from scratch
- New web or app UIs
- Landing pages, dashboards, admin panels
- Component libraries and design systems
- Defining design tokens, themes, Tailwind or CSS scales

### Reviewing or improving existing designs
- “Make this look better / cleaner / more professional”
- UI polish or redesigns
- Visual audits of screens or components
- Colour, contrast, spacing, layout, depth, or typography changes

If the task involves how something **looks**, this skill applies.

---

## Operating mode

1. Decide the context first: light mode, dark mode, or both.
2. Treat every visual decision as intentional, not decorative.
3. Prefer consistency over novelty.
4. Design the system first, then the screen.

---

## Rules (apply in order)

### 1) Colour and contrast
- Avoid pure black (`#000`) and pure white (`#fff`). Use near-black and near-white.
- Tint neutrals slightly toward a single hue if using colour at all.
- Reserve the highest contrast for primary actions and key content.
- Keep structural elements (dividers, borders, shadows) low contrast. High-contrast structural elements fight for attention with the content they're meant to organise.
- Ensure palette colours differ in brightness, not just hue.
- Do not mix warm-tinted and cool-tinted neutrals.

---

### 2) Layout and alignment
- Everything must align with something else.
- Use a clear grid, shared edges, or common baselines.
- Prefer optical alignment when mathematical alignment looks wrong.
- Default to a 12-column grid when using columns.

---

### 3) Spacing and measurement
- Use a single spacing scale (e.g. multiples of 4 or 8).
- Avoid arbitrary values.
- Measure spacing between visible contrast edges.
- Outer padding must be greater than or equal to inner padding.

---

### 4) Containers, borders, and depth
- Borders must contrast with both the container and the background.
- Keep container brightness deltas constrained:
  - Dark UI: ≤ 12% brightness difference
  - Light UI: ≤ 7% brightness difference
- Avoid stacked hard divides (e.g. background change + border + divider).
- Do not use shadows in dark mode. Shadows simulate light falling on a surface — they look physically wrong on dark UIs that have no implied light source.
- Use only one depth technique across the UI.
- If using shadows, set blur to 2× the vertical offset. This ratio keeps shadows soft and natural; lower values produce hard-edged, artificial-looking shadows.

---

### 5) Typography
- Adjust letter spacing and line height based on text size:
  - Larger text → tighter
  - Smaller text → looser
- Body text must be at least 16px.
- Target ~70 characters per line.
- Use no more than two typefaces unless strictly systemised.

---

### 6) Components
- Button horizontal padding ≈ 2× vertical padding.
- Nested corner radii must be mathematically nested:
  - Inner radius = outer radius − gap
- Icons paired with text should be lower contrast than the text.

---

## How to apply this skill

### A) Designing from scratch
1. Choose light or dark mode.
2. Define neutral colours and any tint direction.
3. Define spacing, radius, and typography scales.
4. Choose one depth system.
5. Build components using only defined tokens.
6. Assemble screens without introducing new base values.

### B) Reviewing or refining
1. Identify primary actions and verify highest contrast.
2. Audit colours and brightness relationships.
3. Check alignment against a clear grid.
4. Normalise spacing to the chosen scale.
5. Remove mixed depth techniques and redundant divides.

---

## Response structure for UI tasks

For multi-screen or design-system work, structure output as:

1. **System decisions** — mode, palette approach, spacing and typography scales, depth system.
2. **Layout or component application** — how the rules are applied in practice.
3. **Concrete implementation notes** — specific token values or CSS/Tailwind guidance, if implementation is requested.

For single-component or small-scope tasks, skip the system section and go straight to what changed and why.

---

## Guiding principle

If a design decision cannot be justified using one of the rules above, it is probably unnecessary.