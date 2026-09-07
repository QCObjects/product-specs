# 14 — Build Scripts Blueprint

## Purpose

Pin the build/test/lint pipelines so every repo compiles, types, and ships
the same three distributions — with the complete per-repo script tables.

Sources: core/CLI `package.json` scripts + repo AGENTS.md notes.
CI execution order: [08-ci-conventions](./08-ci-conventions.md).

## Scope

npm `scripts` in core, SDK, CLI, and templates.

## Canonical chains (normative, as observed → binding)

- core: `build:ts-types` (`tsc -p tsconfig.d.json`) → `build:ts`
  (coverage + `tsc`) → `build:browser` (`node ./build-esbuild.js`) →
  `postbuild` (`postbuild.js`).
- cli: `build:ts-types` (`transpile.js tsconfig.d.json`) → `build:ts`
  (`npm test` + `transpile.js tsconfig.json`) → `build:esbuild`
  (`build-esbuild-esm.js` → ESM only; see modalities).
- Mechanism: custom `transpile.js` (TypeScript compiler API) emits CJS;
  `build-esbuild*.js` transpile per-file, unbundled (`bundle:false`) — NOT
  bundles. Scripts MUST be committed in-repo; hermeticity binds ONLY
  `transpile.js`/`build-esbuild*.js` (`lint`, `prepare`, `generate-readme-pdf`
  fetch remote toolchains via `npx -y`).
- Outputs MUST be: `public/cjs` (require), `public/esm` (import),
  `public/browser` (bundle), `public/types` (declarations) — matching the
  `exports` map in [03-core-framework](./03-core-framework.md).

## Module modalities (normative)

Sources: `build-esbuild.js` (core), `build-esbuild-esm.js` + `transpile.js`
(CLI), `tsconfig*.json` (all repos), `package.json` `exports` maps.

| Modality | Extension | How produced | Consumed via |
|---|---|---|---|
| TypeScript sources | `.ts` (+ `.js` via `allowJs`) | authored directly; `tsconfig*.json` in every repo | `transpile.js` / `tsc` / esbuild |
| Per-file CJS | `.ts` → `.js` | `transpile.js` (TS compiler API) over `src/**/*.ts`, unbundled | `require('pkg/path')` → `./public/cjs/*.js` (on-disk `.js`, NOT `.cjs`) |
| Per-file ESM | `.ts` → `.mjs` | esbuild `bundle:false`, `format:esm`, `platform:"browser"`, `outExtension:{".js":".mjs"}`, `target:node22`, `sourcemap:true`, `keepNames:true` | `import 'pkg/path'` → `./public/esm/*.mjs` |
| Browser bundle (core repo) | `.ts` entry | esbuild IIFE bundle, `platform:browser` → `public/browser/QCObjects.js` | `<script>` tag (no bundler needed) |
| Type declarations | `.d.ts` | `tsc -p tsconfig.d.json` → SINGLE `public/types/index.d.ts` (`outFile`, not a dir; no `./types/*` target exists) | `import` type resolution, Deno |
(No `src/index.cts` / `src/index.mts` bundle entries exist in the CLI repo —
the npm-script chain is per-file ESM only. The three-format (CJS+ESM+browser)
build lives in the `build:esbuild` CLI COMMAND and the unwired
`build-esbuild.js`; wire it or use the command.)

- **No TSX/JSX:** zero `.tsx`/`.jsx` files exist in any repo and no JSX transform
  is configured (`tsconfig` has no `jsx` option; esbuild uses the `js` loader).
  Components MUST use HTML templates + `{{}}` bindings, never JSX. Adding JSX
  support REQUIRES a major-line decision with loader + `jsx` config + this spec
  updated first.
- **ESM asset quirk (CLI):** `transpile.js` post-passes `public/cjs/**/*.js` to
  rewrite dynamic `import()` of `.json/.jsonp/.md/.mdc/.text/.txt` asset paths —
  keep asset imports to those extensions or extend `extensionsToConvert` in the
  same PR.
- **QCObjects import interop (CLI esbuild plugin):** static `qcobjects` imports
  stay external; dynamic imports are rewritten through a `__toESM(require())`
  shim (`loader:'js'`). Dual-package consumers MUST test both `require()` and
  `import` paths after build changes.
