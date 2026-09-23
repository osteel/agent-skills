---
name: to-draft
description: Draft a new article for Yannick's tech blog as an outline, created directly in his Notion Posts database (Work / Projects / Yellow Raincoat / Writing / Tech / Posts). Use whenever he asks to draft, start, sketch or outline a blog post, article or tech post — even if he doesn't say "Notion". Also use it for "new post about X", "let's write up Y", "I want to publish something on Z", "I might blog this", or anything else implying a new piece of long-form writing. If in doubt and the request looks like blog writing, prefer this skill. The output is deliberately a skeleton he extends in his own voice, never a finished article.
effort: medium
---

# Draft a Notion Article

Yannick keeps blog posts in a Notion database and wants new drafts created there — not pasted into chat — with the right structure, specifics and properties.

## The output is an outline, not prose

**Never write finished prose.** He rewrites it in his own style, so writing it out wastes effort on both sides. The outline is the deliverable, not a step on the way to one.

Produce **one short declarative sentence per point, each as its own block**, under the article's headings.

- Complete short sentences that convey the point — not bullet fragments ("vocabulary sticky").
- Each block is a paragraph he'll write in place, so keep them in argument order.
- Mark the top as an outline, so a later revisit doesn't mistake it for a finished draft.
- Outline the intro and the closing too. Those get outlined, not written.

Two things decide whether it's actually useful:

**Keep the concrete specifics.** Names, numbers, the actual three places a bug surfaced, "about ten lines". These cost him real time to reconstruct and are what stop a piece becoming a think-piece.

**Flatten the phrasing.** Drop flourishes and rhetorical turns so the voice choices stay his — "That's not foresight" rather than "That's not foresight, that's clairvoyance". Where a phrase *is* the idea rather than a way of saying it, keep it.

## Target location

All drafts go into the **📝 Posts** database at `Work > Projects > Yellow Raincoat > Writing > Tech > Posts`.

- Data source: `collection://ef764a85-c1ab-488e-abac-858d914f9e1a`
- Database page: `https://app.notion.com/p/c481f41294ae4a289a9453230dfe7098`
- **Three databases in the workspace are named "Posts."** Use the data source id, never a search result.

| Property | Type | How to fill |
|---|---|---|
| `Name` | title | The article title. Inline markdown allowed. |
| `Status` | select: `drafting` / `published` / `idea` | `drafting` unless he's asked for a stub. |
| `Published on` | text | Leave empty for new drafts. |
| `Created` | created_time | Auto-set. Do not pass. |

## Workflow

### 1. Ask what he's starting with

Before anything else, ask whether he already has material — rough notes, bullets, a paragraph, a link, or just a topic. Free-form question in chat, not a structured selector; he may want to paste in a wall of notes.

> "Before I get going — do you have any notes or rough material for this one, or are we starting from scratch?"

- **Has material**: it's the source of truth for content and angle. Don't quietly reshape it.
- **Starting fresh**: use the topic as the seed. If the conversation itself is the material (he's just done the work being written about), say so and use it.
- **"Just write it"**: respect that, don't push for more.

Read what he gives you, internalise it, move on. Clarifications can wait.

### 2. Load the brand DNA

The blog belongs to Yellow Raincoat, so its brand DNA page applies. Fetch it with the Notion fetch tool:

- **Yellow Raincoat Brand DNA**: page ID `3e4cc17ef4b081a8aba1d1506192cdf8` (Work > Projects > Yellow Raincoat > Brand DNA)
- If the post is about PixelWatcher, also fetch the **PixelWatcher Brand DNA**: page ID `3e4cc17ef4b08116a22ef7d5bbd1bb52`.

Because the output is an outline, use it for substance, not voice — the prose is his:

- **Point of view**: the positions he holds shape the angle and which points the outline makes.
- **Naming and terminology rules** apply to every sentence in the outline: "Fractional CTO", a year rather than a fixed count of experience, PixelWatcher always with its category ("website change monitoring").
- **Heck-no claims** stay out: guarantees, unverifiable adjectives. Check the outline against the list before creating the page.
- Rules tagged **Observed** or **Decided** are binding; **Proposed** ones are preferences until he confirms them.
- Sentence-level voice rules (humour, rhythm, openings) are for him when he writes it up — don't imitate them in the outline.

### 3. Read two recent published posts

Sample 2–3 of the most recent posts with `Status = published` before outlining. Since you're not writing prose, you're mirroring **shape and register**, not sentence rhythm:

- How he opens — anecdote, tension, thesis?
- How many sections, and how the argument moves between them.
- Whether code appears at all (recent posts: rarely or never).
- What the closing does.
- Level of formality, and how much jargon gets defined in passing.

Recent posts publish to **yellowraincoat.co.uk** and are short, first-person and near-code-free. Older ones (osteel.me, dev.to, medium) are long-form tutorials and are **not** the model any more.

Note the patterns to yourself explicitly before outlining — easier to match a shape you've named than one you've skimmed.

### 4. Confirm scope, lightly

Default to `Status: drafting` and a finished length matched to recent posts (~800–1000 words), which sets how many points the outline carries. Ask only when it's genuinely open — one structured question covering target length and starting status (`AskUserQuestion` in Claude Code, the tappable-options tool in chat), never a series. He wants to move fast.

### 5. Create the outline in Notion

Go straight to the page — the outline *is* the artifact, so there's no separate chat-approval round trip. It's cheap to redirect once it exists.

- Four or five `##` sections is typical.
- Don't put the title as an H1 in the content; the `Name` property is the title.
- Blank line between points so each is its own block, editable in place.
- `icon: "📝"` to match the database.

### 6. Hand back the link

Give the URL, the section headings, and anything you deliberately left out with the reason — a tangent that would double the length, or a thread that belongs in its own post. A named omission is easy to overrule; a silent one isn't.

## Accuracy

These go out under his name, so a reader can check them.

- **Verify claims about what happened**, including ones he asserts in passing. If he misremembers a detail in your favour ("you suggested X"), say so and write the accurate version. Overstating what an AI did is exactly what a reader will catch.
- Naming vendors and tools is fine and expected — it's build-in-public. Keep it observational: if a library's design is reasonable for what it does, say so, so naming it doesn't read as a swipe.
- Watch for terms already used differently in an earlier post. "Harness" means the eval infrastructure in *What are Evals Anyway?*; don't reuse it for an agent's rules and checks. Prefer "context".

## Common pitfalls

- **Writing prose.** The single most likely failure. Full paragraphs get rewritten; the outline is the point.
- **Abstracting away the specifics.** An outline of general claims is useless — the concrete details are the part he can't cheaply regenerate.
- **Skipping "what do you have already?"** Wastes a pass and risks an angle he didn't want.
- **Skipping the brand DNA.** The naming rules and heck-no claims apply to outlines too, and a wrong term in the outline survives into the post.
- **Skipping the published-post read.** Tempting when the topic is exciting; it's what keeps the shape his.
- **Searching for the Posts database by name.** Three share it. Use the data source id.
- **Title as an H1 in the content.** Duplicates it against the `Name` property.
- **Pasting into chat instead of creating the page.** The deliverable is a Notion page.

## When NOT to use this skill

- Editing or updating an existing post — use `notion-update-page` (`update_content` for targeted edits, `replace_content` only when the whole structure changes).
- Non-blog writing (memos, emails, internal docs) — those don't belong in the Posts database.
- General questions about Notion or the workspace.

Don't set up test cases or evals for this skill. Its output is subjective by nature.
