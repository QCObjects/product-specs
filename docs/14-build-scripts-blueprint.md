# 14 — Build Scripts Blueprint

## Purpose

Pin the build/test/lint pipelines so every repo compiles, types, and ships
the same three distributions.

## Scope

npm `scripts` in core, SDK, CLI, and templates. CI execution order is in
[08-ci-conventions](./08-ci-conventions.md).

## Normative

- Every distributable repo MUST implement: `lint` (eslint, zero warnings),
  `test` = `lint` + jasmine suite, `build` = types → code → browser bundle.
- Canonical chains (observed, now binding):
  - core: `build:ts-types` (`tsc -p tsconfig.d.json`) → `build:ts` (coverage + `tsc`)
    → `build:browser` (`node ./build-esbuild.js`) → `postbuild` (`postbuild.js`).
  - cli: `build:ts-types` (`transpile.js tsconfig.d.json`) → `build:ts`
    (`npm test` + `transpile.js tsconfig.json`) → `build:esbuild`
    (`build-esbuild-esm.js`).
- Outputs MUST be: `public/cjs` (require), `public/esm` (import),
  `public/browser` (bundle), `public/types` (declarations) — matching the
  `exports` map in [03-core-framework](./03-core-framework.md).
- `transpile.js` / `build-esbuild*.js` scripts MUST be committed in-repo and
  MUST NOT fetch remote toolchains at build time (hermetic builds).
- `preversion` MUST verify cache + tests; `postversion` MUST push commits and
  tags. Version numbers MUST be cut with `qcobjects v-patch|v-minor|v-major`.
- Templates (parcel-based) MUST keep `targets.default.distDir = public` and
  MUST NOT commit `public/` output.

## Verification

- Clean `npm ci && npm test && npm run build` passes in core, SDK, and CLI.
- `ls public/cjs public/esm public/browser public/types` all non-empty post-build.
