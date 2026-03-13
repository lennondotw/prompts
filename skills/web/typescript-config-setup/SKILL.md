# TypeScript Config Setup

## TypeScript Configuration Architecture

Use a **solution-style** TypeScript setup with project references. Split the configuration into three layers: a shared base, per-project overrides, and a root orchestrator.

### Shared Base

Create a dedicated workspace package (e.g. `packages/tsconfig`) that publishes a `base.json`. This file is the single source of truth for all compiler options shared across the monorepo. Every other project consumes it as a workspace `devDependency`.

This package contains only JSON configuration files — no TypeScript source, no tsconfig of its own, and no entry in the root `references`.

#### Compiler Options

Organize the base config into four categories:

**Project References & Build**

| Option          | Value  | Purpose                                                                                                                                                         |
| --------------- | ------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `composite`     | `true` | Enable project references. Each sub-project becomes a discrete compilation unit that `tsc --build` can build independently. Required for solution-style setups. |
| `incremental`   | `true` | Produce `.tsbuildinfo` files so subsequent builds only re-check changed files. Implied by `composite` but kept explicit for clarity.                            |
| `noEmitOnError` | `true` | Prevent writing output files when type errors exist, ensuring broken builds never produce artifacts.                                                            |

**Module System & Resolution**

| Option                      | Value        | Purpose                                                                                                                                                                    |
| --------------------------- | ------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `target`                    | `ESNext`     | Emit the latest ECMAScript syntax without down-leveling. The bundler or runtime handles compatibility.                                                                     |
| `lib`                       | `["ESNext"]` | Include only ECMAScript standard library types. Exclude DOM types — browser projects add them via per-project overrides.                                                   |
| `module`                    | `ESNext`     | Use ECMAScript module syntax for emit. Works with both bundlers (Vite, webpack) and modern Node.js runtimes.                                                               |
| `moduleResolution`          | `bundler`    | Resolve imports the way modern bundlers do — support `package.json` `exports`, path-less imports, and extensionless specifiers.                                            |
| `resolvePackageJsonImports` | `true`       | Honor the `imports` field in `package.json` for self-referencing subpath patterns (`#internal/*`).                                                                         |
| `resolvePackageJsonExports` | `true`       | Honor the `exports` field in `package.json`, ensuring TypeScript resolves the same entry points as the runtime.                                                            |
| `isolatedModules`           | `true`       | Ensure every file can be transpiled in isolation (required by esbuild, SWC, Vite, and other single-file transpilers). Forbid constructs like `const enum` across files.    |
| `verbatimModuleSyntax`      | `true`       | Enforce explicit `type` modifier on type-only imports/exports (`import type { ... }`). Eliminate ambiguity for transpilers that strip types without full program analysis. |

**Type Checking & Strictness**

| Option                       | Value  | Purpose                                                                                                                                                                                                       |
| ---------------------------- | ------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `strict`                     | `true` | Umbrella flag enabling `strictNullChecks`, `strictFunctionTypes`, `strictBindCallApply`, `strictPropertyInitialization`, `noImplicitAny`, `noImplicitThis`, `alwaysStrict`, and `useUnknownInCatchVariables`. |
| `allowJs`                    | `true` | Include `.js` / `.mjs` / `.cjs` files in the compilation.                                                                                                                                                     |
| `checkJs`                    | `true` | Apply type-checking to JavaScript files, not just TypeScript. Catch errors in config files, scripts, and legacy code.                                                                                         |
| `noUnusedLocals`             | `true` | Error on declared but unused local variables.                                                                                                                                                                 |
| `noUnusedParameters`         | `true` | Error on declared but unused function parameters.                                                                                                                                                             |
| `noFallthroughCasesInSwitch` | `true` | Error on `switch` cases that fall through without `break` or `return`.                                                                                                                                        |
| `noUncheckedIndexedAccess`   | `true` | Add `undefined` to the type of index-signature and array access results. Prevent assuming a value exists at an arbitrary key or index.                                                                        |
| `useUnknownInCatchVariables` | `true` | Type `catch` clause variables as `unknown` instead of `any`. Already implied by `strict` since TS 4.4, but keep explicit for readability.                                                                     |
| `useDefineForClassFields`    | `true` | Emit class fields using `Object.defineProperty` semantics (the TC39 standard) rather than assignment in the constructor. Match native runtime behavior.                                                       |
| `skipLibCheck`               | `true` | Skip type-checking `.d.ts` files from `node_modules`. Speed up compilation significantly. Errors in third-party declarations are not actionable.                                                              |
| `types`                      | `[]`   | Do not auto-include any `@types/*` packages. All type dependencies must be explicitly imported or declared in per-project `types` overrides. Prevent ambient global type pollution.                           |

