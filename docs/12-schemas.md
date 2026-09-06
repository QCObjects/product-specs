# 12 — Schemas

## Purpose

Pin the shape and resolution rules of `config.json`/`config.yaml` and the export
conventions of `package.json` — with the complete field catalogue migrated from
the template `CONFIG.md`, so it can be summarized without loss.

Sources: `qcobjects-new-app` `CONFIG.md` + `config.json` (`v2.4.40-ts`); core
README §§ CONFIG, Processor, backend settings.
Machine schema: `schemas/config.schema.json`; fixtures: `schemas/examples/*.json`.

## Scope

`config.json` (runtime truth) and `package.json` (distribution contract).

## Config file precedence (normative)

1. `config.json` preferred; else `config.yaml`/`config.yml`.
2. JSON **and** YAML both present → **YAML wins, JSON dismissed**.
3. `CONFIG.set('useConfigService',true)` (or equivalent) enables file-backed settings.

## General fields (normative catalogue)

- `devmode`: `info` | `debug` | `warn` | `error`.
- `autodiscover`, `autodiscover_libs`, `autodiscover_commands`,
  `autodiscover_handlers`: booleans. Absent from `config.json` means the CLI
  built-in defaults apply — which turn ON `autodiscover`, `autodiscover_commands`,
  and `autodiscover_handlers` (only `autodiscover_libs` defaults off). To actually
  disable autoload, the config MUST set the flags `false` explicitly; see the
  autoload contract in [05-cli](./05-cli.md). Templates SHOULD ship explicit
  `false` for every flag they don't need.
- `documentRoot` (e.g. `"$config(projectPath)public/"`), `documentRootFileIndex`
  (default `index.html`), `cacheControl` (e.g. `max-age=31536000`).
- `relativeImportPath` (e.g. `js/packages/`), `componentsBasePath`.
- `serverPortHTTP` / `serverPortHTTPS` (e.g. `'8080'` / `'8443'`;
  `process.env.PORT` overrides HTTP).
- `useLocalSDK` (local vs `sdk.qcobjects.dev`), `useLegacyHTTP`,
  `enableShellCommands` (CLI default `true`; templates SHOULD set `false`
  unless shell commands are required).
- `private-key-pem` / `private-cert-pem` (e.g. `"$config(domain)-privkey.pem"`).
- `domain`, `certificate_provider`, plus server-side `basePath`, `projectPath`,
  `dataPath` (e.g. `/etc/qcobjects/data/`).

## Backend fields (normative catalogue)

- `backend.db_engine{name, databaseName}` — `$ENV(ENGINE_NAME)` /
  `$ENV(DATABASE_NAME)` (e.g. `sqlite3` / `admin.db`).
- `backend.auth{enabled, defaultUser, defaultPasswd, microsoftapikey, googleapikey}`
  — all secret values via `$ENV(...)`.
- `backend.routes[]` — each REQUIRES `name`, `path` (regex), `microservice`;
  MAY carry `description`, `redirect_to`, `responseHeaders`, `cors.allow_origins`.
- `package{source{backend,frontend}, build, dist}` for packaged builds.

## Placeholder resolution (normative)

- `$ENV(VAR)` → environment (Node/CLI/Collab only); missing = boot error naming
  the variable, never silent empty. Two-arg form `$ENV(VAR,default)` falls back
  to `default` (e.g. `"$ENV(DOMAIN,localhost)"`, `"$ENV(DEVMODE,info)"`); empty
  default (`$ENV(OPENAI_API_KEY,)`) means empty string, NOT an error.
- `$config(key)` → sibling key or derived value (`$config(domain)`,
  `$config(projectPath)`) — all environments.
- Custom `$NAME(args)` via `Processor.setProcessor(fn)` (non-arrow; `this` is
  the handler; reach `$ENV` as `this.processors.ENV(arg)`).
- Encrypted `config.json` supported; decoding transparent to `CONFIG.get`.

## `package.json` contract (normative)

Every distributable repo MUST declare `main` (CJS), `module` (ESM), `browser`,
`types`, and an `exports` map covering `.`, `./package.json`, and per-module
subpaths. Dual `cjs`/`esm` + `types` output is mandatory.
Schema changes MUST be backward compatible within a major line or MUST bump the
major and document migration in the spec + changelog.

## Verification

- `npx ajv-cli validate -s schemas/config.schema.json -d <any fixture>` passes
  for every template and test fixture (CI enforces over `schemas/examples/`).
- Booting with an unset `$ENV()` variable fails fast with the variable named.
- `CONFIG.md` field list ⊆ this catalogue (this spec is the superset of record).
