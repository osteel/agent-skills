---
name: grill-me
description: Grill the user relentlessly about a plan, decision, or idea. Use when the user wants to stress-test their thinking, or uses any 'grill' trigger phrases.
effort: max
---

Interview the user relentlessly until you reach a shared understanding. Map this as a **design tree**: every decision branches into the decisions that hang off it.

Work the tree in **rounds**. The **frontier** is every decision whose prerequisites are already settled — the questions you can put now without guessing at answers you haven't heard yet. A round is the whole frontier.

## Asking

Put every question through the harness's interactive prompt — the tool that presents selectable options and collects the answers — rather than writing questions out in prose. **In Claude Code that tool is `AskUserQuestion`.** Such tools show questions to the user one at a time, so a round arrives as a sequence even when sent as a single call. If the harness has no equivalent, fall back to numbered questions in one message per round.

Before the prompt, write a short plain-language preamble: a couple of sentences on what is at stake and why this is a genuine decision. Supporting detail only if the user asks for it. Then the questions.

These tools cap how many questions one prompt can carry — four in Claude Code. If the frontier is larger, send one full prompt and then immediately send the rest before acting on anything. Never drop a question or push it to a later round to fit the limit.

Each question needs:

* Plain language. What the choice is about, not how it works underneath.
* A very short label, where the tool takes one — 12 characters in Claude Code.
* Two to four options, your recommended one first, with `(Recommended)` ending its label.
* A line per option saying what choosing it actually means and what it costs, in plain terms.
* Multiple selection enabled where the options are not mutually exclusive.

Do not add an option for discussing the question further when the tool already gives the user a free-text escape — Claude Code supplies *Other* automatically.

Keep technical detail out of the questions. Name systems and constraints only where the user cannot choose without them. If a choice cannot be explained without internals, that is a sign you are asking about mechanism when you should be asking about intent — go up a level and ask what they are trying to achieve.

## Between rounds

Each round the user answers reshapes the tree — settled decisions push the frontier outward and unblock questions that depended on them. Recompute the frontier and ask the next round. A question whose answer depends on another question still open in this round belongs to a _later_ round, not this one.

**Establish the facts before the round, not after it.** A question resting on an unchecked premise spends the user's attention on a decision they will have to make again. So: before you put a round, list the facts each question assumes, and check every one you can — read the code rather than recalling it, and look up the prevailing practice rather than asserting it. Two premises in particular are never safe from memory: what this codebase already does (one folder's shape is not a convention), and what is normal in the wider industry. If you catch yourself writing "the codebase already does X" or "this is standard", go and check.

A recommendation carries the same burden. Retracting one after the user asks "is that common practice?" means the round was guesswork, and every answer downstream of it is now suspect.

Finding _facts_ is your job, never the user's. When a frontier question needs a fact from the environment (filesystem, tools, etc.), dispatch a sub-agent to find it — don't ask the user for anything you could look up yourself. Don't block on it: a running exploration is an unsettled prerequisite, so only the questions downstream of it wait for the sub-agent to report — ask the rest of the frontier now. The _decisions_ are the user's — put each to them and wait.

The session is done when the frontier is empty: every branch of the design tree visited, nothing left silently assumed. Do not act on it until the user confirms you have reached a shared understanding.
