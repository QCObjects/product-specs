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

1. The CLI runtime reads ONLY `config.json` (`defaultsettings.ts`, `JSON.parse`
   on `<projectPath>/config.json`) — no YAML branch exists anywhere in `src/`.
   A shipped `config.yaml` is INERT until a YAML loader lands; since the
   template ships both files with identical content, JSON silently governs.
2. `CONFIG.set('useConfigService',true)` (or equivalent) enables file-backed settings.
3. Config files are STANDALONE — no overlay/merging mechanism exists. Minimal
   configs (`{"autodiscover":true}`, `{"documentRoot":"…browser/"}`) are valid;
   absent keys fall back to built-ins (`documentRoot` → `<cwd>/public`,
   ports/TLS required only when serving). The schema accordingly requires
   NOTHING — every key is optional with documented fallbacks.

## General fields (normative catalogue)

- `devmode`: `info` | `debug` | `warn` | `error`.
- `autodiscover`, `autodiscover_libs`, `autodiscover_commands`,
  `autodiscover_handlers`: booleans. Absent from `config.json` means the CLI
  built-in defaults apply — which turn ON `autodiscover`, `autodiscover_commands`,
  and `autodiscover_handlers` (only `autodiscover_libs` defaults off). To actually
  disable autoload, the config MUST set the flags `false` explicitly; see the
  autoload contract in [05-cli](./05-cli.md). (Drift on record: the canonical
  template ships all three as `true` with no `autodiscover_libs` key — maximal
  autoload. New apps SHOULD still least-privilege to explicit `false`.)
- `documentRoot` (e.g. `"$config(projectPath)public/"`), `documentRootFileIndex`
  (default `index.html`), `cacheControl` (e.g. `max-age=31536000`).
- `relativeImportPath` (e.g. `js/packages/`), `componentsBasePath`.
- `serverPortHTTP` / `serverPortHTTPS` (e.g. `'8080'` / `'8443'`;
  `process.env.PORT` overrides ONLY on legacy/GAE servers — the default HTTP/2
  `start()` ignores it).
- `useLocalSDK` (local vs `sdk.qcobjects.dev`), `useLegacyHTTP`,
  `enableShellCommands` (CLI default `true`; templates SHOULD set `false`
  unless shell commands are required).
- `useTemplate` (CLI default `false`): `true` enables server-side rendering of
  `.html`/`.tpl.html` through `FileDispatcher` (see [02-architecture](./02-architecture.md)
  § Rendering model).
- `private-key-pem` / `private-cert-pem` (e.g. `"$config(domain)-privkey.pem"`).
- `domain`, `certificate_provider`, plus server-side `basePath` (read for chdir,
  set only via config — never defaulted), `projectPath` (defaults to cwd).
  (`dataPath`, e.g. `/etc/qcobjects/data/`, appears in examples but in NO
  runtime read path — do not rely on it.)

## Backend fields (normative catalogue)

- `backend.db_engine{name, databaseName}` — `$ENV(ENGINE_NAME)` /
  `$ENV(DATABASE_NAME)` (e.g. `sqlite3` / `admin.db`).
- `backend.auth{enabled, defaultUser, defaultPasswd, microsoftapikey, googleapikey}`
  — all secret values via `$ENV(...)`.
- `backend.routes[]` — each REQUIRES `name`, `path` (regex), `microservice`;
  MAY carry `description`, `redirect_to`, `responseHeaders`, `cors.allow_origins`,
  and a sibling `headers` key (used by real template routes, distinct from
  `responseHeaders`).
- `backend.interceptors[]` — server-lifecycle plugins, loaded by all three
  servers at boot: each entry imports its `microservice` package, instantiates
  `<microservice>.Interceptor` with `{domain, basePath, projectPath,
  interceptor, server}` (note the live SERVER handle — this is how socket.io
  and similar layers attach), and pushes the instance to
  `interceptorInstances`. Entries carry `name`, `description`, `microservice`,
  `responseHeaders` (proof: video-streaming `Start Streaming` entry).
- **Unknown keys pass through inert — with one verified exception.** `config.json`
  MAY carry app-private keys (e.g. top-level `iceServers` for WebRTC STUN);
  `CONFIG.get` serves any key, but the server acts ONLY on keys it reads.
  The exception: route-level `supported_methods` IS consumed — by the static
  microservice (`src/backend/backend-microservice-static.ts`), which allows
  delivery only when the request method is `"*"`-matched or case-insensitively
  listed (absent = allowed). Do NOT assume any OTHER unknown key takes effect;
  verify the read site first.
- `package{source{backend,frontend}, build, dist}` for packaged builds.

## Placeholder resolution (normative)

- `$ENV(VAR)` → environment (Node/CLI/Collab only); a missing one-arg var
  resolves to EMPTY STRING silently (the CLI `ENV` shim returns `""`) — never a
  boot error. Two-arg form `$ENV(VAR,default)` falls back
  to `default` (e.g. `"$ENV(DOMAIN,localhost)"`, `"$ENV(DEVMODE,info)"`); empty
  default (`$ENV(OPENAI_API_KEY,)`) means empty string.
- `$config(key)` → sibling key or derived value (`$config(domain)`,
  `$config(projectPath)`) — all environments.
- Custom `$NAME(args)` via `Processor.setProcessor(fn)` (non-arrow; `this` is
  the handler; reach `$ENV` as `this.processors.ENV(arg)`).
- Encrypted `config.json` supported; decoding transparent to `CONFIG.get`.

## `package.json` contract (normative)

Framework packages MUST declare `main` (CJS), `module` (ESM), `browser`,
`types`, and an `exports` map covering `.`, `./package.json`, and per-module
subpaths. Dual `cjs`/`esm` + `types` output is mandatory.
(KNOWN VIOLATIONS at `v2.5.158`: five entries point at `./public/mjs/*.mjs`
but `public/mjs/` is never produced — fix to `./public/esm/`; `"./types/*"`
maps a non-existent top-level `types/` — real declarations are the single
`public/types/index.d.ts`. The app template has NO `exports` map at all —
the contract binds framework packages until the template complies.)
Schema changes MUST be backward compatible within a major line or MUST bump the
major and document migration in the spec + changelog.

## Verification

- `npx ajv-cli validate -s schemas/config.schema.json -d <any fixture>` passes
  for every template and test fixture (CI enforces over `schemas/examples/`).
- A `filename` with multiple dots (e.g. `my.page.html`) is misclassified by
  `file_extension()` (first-dot, not last) — keep template basenames single-dot.
- `CONFIG.md` field list ⊆ this catalogue (this spec is the superset of record).
