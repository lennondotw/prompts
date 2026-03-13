# TypeScript Base Config Reference

## Compiler Options

Organize the base config into four categories:

### Project References & Build

| Option          | Value  | Purpose                                                                                            |
| --------------- | ------ | -------------------------------------------------------------------------------------------------- |
| `composite`     | `true` | Enable project references. Each sub-project becomes a discrete compilation unit for `tsc --build`. |
| `incremental`   | `true` | Produce `.tsbuildinfo` files so subsequent builds only re-check changed files.                     |
| `noEmitOnError` | `true` | Prevent writing output files when type errors exist.                                               |

### Module System & Resolution

| Option                      | Value        | Purpose                                                                                            |
| --------------------------- | ------------ | -------------------------------------------------------------------------------------------------- |
| `target`                    | `ESNext`     | Emit the latest ECMAScript syntax without down-leveling.                                           |
| `lib`                       | `["ESNext"]` | Include only ECMAScript standard library types. Browser projects add DOM via per-project override. |
| `module`                    | `ESNext`     | Use ECMAScript module syntax for emit.                                                             |
| `moduleResolution`          | `bundler`    | Resolve imports the way modern bundlers do.                                                        |
| `resolvePackageJsonImports` | `true`       | Honor the `imports` field in `package.json`.                                                       |
| `resolvePackageJsonExports` | `true`       | Honor the `exports` field in `package.json`.                                                       |
| `isolatedModules`           | `true`       | Ensure every file can be transpiled in isolation (required by esbuild, SWC, Vite).                 |
| `verbatimModuleSyntax`      | `true`       | Enforce explicit `type` modifier on type-only imports/exports.                                     |

### Type Checking & Strictness

| Option                       | Value  | Purpose                                                                                 |
| ---------------------------- | ------ | --------------------------------------------------------------------------------------- |
| `strict`                     | `true` | Umbrella flag enabling all strict checks.                                               |
| `allowJs`                    | `true` | Include `.js` / `.mjs` / `.cjs` files in compilation.                                   |
| `checkJs`                    | `true` | Apply type-checking to JavaScript files.                                                |
| `noUnusedLocals`             | `true` | Error on unused local variables.                                                        |
| `noUnusedParameters`         | `true` | Error on unused function parameters.                                                    |
| `noFallthroughCasesInSwitch` | `true` | Error on `switch` cases that fall through.                                              |
| `noUncheckedIndexedAccess`   | `true` | Add `undefined` to index-signature and array access results.                            |
| `useUnknownInCatchVariables` | `true` | Type `catch` clause variables as `unknown`.                                             |
| `useDefineForClassFields`    | `true` | Emit class fields using `Object.defineProperty` semantics.                              |
| `skipLibCheck`               | `true` | Skip type-checking `.d.ts` files from `node_modules`.                                   |
| `types`                      | `[]`   | Do not auto-include any `@types/*` packages. All type deps must be explicitly declared. |

### Output & Source Maps

| Option           | Value  | Purpose                                                                 |
| ---------------- | ------ | ----------------------------------------------------------------------- |
| `sourceMap`      | `true` | Generate `.js.map` files.                                               |
| `declaration`    | `true` | Generate `.d.ts` type declaration files.                                |
| `declarationMap` | `true` | Generate `.d.ts.map` for "Go to Definition" into original `.ts` source. |

## Reference JSON

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

## `tsc --project` vs `tsc --build`

|             | `tsc --project <tsconfig>`   | `tsc --build <tsconfig>`                     |
| ----------- | ---------------------------- | -------------------------------------------- |
| Mode        | Single-project compilation   | Build mode with project references           |
| References  | Ignored                      | Resolved and built in dependency order       |
| Incremental | Only if `incremental` is set | Always uses `.tsbuildinfo` (via `composite`) |
| Typical use | Legacy single-package setups | Monorepos with project references            |

## pnpm `--workspace-concurrency`

Controls the maximum number of tasks running simultaneously. Default is `4`. When set to `<= 0`, pnpm computes the limit as `max(1, (CPU cores) - abs(value))` — so `0` matches the core count exactly, and `-1` uses cores minus one.

## Multi-tsconfig Internal References

Tests import from `src/`, so `tsconfig.test.json` should declare a reference to `tsconfig.app.json`:

```json
{
  "extends": "<shared-base>/base.json",
  "compilerOptions": {
    "rootDir": ".",
    "outDir": "dist/ts/test/",
    "tsBuildInfoFile": "dist/ts/test/.tsbuildinfo"
  },
  "include": ["src", "tests"],
  "references": [{ "path": "./tsconfig.app.json" }]
}
```
