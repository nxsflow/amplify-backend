---
name: changesets
description: How to create and manage changesets in amplify-backend — required for all changes affecting published packages
---

# Changesets in amplify-backend

## What Is a Changeset?

A changeset is a markdown file in `.changeset/` that declares which packages changed and what kind of version bump they need. Changesets are **required** for all changes that affect published packages. The pre-push hook enforces this.

## Creating a Changeset

```bash
npx changeset
```

This interactive prompt asks:
1. Which packages changed? (select from list)
2. What kind of bump? `major` / `minor` / `patch`
3. Description of the change (becomes the CHANGELOG entry)

It creates a file like `.changeset/shy-weeks-invite.md`:

```markdown
---
'@aws-amplify/backend-auth': minor
'@aws-amplify/plugin-types': patch
---

Add support for MFA configuration in auth construct.
```

## Version Bump Guidelines

| Change | Bump |
|--------|------|
| Bug fix, dependency update, security fix | `patch` |
| New feature, non-breaking addition | `minor` |
| Breaking API change | `major` |

## Key Config (.changeset/config.json)

- **baseBranch**: `main` — changesets are compared against this
- **updateInternalDependencies**: `patch` — when package B bumps, packages depending on B get auto-patched
- **commit**: `false` — changeset version bumps are NOT auto-committed
- **access**: `restricted` (individual packages override to `public` via `publishConfig`)

## How Changesets Flow Into Releases

1. You create a changeset file and commit it with your PR
2. PR merges to `main`
3. GitHub Actions (`changesets/action`) auto-creates/updates a **"Version Packages"** PR
4. That PR contains: bumped `package.json` versions, updated `CHANGELOG.md` files, deleted changeset files
5. When "Version Packages" PR is merged, packages are published to npm

## Multiple Changesets

Multiple changeset files can coexist. They accumulate and are all consumed when the Version Packages PR is created. Each PR should have its own changeset.

## Packages to Include

Only include packages whose **published code** changed. Don't add changesets for:
- Private packages (`@aws-amplify/integration-tests`, `@aws-amplify/eslint-plugin-amplify-backend-rules`)
- Changes that only affect tests, CI, or docs

## Dependency Cascading

If you change `@aws-amplify/plugin-types`, packages that depend on it (like `@aws-amplify/backend-auth`) will automatically receive a patch bump via `updateInternalDependencies: "patch"`. You do NOT need to manually list all downstream packages.

## Troubleshooting

**Pre-push fails with "No changesets found"**: Run `npx changeset` and commit the generated file.

**Not sure which packages changed**: Run `npx changeset status --since main` to see what changesets expects.

**Changeset for the fork only (not upstream)**: If your change is nxsflow-specific and won't go to upstream, you still need a changeset since the fork publishes its own packages. Use the same process.
