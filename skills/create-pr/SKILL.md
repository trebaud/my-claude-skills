---
name: create-pr
description: Create a pull request from the current branch using project conventions. Use when the user asks to create a PR, open a pull request, submit changes for review, push and create a PR, or says /create-pr. Keywords: pull request, PR, merge request, code review, gh pr create.
license: Apache-2.0
metadata:
  version: 1.0.0
  author: moose
allowed-tools: Bash Read Glob Grep
---

# Create Pull Request

Create a pull request from the current branch using project conventions.

## When to Use

- User asks to create, open, or submit a pull request
- User says "create PR", "open PR", "submit for review", or "push and create PR"
- User invokes `/create-pr`

## Instructions

### Step 1: Gather Context

Run these commands in parallel to understand the current state:

```bash
# Get current branch name
git branch --show-current

# Check status (staged, unstaged, untracked)
git status -sb

# Detect the default branch (master or main)
git branch | grep -E 'main|master'
```

Store the detected default branch as `{base}` and use it for all subsequent commands.

```bash
# Get commits on this branch not on the default branch
git log {base}..HEAD --oneline

# Get diff summary against the default branch
git diff {base}...HEAD --stat
```

If the current branch is the default branch (`{base}`), stop and tell the user they need to be on a feature branch.

### Step 2: Check for Uncommitted Changes

If there are staged or unstaged changes, ask the user whether to:
- Commit them before creating the PR
- Stash them and proceed
- Abort

Do not silently commit or discard changes.

### Step 3: Determine PR Title

Generate a concise PR title using conventional commit format based on the branch name prefix:

| Branch Prefix | PR Title Format |
|---|---|
| `feat/` | `feat(<scope>): <description>` |
| `fix/` | `fix(<scope>): <description>` |
| `refactor/` | `refactor(<scope>): <description>` |
| `chore/` | `chore(<scope>): <description>` |
| `docs/` | `docs(<scope>): <description>` |
| `perf/` | `perf(<scope>): <description>` |
| `test/` | `test(<scope>): <description>` |
| `hotfix/` | `fix(<scope>): <description>` |
| `ci/` | `ci(<scope>): <description>` |
| `revert/` | `revert(<scope>): <description>` |
| `security/` | `fix(<scope>): <description>` |
| `deps/` | `chore(deps): <description>` |

- Infer the **scope** from the changed files (e.g., `api`, `auth`, `ui`, `docs`).
- Infer the **description** from the commit messages and diff.
- If the branch has no recognizable prefix, derive the title directly from the commits and changes.
- Keep the title under 70 characters.

### Step 4: Generate PR Body

First, check for a repo-specific PR template:

```bash
# Check common template locations
cat .github/pull_request_template.md 2>/dev/null || \
cat .github/PULL_REQUEST_TEMPLATE.md 2>/dev/null || \
cat .github/PULL_REQUEST_TEMPLATE/pull_request_template.md 2>/dev/null || \
echo "NO_TEMPLATE"
```

**If a template exists**: Fill it in using information from the commits and diff. Follow the template's structure exactly.

**If no template exists**: Generate the body using this format:

```markdown
## Summary
<!-- 1-3 concise bullet points describing what changed and why -->

## Changes
<!-- Bulleted list of specific changes, grouped by area if needed -->

```

**Writing style for the PR body**:
- Be concise and information-dense. No filler, no fluff, no generic statements.
- State what changed and why — skip obvious context the reviewer can see in the diff.
- Every sentence should carry new information. If a bullet point could be deleted without losing understanding, delete it.
- Use precise technical language. Say "adds retry with exponential backoff to S3 uploads" not "improves reliability of file uploads".
- Don't pad with "this PR does..." or "the purpose of this change is..." — just state the change directly.

To write an accurate body, analyze the actual code changes:

```bash
git diff {base}...HEAD
```

**IMPORTANT**: Do NOT include any mention of AI, AI agents, or automated generation in the PR body. No "Generated with..." or "Co-authored by..." footers. The PR should read as if written by the developer directly.

### Step 5: Push and Create PR

```bash
# Push branch with upstream tracking
git push -u origin $(git branch --show-current)

# Create the PR targeting the default branch
gh pr create --base {base} -a "@me" --title "{title}" --body "$(cat <<'EOF'
{body}
EOF
)"
```

If `gh` is not installed or not authenticated, tell the user and provide the manual steps.

### Step 6: Return Result

Output the PR URL so the user can view it. If the PR was created successfully, also show:
- PR number
- Target branch
- Link to the PR

### Error Handling

- **No remote**: Tell the user to add a remote first.
- **Branch already has a PR**: Use `gh pr view` to show the existing PR and ask if they want to update it.
- **Push rejected**: Suggest `git pull --rebase` and retry.
- **gh auth failure**: Suggest `gh auth login`.
- **Merge conflicts with base**: Warn the user and suggest rebasing before creating the PR.