**Output & Source Maps**

| Option           | Value  | Purpose                                                                                                                                  |
| ---------------- | ------ | ---------------------------------------------------------------------------------------------------------------------------------------- |
| `sourceMap`      | `true` | Generate `.js.map` files alongside emitted JavaScript.                                                                                   |
| `declaration`    | `true` | Generate `.d.ts` type declaration files. Required by `composite`. Applications with `noEmit: true` override this implicitly.             |
| `declarationMap` | `true` | Generate `.d.ts.map` files so editors can "Go to Definition" into the original `.ts` source of a dependency instead of its declarations. |

#### Reference JSON

```json
{
  "$schema": "https://json.schemastore.org/tsconfig",
  "compilerOptions": {
    "composite": true,
    "incremental": true,
    "noEmitOnError": true,

    "target": "ESNext",
    "lib": ["ESNext"],
    "module": "ESNext",
    "moduleResolution": "bundler",
    "resolvePackageJsonImports": true,
    "resolvePackageJsonExports": true,
    "isolatedModules": true,
    "verbatimModuleSyntax": true,

    "strict": true,
    "allowJs": true,
    "checkJs": true,
    "skipLibCheck": true,
    "types": [],
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true,
    "noUncheckedIndexedAccess": true,
    "useUnknownInCatchVariables": true,
    "useDefineForClassFields": true,

    "sourceMap": true,
    "declaration": true,
    "declarationMap": true
  }
}
```

### Per-Project tsconfig

Give every workspace project its own `tsconfig.json` that extends the shared base. In the minimal case, only two fields are needed:

- **`rootDir`** — typically `"."` (the project root)
- **`outDir`** — a path under `node_modules/.tmp/tsc-out/` by convention, keeping build artifacts out of the source tree

Inherit all other compiler options from the base. Add overrides only when the project's runtime environment differs from the defaults:

| Override                                 | When                                                          |
| ---------------------------------------- | ------------------------------------------------------------- |
| `lib: ["ESNext", "DOM", "DOM.Iterable"]` | Browser projects                                              |
| `jsx: "react-jsx"`                       | React projects                                                |
| `noEmit: true`                           | Applications that do not produce declaration files            |
| `types: [...]`                           | Projects relying on ambient global types (e.g. `vite/client`) |

Set the `include` array to cover all files the project owns: source directories, test directories, and config files (e.g. `["src", "e2e", "*.config.ts"]`).

#### Multi-tsconfig Projects

When a project contains file sets with different compilation requirements (e.g. browser source, Node config files, and test suites), apply the same solution-style pattern at the package level.

Turn the project's `tsconfig.json` into a local orchestrator with `files: []` and `references` pointing to its internal tsconfigs:

```json
{
  "files": [],
  "references": [
    { "path": "./tsconfig.app.json" },
    { "path": "./tsconfig.node.json" },
    { "path": "./tsconfig.test.json" }
  ]
}
```

These internal tsconfigs may have dependency relationships. For example, tests import from `src/`, so `tsconfig.test.json` should declare a reference to `tsconfig.app.json`:

```json
{
  "extends": "<shared-base>/base.json",
  "compilerOptions": {
    "rootDir": ".",
    "outDir": "node_modules/.tmp/tsc-test/",
    "tsBuildInfoFile": "node_modules/.tmp/tsc-test/.tsbuildinfo"
  },
  "include": ["src", "tests"],
  "references": [{ "path": "./tsconfig.app.json" }]
}
```

Follow these rules for multi-tsconfig projects:

- Assign each tsconfig a **distinct `outDir` subdirectory** (e.g. `tsc-app/`, `tsc-node/`, `tsc-test/`) to prevent output collisions.
- Always **explicitly set `tsBuildInfoFile`** to a path inside the corresponding `outDir`. Without this, TypeScript may place `.tsbuildinfo` at unexpected locations, breaking incremental builds.
- Reference only the package-level `tsconfig.json` (the local orchestrator) from the root — let the package manage its own internal structure.

