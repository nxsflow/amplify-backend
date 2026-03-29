# Amplify Backend (nxsflow fork)

Fork of `aws-amplify/amplify-backend`. See skill `branching-strategy` for the fork workflow.

## Project Overview

AWS Amplify Backend is a monorepo of ~30 npm packages under `packages/` that let developers define cloud backends (auth, data, storage, functions, AI) using TypeScript and AWS CDK constructs. The CLI tool is `ampx`.

## Monorepo Setup

- **Package manager**: npm workspaces (`packages/*`)
- **Build**: TypeScript composite projects — `npm run build` runs `tsc --build packages/* scripts`
- **No Turbo/Nx** — custom script orchestration via `tsx scripts/`
- **Node**: 18, 20, or 22

## Key Commands

```bash
npm run setup:local          # npm ci + build + link CLI
npm run build                # TypeScript composite build
npm run watch                # Watch mode
npm run test                 # All unit tests
npm run test:dir <path>      # Tests in specific dir/file
npm run lint                 # ESLint + Prettier (zero warnings)
npm run lint:fix             # Auto-fix
npm run check:api            # API surface validation (api-extractor)
npm run vend                 # Local npm proxy (Verdaccio) for testing
```

## Testing

- **Framework**: Node.js built-in `node:test` + `assert` (no Jest/Mocha)
- **Files**: `*.test.ts` alongside source
- **Run single**: `npm run test:dir packages/<pkg>/lib/<file>.test.ts`
- **Coverage**: 85% threshold (lines, functions, branches) — `.c8rc.json`
- **Integration**: `packages/integration-tests/src/test-in-memory/`
- **E2E**: `packages/integration-tests/src/test-e2e/` (requires AWS creds)

## Code Style

- **Linter**: ESLint with strict rules, zero warnings policy
- **Formatter**: Prettier (single quotes)
- **Naming**: files `snake_case`, folders `kebab-case`, types `PascalCase`, vars `camelCase`, constants `UPPER_CASE`, enum members `UPPER_CASE`
- **No** `util`/`helper` in filenames, no static methods, no `console`, no `.only()` in tests
- **Use** arrow functions (not declarations), `type` (not `interface`), JSDoc on public APIs
- **Spellcheck**: enabled with `.eslint_dictionary.json`

## TypeScript

- Target: ES2022, Module: Node16, strict mode
- Base config: `tsconfig.base.json` — packages extend it
- Experimental decorators enabled
- Output: `lib/` directories

## Git Hooks (Husky)

**Pre-commit**: updates deps metadata, tsconfig refs, validates package-lock, runs lint-staged
**Pre-push**: validates changesets exist (`changeset status --since main`), checks API surface

See skills `committing`, `changesets`, and `release-management` for details.

## Architecture

- **Composition via DI**: `@aws-amplify/plugin-types` defines interfaces
- **Core utilities**: `@aws-amplify/platform-core`
- **Output contracts**: `@aws-amplify/backend-output-schemas`
- **Feature verticals**: auth, data, storage, function, ai — each in own package
- **CLI**: `@aws-amplify/backend-cli` (`ampx` binary)
- **Process separation**: CLI → CDK deploy → synth (defineBackend) → CloudFormation

## Adding Packages

```bash
npm run new -- --template=<construct|empty-package> --name=<name>
```

Minimum: `package.json`, `tsconfig.json`, `api-extractor.json`, `typedoc.json`, `.npmignore`, `README.md`

## PR Process

- PR template at `.github/PULL_REQUEST_TEMPLATE.md`
- CODEOWNERS enforces reviews (API changes need `amplify-backend-api-approvers`)
- CI: health_checks.yml — matrix of Node versions × OS platforms
- Commit style: conventional commits (`feat:`, `fix:`, `chore:`) with PR number
