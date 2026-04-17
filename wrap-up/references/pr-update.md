# PR Body Format

```
## Summary
- <bullet points covering what changed and why>

## Test plan
- <bulleted checklist of what to verify>
```

Do not add "Generated with Claude Code" or any mention of Claude/AI.

Create immediately (no confirmation needed):
```
gh pr create --title "..." --body "$(cat <<'EOF'
...
EOF
)"
```

# Ancestor check (for existing PR)

Determines whether an existing PR belongs to the current branch's history or to a reused branch name:

```bash
git merge-base --is-ancestor <headRefOid> HEAD && echo yes || echo no
```

- `yes` → PR belongs to this branch → update it (step 11)
- `no` → branch name reused, different work → create new PR (step 10)
