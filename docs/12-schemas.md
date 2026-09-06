# 12 — Schemas

## Purpose

Pin the shape and resolution rules of `config.json` and the export conventions
of `package.json` across the ecosystem.

## Scope

`config.json` (runtime truth) and `package.json` (distribution contract).
Machine-readable schema lives in `schemas/config.schema.json`.

## Normative

- `config.json` MUST be the single runtime source of truth. Required keys:
  `domain`, `documentRoot`, `documentRootFileIndex`, `relativeImportPath`,
  `serverPortHTTP`, `serverPortHTTPS`, `backend.routes[]`.
- Route entries MUST contain `name`, `path` (regex string), `microservice`;
  static routes MUST add `redirect_to`; routes MAY add `responseHeaders`, `cors`.
- Placeholders MUST resolve as: `$ENV(VAR)` → environment variable (missing =
  boot error, never silent empty), `$config(key)` → another config key or
  derived value (e.g. `$config(domain)`, `$config(projectPath)`).
- Boolean behaviour flags (`autodiscover`, `autodiscover_commands`,
  `autodiscover_handlers`, `useLocalSDK`, `useLegacyHTTP`, `enableShellCommands`,
  `devmode`) MUST default to the secure/off value when absent, except
  `documentRootFileIndex` which defaults to `index.html`.
- `package.json` in every distributable repo MUST declare `main` (CJS),
  `module` (ESM), `browser`, `types`, and an `exports` map covering `.`,
  `./package.json`, and per-module subpaths. Dual `cjs`/`esm` + `types`
  output is mandatory.
- Schema changes MUST be backward compatible within a major line or MUST bump
  the major and document migration in the spec + changelog.

## Verification

- `npx ajv-cli validate -s schemas/config.schema.json -d config.json` passes
  for every template and test fixture.
- Booting with an unset `$ENV()` variable fails fast with the variable named.
