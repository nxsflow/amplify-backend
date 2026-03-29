---
name: branching-strategy
description: Use when starting any new feature or fix — determines whether to branch from upstream (for aws-amplify contribution) or main (for nxsflow-only). MUST ask the user which target before creating a branch.
---

# Branching Strategy (nxsflow fork)

This repo is a fork of `aws-amplify/amplify-backend`, maintained as `nxsflow/amplify-backend`.

> **When this skill activates:** Before creating any feature or fix branch, you MUST ask the user whether this work targets **upstream** (contribution to aws-amplify) or **nxsflow-only**. Use the AskUserQuestion tool to get this answer. The response determines which branch to base from and what constraints apply.

## Branches

| Branch | Purpose | Contains nxsflow features? |
|--------|---------|---------------------------|
| `upstream` | Exact mirror of `aws-amplify/amplify-backend` main | NO — must stay clean |
| `main` | nxsflow's primary branch with custom features | YES |

## Sync Flow

```
aws-amplify/amplify-backend main
        ↓ (fetch + merge)
    upstream branch
        ↓ (merge commit)
    main branch (preserves nxsflow features)
```

### Syncing upstream changes:

```bash
# Add upstream remote if not already set
git remote add aws-amplify https://github.com/aws-amplify/amplify-backend.git

# Fetch and update upstream branch
git fetch aws-amplify main
git checkout upstream
git merge aws-amplify/main --ff-only
git push origin upstream

# Merge into main (with merge commit, never rebase)
git checkout main
git merge upstream
# Resolve any conflicts, preserving nxsflow features
git push origin main
```

**Always use merge commits** when merging `upstream` → `main`. Never rebase `main` — it's a published branch and rebase would destroy nxsflow feature history.

## Feature Branch Rules

| Target | Branch from | Merge to | Notes |
|--------|------------|----------|-------|
| Upstream contribution (PR to aws-amplify) | `upstream` | `upstream` (then PR to aws-amplify) | Must have zero nxsflow content |
| nxsflow-only feature | `main` | `main` | Can use any nxsflow-specific code |

### For upstream contributions:

```bash
git checkout -b feat/my-upstream-feature upstream
# ... make changes ...
# Rebase on upstream before opening PR to aws-amplify
git rebase upstream
git push origin feat/my-upstream-feature
# Open PR against aws-amplify/amplify-backend main
```

### For nxsflow features:

```bash
git checkout -b feat/my-nxsflow-feature main
# ... make changes ...
git push origin feat/my-nxsflow-feature
# Open PR against nxsflow/amplify-backend main
```

## What must NOT be on `upstream`

- `CLAUDE.md`
- `.claude/` directory
- Any nxsflow-specific packages, configs, or modifications
- Any files that don't exist in `aws-amplify/amplify-backend`

## Reducing Merge Pain

- Keep nxsflow features in **new files/packages** where possible
- Minimize modifications to existing upstream files
- Sync frequently (weekly or after major upstream releases)
- Tag sync points: `git tag sync/YYYY-MM-DD` after successful merges to `main`
