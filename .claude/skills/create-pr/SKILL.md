---
name: create-pr
description: Creates a pull request for the current branch with auto-generated concise title and description.
tools: Bash
---

# Create Pull Request

Create a PR for the current branch with minimal, focused descriptions.

## Workflow

### Step 1: Verify State

Run in parallel:
```bash
git status
git branch --show-current
git rev-parse --abbrev-ref HEAD@{upstream} 2>/dev/null || echo "no-upstream"
```

**Safety checks**:
- If on `main` or `master`: STOP and warn user
- If uncommitted changes: warn and ask to commit first

### Step 2: Ensure Remote Tracking

If no upstream:
```bash
git push -u origin HEAD
```

### Step 3: Analyze Changes

Get the diff against the base branch:
```bash
# Find base branch
git remote show origin | grep 'HEAD branch' | cut -d' ' -f5

# Get commit summary
git log origin/[base]..HEAD --oneline

# Get file changes
git diff origin/[base]...HEAD --stat
```

### Step 4: Generate PR Content

**Title Rules**:
- Max 72 characters
- Format: `[type]: [brief description]`
- Types: `feat`, `fix`, `refactor`, `docs`, `chore`, `perf`, `test`
- No periods at the end
- Sentence case after colon

**Body Rules**:
- Start with `## Summary`
- 2-4 bullet points max
- Focus on WHAT changed, not HOW
- NO "Generated with Claude" or similar signatures
- NO emojis unless in commit messages already
- Use the architect skill to include a mermaid diagram for big features or complex refactoring

### Step 5: Create PR

```bash
gh pr create \
  --title "[type]: [description]" \
  --body "$(cat <<'EOF'
## Summary

- [Main change]
- [Secondary changes if any]

EOF
)" \
  --draft \
  -a "@me"
```

### Step 6: Return Result

Output the PR URL.

## Output Format

```
Created PR: https://github.com/[org]/[repo]/pull/[number]

Title: [PR title]
Status: Draft
```

## Title Examples

Good:
- `feat: add user authentication`
- `fix: resolve race condition in order processing`
- `refactor: simplify payment validation logic`
- `chore: update dependencies`

Bad:
- `Add user authentication feature to the system` (too long, no type)
- `Fix bug` (too vague)
- `WIP: stuff` (not descriptive)

## Body Examples

Good:
```markdown
## Summary

- Add JWT-based authentication for API endpoints
- Implement login/logout endpoints
- Add middleware for protected routes

```

Bad:
```markdown
## Summary

This PR adds user authentication to the system. I implemented JWT tokens
for secure authentication. The changes include new middleware, endpoints,
and database models. Testing was done manually and all tests pass.

Generated with Claude Code
```

## Error Handling

| Error | Action |
|-------|--------|
| On protected branch | Warn and stop |
| No commits ahead | Inform user, nothing to PR |
| PR already exists | Return existing PR URL |
| Push failed | Show error, suggest resolution |

## Notes

- Always creates as draft PR
- Always assigns to self
- Auto-detects base branch
- If PR exists for branch, returns existing URL instead of creating new
