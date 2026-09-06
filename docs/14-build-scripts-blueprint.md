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
  (`build-esbuild-esm.js` → ESM + browser IIFE).
- Mechanism: custom `transpile.js` (TypeScript compiler API) emits CJS;
  `build-esbuild*.js` bundles ESM + browser IIFE. Scripts MUST be committed
  in-repo and MUST NOT fetch remote toolchains (hermetic builds).
- Outputs MUST be: `public/cjs` (require), `public/esm` (import),
  `public/browser` (bundle), `public/types` (declarations) — matching the
  `exports` map in [03-core-framework](./03-core-framework.md).

## Script tables (normative — every repo MUST keep these names/meanings)

Core/SDK/CLI shared: `build`, `build:ts`, `build:ts-types`, `build:browser`,
`build:esbuild`, `start` (`qcobjects-shell`), `test:ts-types`
(`tsc -p tsconfig.jasmine.json`), `test:jasmine` (ts-node + jasmine),
`test` (`lint` + jasmine), `lint` (eslint `src/**/*.ts --fix`),
`coverage` (core: nyc lcov+text over `npm run test`), `preversion`
(`npm cache verify` + tests/coverage), `postversion` (push branch AND tags —
except tag-triggered-publish repos, see below), `sync`
(`git add . && git commit -am`), `v-patch|v-minor|v-major` (via `qcobjects`),
`qcobjects|cli` (passthrough), `prepare` (husky install, no-op outside git),
`cli:help`, `tree`, `generate-readme-pdf` (markdown-pdf Letter → README.pdf +
README-es.pdf, then uninstall).

App template deltas (`qcobjects-new-app`): `test` = eslint + jasmine;
`start` = `createcert` + `serve`; `start:dev` (watch); `serve`/`server`,
`collab`, `shell`, `createcert`, `http-server`, `gae-server`,
`build`/`build:ts` (TS→JS), `publish:local`; parcel `targets.default.distDir =
public` — `public/` MUST NOT be committed.

## `postversion` rule (normative)

- Default: `"postversion": "git push && git push --tags"`.
- Tag-triggered-publish repos (core, CLI): `"postversion": "git push"` —
  `syncGit` pushes the tag once afterwards (see [05-cli](./05-cli.md)).

## Verification

- Clean `npm ci && npm test && npm run build` passes in core, SDK, and CLI.
- `ls public/cjs public/esm public/browser public/types` all non-empty post-build.
- `prepare` is a silent no-op in tarball installs (no `.git`).
