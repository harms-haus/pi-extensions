# pi-* Extension Requirements

Reference standard for all pi-* extensions (excluding pi-acp, which is a standalone CLI tool, not an extension).

Every extension **must** conform to the specifications below. This document is the single source of truth for unification.

---

## 1. Required Files

Every extension must contain these files at the repo root:

| File | Purpose |
|---|---|
| `package.json` | Package manifest (see §2) |
| `tsconfig.json` | TypeScript config (see §3) |
| `eslint.config.js` | Flat ESLint config (see §4) |
| `vitest.config.ts` | Vitest config (see §5) |
| `.prettierrc` | Prettier config (see §6) |
| `.github/workflows/ci.yml` | CI pipeline (see §7) |
| `.github/workflows/publish.yml` | Publish pipeline (see §8) |
| `LICENSE` | MIT |
| `README.md` | Description, usage, install |

---

## 2. package.json

### 2.1 Required Fields

```jsonc
{
  "name": "@harms-haus/pi-<name>",       // scoped
  "version": "0.1.0",                     // or current
  "description": "...",
  "type": "module",
  "main": "src/index.ts",
  "author": "harms-haus",
  "license": "MIT",
  "keywords": ["pi-package"],
  "repository": {
    "type": "git",
    "url": "git+https://github.com/harms-haus/pi-<name>.git"
  },
  "engines": {
    "node": ">=22.0.0"                    // raised from >=20
  },
  "publishConfig": {
    "access": "public"
  },
  "files": [
    "src/**/*.ts",
    "!src/__tests__/**",
    "docs/",                              // only if docs/ exists
    "skills/",                            // only if skills/ exists
    "README.md",
    "LICENSE"
  ],
  "pi": {
    "extensions": ["./src/index.ts"],
    "skills": ["./skills/<skill-name>"]   // only if skills/ exists
  }
}
```

> **Note on scope:** All extensions must use the `@harms-haus/` scope. Only pi-lens is currently scoped; the rest need to be renamed from `pi-*` to `@harms-haus/pi-*`.

### 2.2 Required Scripts

```jsonc
{
  "scripts": {
    "lint": "eslint src/",
    "lint:fix": "eslint --fix src/",
    "format": "prettier --write src/",
    "format:check": "prettier --check src/",
    "typecheck": "tsc --noEmit",
    "test": "vitest run",
    "test:watch": "vitest",
    "test:coverage": "vitest run --coverage"
  }
}
```

### 2.3 devDependencies — Pinned Versions

All extensions must use **exactly** these version ranges:

```jsonc
{
  "devDependencies": {
    "@eslint/js": "^10.0.1",
    "@types/node": "^22.0.0",
    "@vitest/coverage-v8": "^4.1.6",
    "eslint": "^10.4.0",
    "eslint-config-prettier": "^10.1.8",
    "prettier": "^3.8.3",
    "typescript": "^6.0.3",
    "typescript-eslint": "^8.59.3",
    "vitest": "^4.1.6"
  }
}
```

### 2.4 peerDependencies

Extensions must declare the pi packages they actually import from at runtime. Use `*` range
(the host agent resolves the version):

| If the extension imports from… | Declare this peer |
|---|---|
| `@earendil-works/pi-coding-agent` | `"@earendil-works/pi-coding-agent": "*"` |
| `@earendil-works/pi-tui` | `"@earendil-works/pi-tui": "*"` |
| `@earendil-works/pi-ai` | `"@earendil-works/pi-ai": "*"` |
| `typebox` | `"typebox": "*"` |

Do **not** pin peer dependencies to `^0.74.0` — use `*`. The extension should be compatible
with any version the host pi agent provides.

Do **not** put pi packages in `devDependencies` unless they are needed for type-checking at
dev time and are also declared as peers. Having them in both `devDependencies` and
`peerDependencies` is acceptable for local development.

---

## 3. tsconfig.json

```jsonc
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "esModuleInterop": true,
    "strict": true,
    "skipLibCheck": true,
    "noEmit": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "noUncheckedIndexedAccess": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules"]
}
```

Key points:
- `noEmit: true` — extensions ship raw `.ts` source, not compiled output
- `noUncheckedIndexedAccess: true` — catches unsafe index access (`arr[i]`)

---

## 4. eslint.config.js

