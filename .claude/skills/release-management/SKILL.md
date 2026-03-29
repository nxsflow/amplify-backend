---
name: release-management
description: How releases work in amplify-backend — Version Packages PRs, publishing, snapshots, and deprecation
---

# Release Management

## Release Flow (Production)

```
Feature PR merged → changesets consumed → "Version Packages" PR auto-created
    → PR reviewed & merged → CI tests pass → npm publish + git tags
```

### Step-by-step:

1. **Feature PR merges to `main`** with changeset files in `.changeset/`
2. **`update_package_versions` job** runs (health_checks.yml):
   - Detects this is NOT a "Version Packages" commit
   - `changesets/action` creates or updates PR titled **"Version Packages"**
   - PR contains: bumped versions, updated CHANGELOGs, deleted changeset files
3. **Team reviews** the Version Packages PR (version bumps, changelog entries)
4. **PR is merged** to `main`
5. **`publish_package_versions` job** runs:
   - Detects this IS a "Version Packages" commit (author: `github-actions[bot]`)
   - Waits for ALL test suites to pass (unit, e2e, coverage)
   - Requires `release` environment approval
   - Runs `npm run publish` → `changesets publish`
   - Creates git tags for each published package
   - Force-pushes `main` to `hotfix` branch (keeps it in sync)

### Detection Logic

A "Version Packages" commit is identified by:
- Exactly 1 commit in the push
- Author is `github-actions[bot]`
- Message contains "Version Packages"

## Snapshot Releases

For testing changes before merging to main:

- **Branch naming**: push to `snapshot/<name>`
- **Workflow**: `snapshot_release.yml` triggers automatically
- **Version format**: `1.9.3-test.123`
- **No git tags** created
- **Published to npm** with `--tag test`

```bash
npm run publish:snapshot    # Manual snapshot publish
```

## Local Testing (Verdaccio)

Test packages locally without publishing to npm:

```bash
npm run vend                # Start Verdaccio proxy + publish locally
npm run start:npm-proxy     # Start proxy only
npm run publish:local       # Publish to local proxy
npm run stop:npm-proxy      # Stop proxy
```

## Publish Scripts

| Script | Registry | Git Tags | Use Case |
|--------|----------|----------|----------|
| `npm run publish` | npm | Yes | Production release |
| `npm run publish:snapshot` | npm | No | Feature testing |
| `npm run publish:local` | Verdaccio | Yes | Local testing |

## Deprecation & Restoration

Manual GitHub workflow dispatches:

- **`deprecate_release.yml`**: Marks versions as deprecated on npm
  - Inputs: deprecation message, packages to skip, starting commit
  - Can dry-run against local proxy first
- **`restore_release.yml`**: Removes deprecation from versions

## Hotfix Process

- `hotfix` branch is kept in sync with `main` after each publish
- Emergency fixes can branch from `hotfix`
- Health checks also run on PRs to `hotfix`

## Key Files

- `.changeset/config.json` — changeset configuration
- `scripts/publish.ts` — production publish entry
- `scripts/publish_runner.ts` — publish logic (handles snapshots, local, tags)
- `scripts/version_runner.ts` — version bump logic
- `scripts/components/is_version_packages_commit.ts` — commit detection
