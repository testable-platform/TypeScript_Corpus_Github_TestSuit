# TypeScript Order Platform -- Microservices (TS_V12_ROLLUP_BUN_MICRO)

Tool-evaluation repository for **Node 12**, bundled with **rollup**,
managed with **bun**, in a **Microservices** layout.

This is branch **TS_V12_ROLLUP_BUN_MICRO** of the consolidated `typescript-corpus` repository, which holds all 216 TypeScript branches across every Node version, bundler, package manager and architecture combination in this corpus.

## Project type

- **Language:** TypeScript 5.0.4
- **Runtime:** Node 12 (verified against 12.22.12)
- **Scenario:** 2 - Microservices
- **Architecture:** Microservices
- **Module layout:** workspace
- **Bundler:** Rollup 2.80.0 (Rollup linker + tsc 5.0.4 transform)
- **Package manager:** bun 1.4.0
- **Source root:** `packages/domain/src`

Node 12 is end-of-life, and that is deliberate: it pins the entire toolchain to
the last release of each tool that still supports it. Every version in this
repository was resolved against the live npm registry and then executed on a
real Node 12.22.12 interpreter. None was written from memory.

## Build

```bash
# bun is a standalone binary, not an npm package -- see Tool Triggering (Synthetic Data)/npm-audit/run_npm_audit.sh
bun install --frozen-lockfile
bun run build
bun run bundle
```

`bun run build` type-checks with `tsc --noEmit` and emits CommonJS + declarations to
`dist/`; `bun run bundle` then bundles with **Rollup 2.80.0 (Rollup linker + tsc 5.0.4 transform)** and **executes the bundle**.
Emitting is not proof; running it is.

## Run

```bash
node dist/packages/domain/src/index.js
```

Each service is independently runnable:

```bash
node dist/services/gateway-service/src/index.js
node dist/services/pricing-service/src/index.js
```

## Test

```bash
bun run test        # mocha over tests/
bun run coverage    # c8 (primary) AND nyc + ts-node (cross-check)
```

Both coverage tools must report non-zero. They deliberately disagree: c8 reads
V8 coverage of the emitted output and remaps it, while nyc instruments the
TypeScript AST directly through `ts-node/register`. Identical numbers would mean
one of them is not an independent second opinion.

`nyc` here never reports through a source-map remap. Doing so silently yields
0% -- the file is remapped to `.ts`, an `--include` written against `dist/**`
stops matching, and the report empties while the process still exits 0.

## Architecture

**Microservices.** Three independently deployable services over a shared domain
package, wired through a workspace root.

```
packages/
  domain/         @orderkit/domain     -- models, services, analysis fixtures
  contracts/      @orderkit/contracts  -- inter-service message types
services/
  gateway-service/    accepts payloads, emits order.submitted
  order-service/      validates and prices, emits order.priced / order.rejected
  pricing-service/    owns tier and volume rates (leaf -- calls no one)
```

Each service has its own `package.json`, its own `src/index.ts` entry point and
its own start script. The workspace root lists them under `workspaces`.

**Documented inter-service call path:**

```
http.request -> gateway-service -> order.submitted
                order-service   -> pricing.quote -> pricing-service
                pricing-service -> pricing.rate  -> order-service
                order-service   -> order.priced | order.rejected
```

This matters because of what the JavaScript corpus got wrong: FlintAtlas's sheet
said `Microservices` while the repository was a flat monolith with no
workspaces, and its own README and `dataset.json` both said "monolith". Nothing
in that repo compared the two. Here `Tool Triggering (Synthetic Data)/full_check.ts` fails the build if the
declared architecture and the actual layout disagree.


## Tool test-data folders

This repo carries only `Tool Triggering (Tool Github Test data)/` from testable-platform/typescript-corpus's original per-branch task-data folders.

### `Tool Triggering (Tool Github Test data)/`
Each of the 39 tool subfolders is that tool's own real upstream code and test
suite, pulled as-is from its actual GitHub project -- not generated. `covgate/`
is the clearest case: it's genuinely Rust, not TypeScript -- `cli_interface.rs`,
`coverage_parse.rs`, `gate.rs`, `git_module.rs`, `metrics.rs`,
`render_console.rs`, `render_markdown.rs` plus its own `fixtures/`, `helpers/`
and `support/` test directories, straight from the covgate project. `ESLint/`,
`StrykerJS/`, `Opengrep/`, `pydriller/` and the rest are each that project's
own real test suite. A correct run finds whatever that upstream project's own
tests genuinely contain. `TraceGraph/` is the exception: it holds only a note, because that project publishes no public repository.