```js
import eslint from "@eslint/js";
import tseslint from "typescript-eslint";
import prettierConfig from "eslint-config-prettier";

export default tseslint.config(
  eslint.configs.recommended,
  ...tseslint.configs.strictTypeChecked,
  prettierConfig,
  {
    ignores: ["dist/", "node_modules/", "coverage/", "vitest.config.ts"],
  },
  {
    files: ["src/**/*.ts"],
    languageOptions: {
      parserOptions: {
        projectService: true,
        tsconfigRootDir: import.meta.dirname,
      },
    },
    rules: {
      "@typescript-eslint/no-explicit-any": "error",
      "@typescript-eslint/explicit-function-return-type": "off",
      "@typescript-eslint/no-unused-vars": ["error", { argsIgnorePattern: "^_" }],
      "@typescript-eslint/consistent-type-imports": ["error", { prefer: "type-imports" }],
      "@typescript-eslint/no-non-null-assertion": "error",
      "no-console": ["warn", { allow: ["warn", "error"] }],

      "max-depth": ["error", 5],
      "max-lines-per-function": ["error", { max: 100, skipBlankLines: true, skipComments: true }],
      complexity: ["error", 15],

      "@typescript-eslint/no-unsafe-argument": "error",
      "@typescript-eslint/no-unsafe-assignment": "error",
      "@typescript-eslint/no-unsafe-call": "error",
      "@typescript-eslint/no-unsafe-member-access": "error",
      "@typescript-eslint/no-unsafe-return": "error",
      "@typescript-eslint/no-unsafe-enum-comparison": "error",
      "@typescript-eslint/no-floating-promises": "error",
      "@typescript-eslint/no-misused-promises": [
        "error",
        { checksConditionals: true, checksVoidReturn: true },
      ],
      "@typescript-eslint/no-unnecessary-condition": "error",
      "@typescript-eslint/restrict-template-expressions": [
        "error",
        { allowNumber: true, allowBoolean: true },
      ],
      "@typescript-eslint/require-await": "warn",
    },
  },
  {
    files: ["src/**/*.test.ts", "src/**/setup.ts", "src/**/helpers/*.ts"],
    rules: {
      "@typescript-eslint/no-explicit-any": "off",
      "@typescript-eslint/no-non-null-assertion": "off",
      "@typescript-eslint/unbound-method": "off",
      "@typescript-eslint/no-unsafe-argument": "off",
      "@typescript-eslint/no-unsafe-assignment": "off",
      "@typescript-eslint/no-unsafe-call": "off",
      "@typescript-eslint/no-unsafe-member-access": "off",
      "@typescript-eslint/no-unsafe-return": "off",
      "@typescript-eslint/no-misused-promises": [
        "error",
        { checksConditionals: false, checksVoidReturn: false },
      ],
      "@typescript-eslint/no-unnecessary-condition": "warn",
      "@typescript-eslint/no-base-to-string": "warn",
      "@typescript-eslint/require-await": "off",
      "max-lines-per-function": "off",
      complexity: "off",
      "max-depth": "off",
    },
  },
);
```

Key points:
- Uses flat config (`eslint.config.js`), not legacy `.eslintrc`
- `strictTypeChecked` base with explicit `error` on all unsafe-* rules
- Test files get relaxed rules for ergonomics
- All src rules are `error` (not `warn`) except `require-await` which is `warn` and `no-console` which is `warn`
- Test overrides include `no-unnecessary-condition: warn` and `no-base-to-string: warn` for practicality

---

## 5. vitest.config.ts

```ts
import { defineConfig } from "vitest/config";

export default defineConfig({
  test: {
    include: ["src/**/*.test.ts"],
    coverage: {
      provider: "v8",
      reporter: ["text", "lcov"],
      include: ["src/**/*.ts"],
      exclude: [
        "src/__tests__/**",
        "src/**/*.test.ts",
        "src/**/setup.ts",
        "src/**/helpers/**",
        "src/**/*.d.ts",
        "src/types.ts",
        "src/types/**",
      ],
      thresholds: {
        statements: 90,
        branches: 90,
        functions: 90,
        lines: 90,
      },
    },
  },
});
```

Add `setupFiles: ["src/__tests__/setup.ts"]` only if a setup file exists.

---

## 6. .prettierrc

```json
{
  "singleQuote": false,
  "trailingComma": "all",
  "printWidth": 100
}
```

> Adjust if a different canonical format is desired, but it must be **identical** across all extensions.

---

## 7. CI Workflow — `.github/workflows/ci.yml`

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  check:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [22, 24]

    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
          cache: npm

      - run: npm ci

      - name: Type check
        run: npm run typecheck

      - name: Lint
        run: npm run lint

      - name: Format check
        run: npm run format:check

      - name: Test with coverage
        run: npm run test:coverage
