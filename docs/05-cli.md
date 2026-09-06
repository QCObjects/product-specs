# 05 — CLI (`qcobjects-cli`)

## Purpose

Specify the CLI: the developer's single entry point for scaffold, serve,
build, publish, and version — with the complete command and server catalogue,
so the CLI README and repo docs can be summarized without loss.

Sources: `qcobjects-cli` README + repo structure (`v2.5.158`); core README
§§ HTTP2 server, CLI tool.
Code pins: `https://github.com/QCObjects/qcobjects-cli/blob/v2.5.158/src/<file>.ts`.
Definitions below are authoritative.

## Scope

Repo `QCObjects/qcobjects-cli`, npm `qcobjects-cli` (`v2.5.158`).
Sources in `src/*.ts`, binaries in `bin/` (`qcobjects-cli.js`).
Node >= 22, npm >= 10; install with `npm i --legacy-peer-deps`.

## Service model (normative)

- Server settings file: `/etc/qcobjects/config.json` (service installs).
- `service qcobjects status` | `start` | `stop` | `restart`.
- Scaffold: `qcobjects create mynewapp --pwa` | `--amp`; serve via
  `qcobjects launch mynewapp` or `qcobjects-server` (serves CWD over HTTP/2
  with default config).
- Built-in command surface (`qcobjects [options] [command]`):
  `create <appname>`, `publish <appname>`, `generate-sw <appname>`,
  `launch <appname>`; `-V/--version`, `-h/--help`; per-command help via
  `qcobjects-cli [command] --help`.

## Binaries (normative)

`qcobjects` (main) MUST exist alongside: `qcobjects-server`
(HTTP/HTTPS/HTTP2), `qcobjects-collab`, `qcobjects-shell`,
`qcobjects-createcert`, plus GAE server variants (`main-http-gae-server`).

## Servers (normative)

- Implementations: `main-http-server.ts` (HTTP), `main-http2-server.ts`
  (HTTP/2, default for `serve`), `main-http-gae-server.ts` (App Engine),
  entered via `qcobjects-http-server.ts` / `qcobjects-http2-server.ts` /
  `qcobjects-gae-http-server.ts`.
- ALL behavior from `config.json`: ports (`serverPortHTTP/HTTPS`),
  `documentRoot`, `backend.routes`, TLS via `$config(domain)`-derived filenames.
- `process.env.PORT` overrides the HTTP listen port.
- local `config.json` at CLI root is gitignored dev-only (default
  `{"devmode":"debug"}`); `$ENV(VAR)` templates resolve in `defaultsettings.ts`.
- Production recommendation: HTTP/2 server on Ubuntu 18.x+ with NodeJS 12.x+.

## Commands & internals (normative)

- Framework: Commander — `SwitchCommander` (`cli-main.ts`); families in
  `cli-commands*.ts` (build-esbuild, build-typescript, jira, publish-static,
  version, enterprise, collab), registered via `cli-commands.ts`.
- Modules import `qcobjects` and use `InheritClass`, `Package()`, `Export()`,
  `CONFIG`, `logger`, `Component`, `Service` (source convention, binding).
- Plugin autodiscovery: see "Handlers/plugins/commands autoload" below.
- Entrypoints: `qcobjects-cli.ts`, `qcobjects-http{-2,}-server.ts`,
  `qcobjects-shell.ts`, `qcobjects-collab.ts`; Deno via `deno.json` + `mod.ts`.
- `createcert` generates self-signed local TLS; production MUST use external certs.

## Handlers/plugins/commands autoload (normative)

Source: `src/defaultsettings.ts` (`__load_default_settings__`, runs at CLI boot;
`__reset_settings__` re-runs it), pinned at
`https://github.com/QCObjects/qcobjects-cli/blob/v2.5.158/src/defaultsettings.ts`.

- **Scan:** `<projectPath>/package.json` `dependencies` (and `devDependencies`
  for dev commands) are read; each installed package's own `package.json`
  `keywords` are inspected (cached per package) for `qcobjects-lib`,
  `qcobjects-handler`, `qcobjects-command`. Matches are `import()`ed via
  `findPackageNodePath` resolution.
- **Flags:** master `autodiscover` OR per-type `autodiscover_libs`,
  `autodiscover_handlers`, `autodiscover_commands`. CLI built-in defaults turn
  ON `autodiscover`, `autodiscover_commands`, `autodiscover_handlers`
  (`defaultsettings.ts` lines ~100-102) — so autoload is active unless the app
  `config.json` explicitly sets them `false`. `autodiscover_libs` has NO built-in
  default: libs load only with explicit opt-in. Production configs SHOULD set
  exactly the flags they need and `false` for the rest (least privilege:
  every auto-imported package runs code at boot).
- **Order:** libs → handlers → commands → devCommands, each as `Promise.all`
  over dynamic imports.
- **Failure semantics:** lib/handler load errors warn and continue
  (`An error ocurred loading libs/handlers`); command load errors are FATAL
  (logged, rethrown — boot aborts). A broken `qcobjects-command` dependency
  therefore blocks server start by design.
- **Registry:** discovered lists are published under `CONFIG.backend` as
  `libs`, `handlers`, `commands`, `devCommands`, plus the raw `dependencies` /
  `devDependencies` name lists; `backend.plugins = commands + devCommands`.
  Introspection MUST read these keys, never re-scan `node_modules`.
- **Publishing contract:** a handler/plugin/command package MUST declare its
  role in `package.json` `keywords` (`qcobjects-handler`, `qcobjects-command`,
  or `qcobjects-lib`) or it will never load, no matter the flags.

## Synced semantic versioning (normative, from CLI README)

Version lives in the `VERSION` file; commands sync it to `package.json`/git:

- `v-patch` (`1.2.3→1.2.4`), `v-minor`, `v-major` — same options:
  `--sync-git/--git` (commit+tag+push), `--sync-npm/--npm` (also `npm version`,
  implies `--git`), `--commit-msg/-m`.
- `v-sync` — adopt `git describe` (latest tag) into `VERSION` + `package.json`,
  commit, tag, push (`-m` default `Synced Version v<version>`).
- `v-changelog` — changelog from annotated tags grouped by minor → stdout
  (`v-changelog > CHANGELOG.md`).
- Typical flow: `v-patch --git --npm -m "msg"` → CI publishes → `v-changelog`.
- **Duplicate-push rule:** `v-* --git --npm` runs `npm version` (fires
  `preversion`/`postversion`) AND `syncGit` pushes again — the tag pushes twice.
  GitHub Actions (tag-triggered publish) repos MUST set
  `"postversion": "git push"` (branch only; `syncGit` pushes the tag once).
  Without `--npm`, no `npm version` runs: single commit, single tag push.
- Tests: jasmine 3.7, single `spec/testsSpec.ts` asserting `qcobjects` version
  parity between `peerDependencies` and `devDependencies`; SDK mocked via
  `tsconfig.jasmine.json` mapping; `stopSpecOnExpectationFailure`,
  `failSpecWithNoExpectations`, `random:false`.
- Lint is permissive (`recommendedTypeChecked` with core `no-unsafe-*`,
  `no-explicit-any`, `no-unused-vars` off); ignores `**/*.js`, `spec/**`.

## Verification

- Fresh `qcobjects create myapp --pwa && qcobjects-server` serves the PWA shell
  on configured ports with zero manual edits.
- `npm test` (eslint + jasmine) green; `tsc` declaration build emits `public/types`.
- `v-patch --git` (no `--npm`) produces exactly one tag push (no duplicate CI).
