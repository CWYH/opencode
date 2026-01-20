# Agent Handbook (read before editing)

- Default branch is `dev`; branch from it unless told otherwise.
- Package manager is `bun@1.3.5`; install via `bun install` at repo root (workspace-aware).
- ALWAYS USE PARALLEL TOOLS WHEN APPLICABLE.
- Respect existing user changes; never hard reset or delete files you did not create.
- No Cursor or Copilot instruction files found in the repo.
- Package-level AGENTS exist in `packages/opencode` and `packages/app`; guidance from them is incorporated below—do not ignore local notes.

## Repo map (top-level)

- Root: orchestration only; `bun run dev` proxies to `packages/opencode` dev entry.
- `packages/opencode`: core CLI/server and primary test suite (Bun).
- `packages/app`: SolidJS front-end with Playwright e2e tests.
- `packages/web`: Astro docs site; build-only by default.
- `packages/sdk/js`: SDK generator scripts.
- Other packages (ui, util, plugin, script, console/*, enterprise, desktop, slack) follow workspace conventions; inspect local package.json when working there.

## Installation and env

- Run `bun install` once at repo root to install all workspaces.
- Prefer Bun APIs (`Bun.file`, `Bun.write`, `Bun.argv`) over Node equivalents.
- Husky is present; allow hooks to run. Avoid disabling or skipping hooks.
- Keep ASCII unless a file already contains Unicode.

## Build commands (package-specific)

- Root: no meaningful build; use package builds directly.
- `packages/opencode`: `bun run build` (executes `script/build.ts`).
- `packages/app`: `bun run build` (Vite/Solid build).
- `packages/web`: `bun run build` (Astro build).
- `packages/sdk/js`: regenerate SDK with `bun run ./packages/sdk/js/script/build.ts` from repo root.
- `packages/script` and others: consult local scripts when needed; default to none.

## Dev servers

- `packages/opencode`: `bun run --conditions=browser ./src/index.ts` or root `bun run dev`.
- `packages/app`: `bun run dev` (Vite). IMPORTANT: Per package AGENTS, the app is often already running at http://localhost:3000 via Playwright MCP server—NEVER restart that running app or its server process.
- `packages/web`: `bun run dev` or `bun run dev:remote` (uses production API).

## Type checking

- Root: `bun run typecheck` (turbo pipeline across workspaces).
- `packages/opencode`: `bun run typecheck` (tsgo --noEmit).
- `packages/app`: `bun run typecheck` (tsgo -b).
- Others: run `bun run typecheck` where defined; check package.json.

## Linting and formatting

- No repo-wide lint. Follow package scripts.
- `packages/opencode`: `bun run lint` currently runs `bun test --coverage`; prefer direct tests instead of relying on lint.
- Formatting: Prettier config in root package.json (`semi: false`, `printWidth: 120`). Run package-level formatter only when requested; avoid mass reformatting.

## Testing overview

- Do NOT run `bun test` at repo root (script exits with error intentionally).
- `packages/opencode`: `bun test` runs all tests.
- `packages/app`: `bun run test` or `bun run test:e2e` runs Playwright; `bun run test:e2e:ui` opens UI; `bun run test:e2e:report` shows report.
- `packages/web`: no tests defined.
- Other packages: check local scripts before running tests.

## Single-test recipes

- `packages/opencode`: `bun test path/to/file.test.ts` or `bun test path/to/file.test.ts --filter "name"`.
- `packages/app`: `bun run test -- path/to/spec.ts` or `bun run test -- --grep "keyword"` for targeted Playwright runs.
- Turborepo scoped: `bun x turbo run test --filter=<package>` when you need package-level isolation.

## Debugging notes

- From `packages/app` AGENTS: use the Playwright MCP server; app likely already running on port 3000—do not restart it.
- For Bun debugging, prefer `bun --inspect ...` and keep logs minimal.

## SDK generation

- After server API changes (e.g., in `packages/opencode/src/server/server.ts`), run `bun run ./packages/sdk/js/script/build.ts` from repo root to regenerate the JS SDK and commit resulting outputs.

## Style guide (must follow)

- Keep logic in one function unless composability/reuse justifies splitting.
- Prefer `const`; avoid `let`, especially paired with conditionals.
- Avoid `else`; use early returns or IIFEs.
- Avoid unnecessary destructuring; use property access (`obj.a`) to retain context.
- Avoid `try`/`catch` when possible; design control flow to not rely on exceptions.
- Avoid `any`; prefer explicit types or inference.
- Prefer single-word names for variables/functions; expand only when unavoidable.
- Use Bun APIs when available instead of Node alternatives.

## Imports and modules

- TypeScript ESM throughout; use `type` imports when importing only types.
- Favor relative imports for local modules; avoid introducing new path aliases without config.
- Keep import grouping minimal: external libs, then local modules; do not churn ordering unnecessarily.

## Formatting expectations

- Honor Prettier settings (`semi: false`, `printWidth: 120`).
- Keep statements expression-oriented; favor ternaries and early returns over nested branches.
- Maintain existing ASCII unless a file already uses Unicode.

## Types and validation

- Use Zod for runtime validation when inputs cross boundaries.
- Prefer interfaces/types over classes for data shapes; keep them narrow and reusable.
- Reuse shared types from `packages/util`/SDK where available; avoid duplicating shapes.

## Naming conventions

- camelCase for variables/functions, PascalCase for types/components.
- File names follow existing conventions (kebab-case common); avoid renaming without need.
- Single-word preference per style guide.

## Error handling

- Prefer result-style returns or boolean guards instead of throws.
- If throwing is unavoidable, throw `Error` with concise message; avoid broad catches.
- Keep logs lean; avoid noisy console output in tests and libraries.

## Control flow

- Remove `else` blocks via early returns.
- Collapse simple branches with ternaries where clear.
- Avoid mutable state; rely on `const` and immutable helpers.

## Async and promises

- Prefer `await` over `.then` chains; keep functions `async` when awaited.
- Do not ignore promise rejections; return or await them.

## Data structures

- Prefer `Map`/`Set` for keyed lookups over plain objects when appropriate.
- Use array helpers (`map`, `filter`, `reduce`) for transformations; avoid manual loops unless clearer.

## SolidJS (packages/app)

- Prefer `createStore` over multiple `createSignal` calls (per local AGENTS guidance).
- Keep components small; lift state when shared.
- Use effects with care and cleanup; minimize dependencies.

## Styling (UI packages)

- Respect existing design tokens/themes; avoid introducing new globals casually.
- Prefer utility classes already in use; use inline styles only for dynamic needs.

## Logging and telemetry

- In `packages/opencode`, use `Log.create({ service: "name" })` when logging.
- Keep logs structured; never log secrets, tokens, or credentials.

## File structure and architecture (packages/opencode)

- Tools implement `Tool.Info` with `execute()`; inputs validated via Zod.
- Pass `sessionID` in tool context; use `App.provide()` for dependency injection.
- Use `Storage` namespace for persistence; avoid ad-hoc fs usage unless necessary.
- Client/server communication uses `@opencode-ai/sdk`; keep API changes in sync with SDK regeneration.

## CLI and commands (packages/opencode)

- Bin entry: `./bin/opencode`; exports map `./*` to `src/*.ts`.
- When adding commands, keep options minimal and typed; reuse existing parsers/helpers.

## Docs

- `packages/opencode`: `bun run docs` is a stub that echoes processed files; run only when asked.
- `packages/web`: use `bun run dev`/`build` for docs site work.

## Testing tips

- Keep tests hermetic; avoid real network unless explicitly mocked or required.
- Prefer focused test files; use `--filter` (Bun) or `--grep` (Playwright) for fast iteration.
- Clean up temp files via Bun APIs; avoid changing global cwd inside tests.

## Performance and cleanup

- Remove unused imports/vars; keep bundles lean.
- Prefer streaming/iterators for large data handling where applicable.

## Git hygiene

- Never amend or force-push unless explicitly directed.
- Do not commit secrets or env files; check for `.env`, credentials, or keys before staging.
- After changes, run relevant package tests before review.

## When in doubt

- Read package-level README/AGENTS for extra rules before editing.
- Ask for clarification before modifying app/server lifecycle behaviors or long-running processes.