```

Key points:
- Tests against **Node 22 and Node 24** only (Node 20 removed)
- Runs `test:coverage` (not plain `test`) so coverage thresholds are enforced
- No build step — extensions ship raw TypeScript

---

## 8. Publish Workflow — `.github/workflows/publish.yml`

```yaml
name: Publish to npm

on:
  release:
    types: [published]

permissions:
  id-token: write
  contents: read

jobs:
  publish:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 22
          cache: npm
          registry-url: https://registry.npmjs.org
      - run: npm ci
      - run: npm publish --access public
        env:
          NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}
```

Key points:
- Triggered by GitHub **release** events (not tag push)
- Uses Node 22 for publishing
- `permissions: id-token: write` enables npm provenance signing
- Requires `NPM_TOKEN` secret to be configured in the repo

---

## 9. Extension-by-Extension Gap Summary

Below is a checklist of what needs to change in each extension to meet the standard above.
This is a living checklist — update as changes are applied.

### Legend

- ✅ Already conforms
- ❌ Needs change
- ➖ Not applicable (extension doesn't use that dep)

### pi-cwd

| Item | Status | Notes |
|---|---|---|
| Package name scoped | ❌ | `pi-cwd` → `@harms-haus/pi-cwd` |
| `@types/node` in devDeps | ❌ | Missing — add `"^22.0.0"` |
| peerDeps use `*` range | ❌ | `^0.74.0` → `*` |
| eslint config: `require-await` rule | ❌ | Missing — add `warn` |
| eslint config: test overrides | ❌ | Missing `no-unnecessary-condition: warn`, `no-base-to-string: warn`, `require-await: off` |
| tsconfig: `noUncheckedIndexedAccess` | ❌ | Missing — add `true` |
| CI node matrix | ❌ | `[20, 22]` → `[22, 24]` |
| CI test command | ❌ | `npm test` → `npm run test:coverage` |
| publish.yml: real publish | ❌ | Dry-run only — switch to real publish with release trigger |
| Dependency versions | ❌ | eslint `^9.0.0` → `^10.4.0`, @eslint/js `^9.0.0` → `^10.0.1`, vitest/coverage `^4.1.6`, typescript-eslint `^8.59.3` |

### pi-git

| Item | Status | Notes |
|---|---|---|
| Package name scoped | ❌ | `pi-git` → `@harms-haus/pi-git` |
| peerDeps use `*` range | ❌ | `^0.74.0` → `*` |
| eslint config: `require-await` rule | ❌ | Missing |
| eslint config: test overrides | ❌ | Missing `no-unnecessary-condition`, `no-base-to-string`, `require-await` |
| tsconfig: `noUncheckedIndexedAccess` | ❌ | Missing |
| CI node matrix | ❌ | `[20, 22]` → `[22, 24]` |
| CI test command | ❌ | `npm test` → `npm run test:coverage` |
| publish.yml: real publish | ❌ | Dry-run only |
| Dependency versions | ❌ | eslint → `^10.4.0`, @eslint/js → `^10.0.1`, typescript → `^6.0.3`, vitest/coverage → `^4.1.6`, typescript-eslint → `^8.59.3` |

### pi-lens

| Item | Status | Notes |
|---|---|---|
| Package name scoped | ✅ | Already `@harms-haus/pi-lens` |
| peerDeps use `*` range | ❌ | `^0.74.0` / `^1.1.38` → `*` |
| eslint config: `require-await` in src rules | ❌ | Missing |
| eslint config: src `no-unnecessary-condition` | ❌ | Missing from src rules (only in test overrides) |
| tsconfig: `noUncheckedIndexedAccess` | ❌ | Missing |
| CI node matrix | ❌ | `[20, 22]` → `[22, 24]` |
| CI test command | ❌ | `npm test` → `npm run test:coverage` |
| publish.yml: node version | ❌ | `24` → `22` |
| Dependency versions | ❌ | eslint → `^10.4.0`, @eslint/js → `^10.0.1`, typescript → `^6.0.3`, vitest/coverage → `^4.1.6`, typescript-eslint → `^8.59.3` |
| eslint config: remove extra test overrides | ❌ | Remove `no-unnecessary-type-assertion`, `no-redundant-type-constituents`, `no-unsafe-function-type` from test block |

### pi-powerline

| Item | Status | Notes |
|---|---|---|
| Package name scoped | ❌ | `pi-powerline` → `@harms-haus/pi-powerline` |
| `@types/node` in devDeps | ✅ | Already present |
| peerDeps use `*` range | ❌ | `^0.74.0` → `*` |
| eslint config: `require-await` rule | ❌ | Missing |
| eslint config: test overrides | ❌ | Missing `no-unnecessary-condition`, `no-base-to-string`, `require-await` |
| tsconfig: `noUncheckedIndexedAccess` | ❌ | Missing |
| CI node matrix | ❌ | `[20, 22]` → `[22, 24]` |
| CI test command | ❌ | `npm test` → `npm run test:coverage` |
| publish.yml: real publish | ❌ | Dry-run only |
| Dependency versions | ❌ | eslint → `^10.4.0`, @eslint/js → `^10.0.1`, typescript → `^6.0.3`, vitest/coverage → `^4.1.6`, typescript-eslint → `^8.59.3` |

### pi-processes

| Item | Status | Notes |
|---|---|---|
| Package name scoped | ❌ | `pi-processes` → `@harms-haus/pi-processes` |
| peerDeps use `*` range | ❌ | `^0.74.0` → `*` |
| eslint config: `require-await` rule | ❌ | Missing from src rules |
| eslint config: test overrides | ❌ | Missing `no-unnecessary-condition`, `no-base-to-string`, `require-await` |
| tsconfig: `noUncheckedIndexedAccess` | ❌ | Missing |
| CI node matrix | ❌ | `[20, 22]` → `[22, 24]` |
| CI test: duplicate test steps | ❌ | Remove `npm test` step, keep only `npm run test:coverage` |
| publish.yml: real publish | ❌ | Dry-run only |
| Dependency versions | ✅ | Already at highest versions |

### pi-searxng

| Item | Status | Notes |
|---|---|---|
| Package name scoped | ❌ | `pi-searxng` → `@harms-haus/pi-searxng` |
| peerDeps use `*` range | ❌ | `^0.74.0` / `^1.1.38` → `*` |
| eslint config: `require-await` rule | ❌ | Missing |
| eslint config: test overrides | ❌ | Missing `no-unnecessary-condition`, `no-base-to-string`, `require-await` |
| tsconfig: `noUncheckedIndexedAccess` | ❌ | Missing |
| CI node matrix | ❌ | `[20, 22]` → `[22, 24]` |
| CI test command | ❌ | `npm test` → `npm run test:coverage` |
| publish.yml: real publish | ❌ | Dry-run only |
| Dependency versions | ❌ | eslint → `^10.4.0`, @eslint/js → `^10.0.1`, typescript → `^6.0.3`, vitest/coverage → `^4.1.6`, typescript-eslint → `^8.59.3` |

### pi-subagents

| Item | Status | Notes |
|---|---|---|
| Package name scoped | ❌ | `pi-subagents` → `@harms-haus/pi-subagents` |
| eslint config: `max-depth` severity | ❌ | `warn` → `error` |
| eslint config: `complexity` severity | ❌ | `warn` → `error` |
| eslint config: `no-unnecessary-condition` in src | ❌ | `warn` → `error` |
| eslint config: `restrict-template-expressions` severity | ❌ | `warn` → `error` |
| eslint config: test overrides | ❌ | Missing `no-unnecessary-condition: warn`, `no-base-to-string: warn` |
| tsconfig: `noUncheckedIndexedAccess` | ❌ | Missing |
| peerDeps use `*` range | ❌ | `^0.74.0` → `*` |
| CI node matrix | ❌ | `[20, 22]` → `[22, 24]` |
| CI test command | ❌ | `npm test` → `npm run test:coverage` |
| publish.yml: real publish | ❌ | Dry-run only |
| Dependency versions | ❌ | typescript → `^6.0.3`, vitest/coverage → `^4.1.6` |
| vitest @vitest/coverage-v8 version format | ❌ | `"3"` → `"^4.1.6"` |

### pi-tasks

| Item | Status | Notes |
|---|---|---|
| Package name scoped | ❌ | `pi-tasks` → `@harms-haus/pi-tasks` |
| `publishConfig.access: "public"` | ❌ | Missing — add `"public"` |
| `repository` field | ❌ | Missing — add |
| `@types/node` in devDeps | ✅ | Present |
| peerDeps use `*` range | ✅ | Already `*` |
| eslint config: `require-await` rule | ❌ | Missing |
| eslint config: test overrides | ⚠️ | Has `no-unnecessary-condition` and `no-base-to-string` but missing `require-await: off` |
| tsconfig: `noUncheckedIndexedAccess` | ❌ | Missing |
| CI node matrix | ❌ | `[20, 22]` → `[22, 24]` |
| publish.yml | ❌ | **Missing entirely** — create real publish workflow |
| Dependency versions | ❌ | eslint → `^10.4.0`, @eslint/js → `^10.0.1`, typescript → `^6.0.3`, vitest/coverage → `^4.1.6`, typescript-eslint → `^8.59.3` |

### pi-til-done

| Item | Status | Notes |
|---|---|---|
| Package name scoped | ❌ | `pi-til-done` → `@harms-haus/pi-til-done` |
| peerDeps use `*` range | ✅ | Already `*` |
| eslint config: `require-await` rule | ❌ | Missing from src rules |
| eslint config: test overrides | ⚠️ | Has `no-unnecessary-condition` and `no-base-to-string` but missing `require-await: off` |
| tsconfig: `noUncheckedIndexedAccess` | ✅ | Already `true` |
| CI node matrix | ❌ | `[20, 22]` → `[22, 24]` |
| CI test command | ❌ | `npm test` → `npm run test:coverage` |
| publish.yml: real publish | ❌ | Dry-run only |
| Dependency versions | ❌ | eslint → `^10.4.0`, @eslint/js → `^10.0.1`, typescript → `^6.0.3`, vitest/coverage → `^4.1.6`, typescript-eslint → `^8.59.3` |

### pi-update

| Item | Status | Notes |
|---|---|---|
| Package name scoped | ❌ | `pi-update` → `@harms-haus/pi-update` |
| `@types/node` in devDeps | ❌ | Missing — add `"^22.0.0"` |
| peerDeps use `*` range | ❌ | `^0.74.0` → `*` |
| eslint config: `require-await` rule | ❌ | Missing |
| eslint config: test overrides | ❌ | Missing `no-unnecessary-condition`, `no-base-to-string`, `require-await` |
| tsconfig: `noUncheckedIndexedAccess` | ❌ | Missing |
| CI node matrix | ❌ | `[20, 22]` → `[22, 24]` |
| CI test command | ❌ | `npm test` → `npm run test:coverage` |
| publish.yml: real publish | ❌ | Dry-run only |
| Dependency versions | ❌ | eslint → `^10.4.0`, @eslint/js → `^10.0.1`, typescript → `^6.0.3`, vitest/coverage → `^4.1.6`, typescript-eslint → `^8.59.3`, eslint-config-prettier → `^10.1.8`, prettier → `^3.8.3` |
| pi-coding-agent in devDeps | ⚠️ | Present as devDep but also needed as peer — keep both |

### pi-web-content

| Item | Status | Notes |
|---|---|---|
| Package name scoped | ❌ | `pi-web-content` → `@harms-haus/pi-web-content` |
| peerDeps use `*` range | ✅ | Already `*` |
| eslint config: `require-await` rule | ❌ | Missing |
| eslint config: test overrides | ❌ | Missing `no-unnecessary-condition`, `no-base-to-string`, `require-await` |
| tsconfig: `noUncheckedIndexedAccess` | ❌ | Missing |
| CI node matrix | ❌ | `[20, 22]` → `[22, 24]` |
| CI test command | ❌ | `npm test` → `npm run test:coverage` |
| publish.yml: real publish | ❌ | Dry-run only |
| Dependency versions | ❌ | eslint → `^10.4.0`, @eslint/js → `^10.0.1`, typescript → `^6.0.3`, typescript-eslint → `^8.59.3` |
| vitest.config.ts: uses `npx` prefix | ⚠️ | Not needed with vitest in devDeps, but harmless |

### pi-workflows

| Item | Status | Notes |
|---|---|---|
| Package name scoped | ❌ | `pi-workflows` → `@harms-haus/pi-workflows` |
| `@types/node` in devDeps | ❌ | Missing — add `"^22.0.0"` |
| peerDeps use `*` range | ❌ | `^0.74.0` → `*` |
| eslint config: `require-await` rule | ❌ | Missing |
| eslint config: test overrides | ❌ | Missing `no-unnecessary-condition`, `no-base-to-string`, `require-await` |
| tsconfig: `noUncheckedIndexedAccess` | ❌ | Missing |
| CI node matrix | ❌ | `[20, 22]` → `[22, 24]` |
| CI test command | ❌ | `npm test` → `npm run test:coverage` |
| publish.yml: real publish | ❌ | Dry-run only |
| Dependency versions | ❌ | eslint → `^10.4.0`, @eslint/js → `^10.0.1`, typescript → `^6.0.3`, typescript-eslint → `^8.59.3` |
| `dependencies` contains pi packages | ⚠️ | `@earendil-works/pi-ai`, `@earendil-works/pi-tui`, `typebox` are in `dependencies` — should be `peerDependencies` only |

### pi-worktrees

| Item | Status | Notes |
|---|---|---|
| Package name scoped | ❌ | `pi-worktrees` → `@harms-haus/pi-worktrees` |
| peerDeps use `*` range | ❌ | `^0.74.0` → `*` |
| eslint config: `require-await` rule | ❌ | Missing |
| eslint config: test overrides | ❌ | Missing `no-unnecessary-condition`, `no-base-to-string`, `require-await` |
| tsconfig: `noUncheckedIndexedAccess` | ❌ | Missing |
| CI node matrix | ❌ | `[20, 22]` → `[22, 24]` |
| CI test command | ❌ | `npm test` → `npm run test:coverage` |
| publish.yml: real publish | ❌ | Dry-run only |
| Dependency versions | ❌ | eslint → `^10.4.0`, @eslint/js → `^10.0.1`, typescript → `^6.0.3`, typescript-eslint → `^8.59.3` |
| `pi-coding-agent` in devDeps | ⚠️ | Present as devDep but also needed as peer — keep both |

### pi-zai-usage

| Item | Status | Notes |
|---|---|---|
| Package name scoped | ❌ | `pi-zai-usage` → `@harms-haus/pi-zai-usage` |
| CI workflow | ❌ | **Missing entirely** — create ci.yml |
| publish.yml | ❌ | **Missing entirely** — create publish.yml |
| peerDeps use `*` range | ❌ | `^0.74.0` → `*` |
| eslint config: `require-await` rule | ❌ | Missing |
| eslint config: test overrides | ❌ | Missing `no-unnecessary-condition`, `no-base-to-string`, `require-await` |
| tsconfig: `noUncheckedIndexedAccess` | ❌ | Missing |
| Dependency versions | ❌ | eslint → `^10.4.0`, @eslint/js → `^10.0.1`, typescript → `^6.0.3`, vitest/coverage → `^4.1.6`, typescript-eslint → `^8.59.3` |
| `pi-coding-agent` in devDeps | ⚠️ | Present as devDep but also needed as peer — keep both |

---

## 10. Migration Checklist

Apply these changes to every extension (excluding pi-acp):

### Dependency bumps (all extensions)
- [ ] `eslint` → `^10.4.0`
- [ ] `@eslint/js` → `^10.0.1`
- [ ] `typescript` → `^6.0.3`
- [ ] `typescript-eslint` → `^8.59.3`
- [ ] `vitest` → `^4.1.6`
- [ ] `@vitest/coverage-v8` → `^4.1.6`
- [ ] `eslint-config-prettier` → `^10.1.8`
- [ ] `prettier` → `^3.8.3`
- [ ] `@types/node` → `^22.0.0` (add if missing)
- [ ] Remove `^0.74.0` from peer deps → use `*`

### Config files (all extensions)
- [ ] Replace `eslint.config.js` with canonical version from §4
- [ ] Add `noUncheckedIndexedAccess: true` to tsconfig.json
- [ ] Confirm vitest.config.ts matches §5

### CI (all extensions)
- [ ] Change node matrix from `[20, 22]` to `[22, 24]`
- [ ] Change test step from `npm test` to `npm run test:coverage`
- [ ] Create ci.yml for pi-zai-usage
- [ ] Remove duplicate test step in pi-processes

### Publish (all extensions)
- [ ] Replace dry-run publish.yml with real publish workflow from §8
- [ ] Create publish.yml for pi-tasks and pi-zai-usage
- [ ] Ensure `NPM_TOKEN` secret is configured in each GitHub repo

### package.json fields (all extensions)
- [ ] Rename package from `pi-*` to `@harms-haus/pi-*` (pi-lens already done)
- [ ] Ensure `publishConfig.access: "public"` is present
- [ ] Ensure `repository` field is present
- [ ] Change `engines.node` from `>=20.0.0` to `>=22.0.0`
- [ ] Move pi packages from `dependencies` to `peerDependencies` where applicable (pi-workflows)