- The `exports` map MUST keep the `./*.js|cjs|mjs` extension shims so deep
  imports resolve per-modality (`./public/*.js`, `./public/cjs/*.cjs`,
  `./public/esm/*.mjs`) alongside the `./*` dual branch.

## Script tables (normative — every repo MUST keep these names/meanings)

Core/SDK/CLI shared: `build`, `build:ts`, `build:ts-types`, `build:browser`
(= alias for `build:esbuild` in CLI — produces ESM only, NO browser bundle),
`build:esbuild`, `start` (`qcobjects-shell`), `test:ts-types`
(`tsc -p tsconfig.jasmine.json`), `test:jasmine` (ts-node + jasmine),
`test` (`lint` + jasmine), `lint` (eslint `src/**/*.ts --fix`),
`coverage` (TEMPLATE-ONLY — core/CLI have no `coverage` script; template runs
nyc lcov+text over `npm run test`), `preversion`
(`npm cache verify` + tests/coverage), `postversion` (push branch AND tags —
except tag-triggered-publish repos, see below), `sync`
(`git add . && git commit -am`), `v-patch|v-minor|v-major` (via `qcobjects`),
`qcobjects|cli` (passthrough), `prepare` (husky install, no-op outside git),
`cli:help`, `tree`, `generate-readme-pdf` (markdown-pdf Letter → README.pdf +
README-es.pdf, then uninstall — NOTE: `README-es.md` source is absent from the
CLI repo, so the second half fails on clean checkout).

App template deltas (`qcobjects-new-app`): `test` = eslint + jasmine;
`start` = `createcert` + `serve`; `start:dev` (watch); `serve`/`server`,
`collab`, `shell` (runs `qcobjects shell` — no such CLI subcommand; use the
`qcobjects-shell` binary), `createcert`, `http-server`, `gae-server`,
`build` (= `publish:web`, the full chain — NOT a TS step) / `build:ts`
(`npm test && npx tsc`); parcel `targets.default.distDir =
public` — `public/` MUST NOT be committed. (No `publish:local` script exists.)

## App-level JSX pattern (normative, reference: `qcobjects-web-2025`)

Framework repos ship no JSX transform (`tsconfig` has no `jsx` option; zero
`.tsx`/`.jsx` in core/SDK/CLI). Apps MAY still author components as
`src/jsx/*.jsx` under these rules:

- `.jsx` files contain plain JS component classes with template literals and
  `$…()` meta processors (e.g. `$mapper(li,options)` inside `template`) —
  NOT React-style angle-bracket syntax.
- Two-stage build: (1) `build:jsx`: `esbuild src/jsx/*.jsx --bundle
  --outdir=src/js --format=esm --target=es2021 --loader:.js=jsx` (the jsx loader
  permits the extension; markup stays in strings); (2) `build:js`: bundle
  `src/js/*.js` to the served root as usual.
- **React interop is allowed but partial (not demonstrated in the reference app,
  which ships no React dependency):** the same `--loader:.js=jsx` setup accepts
  angle-bracket syntax — esbuild's default classic transform emits
  `React.createElement` calls, so adding React plus a `jsx-factory` decision
  compiles. Interop is NOT full by design: templating differs between the
  frameworks (QCObjects `{{}}` + `$…()` + `.tpl.html` vs React's virtual DOM),
  but React components MAY use QCObjects templates under the hood (e.g. React
  renders a mount shell, QCObjects builds components inside it, or a QCObjects
  template hosts a React root). Either direction MUST own exactly one renderer
  per DOM subtree — never let both frameworks reconcile the same nodes.

## `postversion` rule (normative)

- Default: `"postversion": "git push && git push --tags"`.
- Tag-triggered-publish repos (core, CLI): `"postversion": "git push"` —
  `syncGit` pushes the tag once afterwards (see [05-cli](./05-cli.md)).

## Verification

- Clean `npm ci && npm test && npm run build` passes in core, SDK, and CLI.
- `ls public/cjs public/esm public/browser public/types` all non-empty post-build.
- `prepare` is a silent no-op in tarball installs (no `.git`).
