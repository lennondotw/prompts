---
name: typescript-config-setup
description: TypeScript solution-style configuration with project references for monorepos. Use when setting up TypeScript in a monorepo, configuring tsconfig, adding a new project to a workspace, or when the user asks about TypeScript project references, tsc --build, or tsconfig architecture.
---

# TypeScript Config Setup

Use a **solution-style** TypeScript setup with project references. Split the configuration into three layers: a shared base, per-project overrides, and a root orchestrator.

## Shared Base

Create a dedicated workspace package (e.g. `packages/tsconfig`) that publishes a `base.json`. This file is the single source of truth for all compiler options shared across the monorepo. Every other project consumes it as a workspace `devDependency`.

This package contains only JSON configuration files — no TypeScript source, no tsconfig of its own, and no entry in the root `references`.

For the complete base config reference JSON and compiler option rationale, see [reference.md](reference.md).

## Per-Project tsconfig

Give every workspace project its own `tsconfig.json` that extends the shared base. In the minimal case, only two fields are needed:

- **`rootDir`** — typically `"."` (the project root)
- **`outDir`** — a path under `dist/ts/` by convention (e.g. `dist/ts/default/`)

Inherit all other compiler options from the base. Add overrides only when the project's runtime environment differs:

| Override                                 | When                                                          |
| ---------------------------------------- | ------------------------------------------------------------- |
| `lib: ["ESNext", "DOM", "DOM.Iterable"]` | Browser projects                                              |
| `jsx: "react-jsx"`                       | React projects                                                |
| `noEmit: true`                           | Applications that do not produce declaration files            |
| `types: [...]`                           | Projects relying on ambient global types (e.g. `vite/client`) |

Set `include` to cover all files the project owns: source directories, test directories, and config files.

### Multi-tsconfig Projects

When a project contains file sets with different compilation requirements, turn `tsconfig.json` into a local orchestrator with `files: []` and `references`:

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

Rules for multi-tsconfig projects:

- Assign each tsconfig a **distinct `outDir` subdirectory** to prevent output collisions.
- Always **explicitly set `tsBuildInfoFile`** to a path inside the corresponding `outDir`.
- Reference only the package-level `tsconfig.json` from the root.

## Root tsconfig (Solution Style)

Create a root `tsconfig.json` with `files: []` and `references` listing every sub-project. It compiles nothing — serves as the entry point for `tsc --build` and Language Server project discovery.

## `tsc --build` vs `tsc --project`

**Prefer `tsc --build` in all cases.** It respects the dependency graph, skips up-to-date projects via `.tsbuildinfo`, and handles multi-project builds correctly. Use `tsc --build --clean` to delete all output files for a full rebuild.

## pnpm Workspace Commands

| Command                                | Scope                                     |
| -------------------------------------- | ----------------------------------------- |
| `pnpm --filter <pkg> run typecheck`    | The package itself                        |
| `pnpm --filter <pkg>... run typecheck` | The package and all upstream dependencies |
| `pnpm --filter '*' run typecheck`      | All packages in the workspace             |

For full workspace type-checking:

```bash
pnpm --filter '*' --workspace-concurrency=0 run typecheck
```

**Do not use `--parallel`** — it ignores the DAG and breaks TypeScript's requirement that upstream packages are checked first.

## Adding a New Project

1. A `tsconfig.json` extending the shared base with `rootDir`, `outDir`, and environment-specific overrides.
2. The shared tsconfig package as a `devDependency` (`workspace:*`).
3. A corresponding `{ "path": "..." }` entry in the root `tsconfig.json` references.

For the complete compiler options reference, see [reference.md](reference.md).
