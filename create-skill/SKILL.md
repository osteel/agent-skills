---
name: create-skill
description: Create a new global skill and install it across all agent harnesses. Use this over `skill-creator` directly when the skill should be installed globally (available in all harnesses). Use when the user wants to build a new skill, add a skill to the global skills library, or says things like "create a skill for X", "make a skill that does X", "add a skill to my library". Triggers the full skill-creator workflow, then moves the result into ~/.agents/skills/ and symlinks it everywhere.
effort: high
---

# Create Global Skill

This skill wraps `skill-creator` with a post-creation step: once the skill is built and the user is happy with it, move it into the global skills library and install it.

## Step 1: Run the skill-creator workflow

Follow the full `skill-creator` skill from start to finish. Do everything it says — draft, test, evaluate, iterate — until the user is satisfied.

One thing to clarify upfront with the user: **is this skill meant to be global?** A global skill lives in `~/.agents/skills/` and gets symlinked into all agent harnesses. If they say yes (or it's implied), proceed with the installation steps below after the skill is done. If they say no, just finish the skill-creator workflow and leave the skill wherever it was created.

## Step 2: Move to the global skills library

Once the skill is finished and the user confirms it's ready:

1. Check `~/.agents/skills/<skill-name>` doesn't already exist before moving (to avoid clobbering an existing skill). Then move the skill folder:
   ```bash
   mv <skill-path> ~/.agents/skills/<skill-name>
   ```

2. Run the sync script to create symlinks in all agent harnesses:
   ```bash
   bash ~/.agents/skills/sync.sh
   ```
   `sync.sh` iterates over every subdirectory in `~/.agents/skills/` and symlinks it into each configured provider's skills directory (whichever ones exist on this machine). If it fails, check that the target directories are writable and that the skill folder name isn't already symlinked.

3. Confirm to the user which symlinks were created.

That's it — the skill is now available globally.