### Root tsconfig (Solution Style)

Create a root `tsconfig.json` that compiles nothing. It serves two purposes:

- **Project discovery for `tsc --build`**: Set `files: []` so no source files are associated with the root project. List every sub-project in `references`. Running `tsc --build` from the root traverses these references in dependency order and type-checks each project.
- **Language Server discovery**: The TypeScript Language Server uses the root `references` to locate all tsconfig files in the workspace, including non-standard names like `tsconfig.node.json` or `tsconfig.test.json`. Without these references, the LS only discovers files named `tsconfig.json`.

Create a companion `tsconfig.node.json` at the root to cover Node-side configuration files that live outside any project (e.g. `commitlint.config.js`). Extend the same shared base and list it as a reference in the root tsconfig.

### `tsc --project` vs `tsc --build`

|             | `tsc --project <tsconfig>`   | `tsc --build <tsconfig>`                     |
| ----------- | ---------------------------- | -------------------------------------------- |
| Mode        | Single-project compilation   | Build mode with project references           |
| References  | Ignored                      | Resolved and built in dependency order       |
| Incremental | Only if `incremental` is set | Always uses `.tsbuildinfo` (via `composite`) |
| Typical use | Legacy single-package setups | Monorepos with project references            |

**Prefer `tsc --build` in all cases.** It respects the dependency graph defined by `references`, skips up-to-date projects via incremental `.tsbuildinfo`, and correctly handles multi-project builds. `tsc --project` compiles a single tsconfig in isolation and is only appropriate when intentionally ignoring project references.

Use `tsc --build --clean` to delete all output files (`.js`, `.d.ts`, `.js.map`, `.d.ts.map`, `.tsbuildinfo`) generated by previous builds across the referenced project graph. This is useful when switching branches, resolving stale output issues, or forcing a full rebuild from scratch. It does not re-compile — follow it with `tsc --build` to rebuild.

### pnpm Workspace Commands

Leverage pnpm's `--filter` flag for dependency-aware type-checking across the workspace:

| Command                                   | Scope                                                         |
| ----------------------------------------- | ------------------------------------------------------------- |
| `pnpm --filter <pkg> run typecheck`       | The package itself                                            |
| `pnpm --filter <pkg>... run typecheck`    | The package and all its upstream dependencies                 |
| `pnpm --filter <pkg>^... run typecheck`   | Only the upstream dependencies (excluding the package itself) |
| `pnpm --filter ...<pkg> run typecheck`    | The package and all its downstream dependents                 |
| `pnpm --filter ...^<pkg> run typecheck`   | Only the downstream dependents (excluding the package itself) |
| `pnpm --filter ...<pkg>... run typecheck` | The package, all upstream, and all downstream                 |
| `pnpm --filter '*' run typecheck`         | All packages in the workspace                                 |

For full workspace type-checking:

```bash
pnpm --filter '*' --workspace-concurrency=0 run typecheck
```

**`--workspace-concurrency`** controls the maximum number of tasks running simultaneously. The default is `4`. When set to `<= 0`, pnpm computes the limit as `max(1, (CPU cores) - abs(value))` — so `0` matches the core count exactly, and `-1` uses cores minus one. Set to a positive integer for a specific limit, or `Infinity` for no limit.

**Do not use `--parallel`.** Packages in a monorepo form a DAG through their dependency relationships. By default, pnpm respects this topological ordering — upstream dependencies are processed before their dependents. `--parallel` ignores the DAG entirely and runs all matched tasks concurrently regardless of dependency order. Since TypeScript type checking requires upstream packages to be checked first (a downstream project depends on the type declarations of its upstream dependencies), `--parallel` would break this constraint and produce incorrect results.

### Adding a New Project

When adding a new project to the monorepo, ensure it has:

1. A `tsconfig.json` that extends the shared base with `rootDir`, `outDir`, and any environment-specific overrides.
2. The shared tsconfig package as a `devDependency` (`workspace:*`).
3. A corresponding `{ "path": "..." }` entry in the root `tsconfig.json` references.
