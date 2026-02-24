# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

A GitHub Action that deploys applications to Clever Cloud via the Clever Cloud CLI. It runs as a Docker container (`ghcr.io/47ng/actions-clever-cloud`). The action authenticates with Clever Cloud, optionally links an app, sets environment variables, then runs `clever deploy`.

## Commands

```bash
pnpm test          # Run tests with Vitest (includes type checking)
pnpm build         # Bundle src/main.ts → dist/main.js via tsdown
pnpm knip          # Dead code detection
```

There is no lint script in package.json. ESLint is configured but run ad-hoc.

To run a single test file: `pnpm vitest run src/action.test.ts`
To run a single test by name: `pnpm vitest run -t "test name pattern"`

## Architecture

**Entry flow**: `src/main.ts` → `processArguments()` → `run(args)`

- `src/main.ts` — Entry point. Fixes git dubious ownership, parses args, calls `run()`.
- `src/arguments.ts` — Reads GitHub Action inputs via `@actions/core` and env vars (`CLEVER_TOKEN`, `CLEVER_SECRET`). Returns a typed `Arguments` object.
- `src/action.ts` — Core logic: checks for shallow clone, authenticates CLI, links app (with unlink-first for idempotency), sets env vars, deploys with optional timeout via `Promise.race()`.

**Build**: tsdown bundles to `dist/main.js`. Multi-stage Dockerfile compiles in builder, copies dist + production node_modules to final image based on `node:24.9.0-slim`.

**Docker image versioning**: The version in `action.yml` (`image: docker://ghcr.io/47ng/actions-clever-cloud:X.Y.Z`) must match `package.json` version. CI checks this.

## Code Style

- ES Modules (`"type": "module"`) with `import.meta.main` guard
- No semicolons, single quotes, no trailing commas, 2-space indent
- Strict TypeScript with `noUncheckedIndexedAccess`

## Testing Patterns

Tests mock `@actions/exec` and `@actions/core` using `vi.mock()` at the top level. The `exec` mock captures calls so tests can assert on CLI arguments and call order. Tests use `vi.resetModules()` + dynamic `import()` in `beforeEach` to get fresh module state.
