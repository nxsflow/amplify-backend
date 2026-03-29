---
name: create-pr
description: Create pull requests following conventions. Use when opening PRs, writing PR descriptions, or preparing changes for review. Determines target repo (aws-amplify upstream or nxsflow) and follows PR template.
---

# Create Pull Request

Create pull requests following engineering practices.

**Requires**: GitHub CLI (`gh`) authenticated and available.

## Prerequisites

Before creating a PR, ensure all changes are committed. If there are uncommitted changes, run the `committing` skill first.

```bash
git status --porcelain
```

## Step 1: Determine PR Target

Check which branch this work is based on to determine the PR target:

```bash
# Show current branch and its merge base
git branch --show-current
git merge-base --is-ancestor upstream HEAD && echo "Based on upstream" || echo "Based on main"
```

| Based on | PR target | Remote |
|----------|-----------|--------|
| `upstream` | `aws-amplify/amplify-backend` main | Open PR against upstream repo |
| `main` | `nxsflow/amplify-backend` main | Open PR against origin |

If unclear, use the AskUserQuestion tool to ask:
> "Should this PR target **upstream** (aws-amplify/amplify-backend) or **nxsflow** (your fork)?"

**For upstream PRs**: Verify no nxsflow-specific content is included (CLAUDE.md, .claude/, nxsflow packages).

## Step 2: Verify Branch State

```bash
BASE=$(gh repo view --json defaultBranchRef --jq '.defaultBranchRef.name')

git status
git log $BASE..HEAD --oneline
```

Ensure:
- All changes are committed
- Branch is up to date with remote
- Changes are rebased on the base branch if needed

## Step 3: Analyze Changes

```bash
git log $BASE..HEAD
git diff $BASE...HEAD
```

Understand the scope and purpose of all changes before writing the description.

## Step 4: Write the PR Description

Check for a PR template:

```bash
gh repo view --json pullRequestTemplates --jq '.pullRequestTemplates[0].body'
```

This repo has a PR template with these sections:
- **Problem** — issue number and what the PR fixes
- **Changes** — summary with critical/problematic parts highlighted
- **Validation** — how changes were validated (unit/integration/E2E tests)
- **Checklist** — functional test coverage, architecture docs, license

If the template exists, follow its structure. Otherwise:

```markdown
<brief description of what the PR does>

<why these changes are being made>

<alternative approaches considered, if any>

<any additional context reviewers need>
```

**Do NOT include**: "Test plan" checklists or redundant diff summaries.
**Do include**: Clear what/why, issue links, context not obvious from code.

## Step 5: Create the PR

### For nxsflow PRs (origin):

```bash
gh pr create --title "<type>(<scope>): <description>" --body "$(cat <<'EOF'
<description body here>
EOF
)"
```

### For upstream PRs (aws-amplify):

```bash
# Push to your fork first
git push origin <branch-name>

# Create PR against upstream
gh pr create --repo aws-amplify/amplify-backend \
  --title "<type>(<scope>): <description>" \
  --body "$(cat <<'EOF'
<description body here>
EOF
)"
```

**Title format** follows commit conventions:
- `feat(scope): Add new feature`
- `fix(scope): Fix the bug`
- `chore: Maintenance task`

## Issue References

| Syntax | Effect |
|--------|--------|
| `Fixes #1234` | Closes GitHub issue on merge |
| `Refs #1234` | Links without closing |

## Guidelines

- **One PR per feature/fix** — don't bundle unrelated changes
- **Keep PRs reviewable** — smaller PRs get faster, better reviews
- **Explain the why** — code shows what; description explains why
- **Mark WIP early** — use draft PRs for early feedback

## Editing Existing PRs

Use `gh api` instead of `gh pr edit`:

```bash
# Update PR description
gh api -X PATCH repos/{owner}/{repo}/pulls/PR_NUMBER -f body="$(cat <<'EOF'
Updated description here
EOF
)"

# Update PR title
gh api -X PATCH repos/{owner}/{repo}/pulls/PR_NUMBER -f title='new: Title here'
```
