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

## Built-in commands, handlers, and libs (normative)

The framework ships a minimal set of built-ins that are always available
without installing extra packages. All other capabilities enter via the
keyword autoload contract ([05-cli](./05-cli.md) § Handlers/plugins/commands autoload,
[16-addons](./16-addons.md)).

- **Built-in commands** — two tiers, all shipping in-repo:
  - `cli-main.ts` `choiceOption`: `create`, `publish` (STUB — logs
    `"publish is not yet implemented"`, ignores flags), `upgrade-to-enterprise`,
    `generate-sw`, `launch` (ignores its `<appname>` argument; serves CWD after
    a 5s delay). Sub-flags: `--pwa`, `--amp`, `--php`, `--custom`
    (takes a value ONLY on `create`; valueless on `publish`), `--tests`
    (accepted but silently ignored — no-op).
  - In-repo families via `cli-commands.ts`: `v-major/v-minor/v-patch/v-sync/
    v-changelog` (version), `jira`, `publish:static`, `build:typescript`
    (`build:ts`), `build:esbuild` (`build:esb`); `upgrade-to-enterprise` wired
    directly in `cli-main.ts`. `collab` is NOT a commander family — it is a
    separate binary entry (`qcobjects-collab.ts` → `collab-server.ts`).

- **Built-in handler records** (`com.qcobjects.backend.microservice.static`):
  `defaultsettings.ts` APPENDS three static routes unconditionally (concat,
  not gated on empty `backend.routes`) at every boot:
  `^/QCObjects.js$` → core `src/QCObjects.js`,
  `^/js/packages/QCObjects-SDK.js$` → SDK `src/QCObjects-SDK.js`,
  `^/qcobjects-sdk/(.*)$` → SDK tree — all CORS `*`. These are route RECORDS
  naming the static microservice; no `BackendMicroservice` subclass is defined
  in this repo. Use them for framework-asset serving instead of custom code.

- **Core libraries** (always present as peer dependencies):
  `qcobjects` (core framework) and `qcobjects-sdk` (controllers, views,
  components, effects, cloud auth, i18n). These are NOT autoloaded — they
  are hard peer dependencies of every QCObjects app and CLI command.

- **No other built-in handlers, libs, or commands exist.** Any additional
  capability (payment handlers, email libs, admin panels, custom commands)
  MUST enter via the autoload keyword contract (`qcobjects-handler`,
  `qcobjects-lib`, `qcobjects-command`, `qcobjects-admin-lib`) or explicit
  `require`/`import` in app code.

## Custom templates (`create --custom`, normative)

Source: `src/cli-main.ts` (`choiceOption.create`, `copyTemplate`), pinned at
`https://github.com/QCObjects/qcobjects-cli/blob/v2.5.158/src/cli-main.ts`.

- **Flags:** `create <appname>` resolves the template package by flag:
  `--amp` → `qcobjects-ecommerce-amp`, `--pwa` (or no flag) → `qcobjectsnewapp`,
  `--php` → `qcobjectsnewphp`, `--custom <templateappname>` → any npm package
  name, `--tests` → test suite. `publish` mirrors the same flags.
- **Flow (binding):** `npm init -y` → `npm i --save-dev --legacy-peer-deps
  <template>` → adopt the template's `package.json` (renamed to `<appname>`,
  version reset to `1.0.0` via direct mutation, `repository` cleared) →
  `copyTemplate()` from the installed package dir into the project (excluding
  `package.json`, `node_modules`, `.DS_Store`) → `npm uninstall <template>
  --save` + `npm install qcobjects-cli` + full `npm i --legacy-peer-deps` +
  `npm cache verify` → tail: `qcobjects-createcert`, fetch of `.gitignore`
  from GitHub, `git init`.
- **Key consequence:** the template package is scaffolding only — installed,
  copied, then UNINSTALLED. Apps MUST NOT retain a runtime dependency on their
  template package; all cohesion lives in the copied files
  (see [06-app-structure](./06-app-structure.md)).
- **Authoring custom templates:** any npm package with the app layout
  ([06-app-structure](./06-app-structure.md)) + a `package.json` works as a
  `--custom` template (`options.createCustom` is used verbatim as the npm name —
  no naming constraint is enforced by tooling). Template packages MUST carry
  `-template` as a SUFFIX by project convention (review-enforced, not
  tool-gated) — `qcobjects-<name>-template` (e.g. `qcobjects-app-template`) —
  and MUST declare the layout they stamp in their README. Kind-specific starters
  keep their kind infix: `qcobjects-handler-<name>-template`,
  `qcobjects-lib-<name>-template`, `qcobjects-command-<name>-template`.
- **Beyond apps — custom commands/libs/handlers:** `copyTemplate` copies the
  whole package dir, so `--custom` templates MAY stamp any package kind, not
  just apps: a command starter (class in a `com.qcobjects.cli.commands.*`
  package ending in `CommandHandler`, picked up by `getPluginCommandsList()` and
  constructed with `{switchCommander}`), a lib starter (`qcobjects-lib`
  keyword), or a handler starter (`qcobjects-handler` keyword, microservice
  skeleton). The stamped package then follows the autoload contract
  (§ Handlers/plugins/commands autoload) and the add-on lifecycle
  ([16-addons](./16-addons.md)). Prefer stamping starters over documenting
  manual file creation.

## Binaries (normative)

Single dispatcher: `bin/qcobjects-cli.js` serves all 10 `package.json` bin
aliases (`qco`, `qcobjects`, `qcobjects-cli`, `qcobjects-server`,
`qcobjects-http-server`, `qcobjects-http2-server`, `qcobjects-gae-server`,
`qcobjects-shell`, `qcobjects-collab`, `qcobjects-createcert`) via an
`entryMap` dispatch on the invoked basename. There are no sibling binaries.

## Servers (normative)

- Implementations: `main-http-server.ts` (HTTP legacy), `main-http2-server.ts`,
  `main-http-gae-server.ts` (App Engine), entered via `qcobjects-http-server.ts` /
  `qcobjects-http2-server.ts` / `qcobjects-gae-http-server.ts`. NOTE: the
  `qcobjects-http2-server` entrypoint picks `HTTPServer` vs `HTTP2Server` on
  `useLegacyHTTP` — it is not always HTTP/2.
- ALL behavior from `config.json`: ports (`serverPortHTTP/HTTPS`),
  `documentRoot`, `backend.routes`, TLS via `$config(domain)`-derived filenames.
- `process.env.PORT` overrides the listen port ONLY on the legacy HTTP and GAE
  servers; the default HTTP/2 `start()` ignores `PORT`.
- local `config.json` at CLI root is gitignored dev-only (default devmode
  `$ENV(DEVMODE,info)` → `info`); `$ENV(VAR)` templates resolve in `defaultsettings.ts`.
- Production recommendation: HTTP/2 server on Ubuntu 18.x+ with NodeJS 12.x+.

## Commands & internals (normative)

- Framework: Commander — `SwitchCommander` (`cli-main.ts`); in-repo families in
  `cli-commands*.ts` (build-esbuild, build-typescript, jira, publish-static,
  version) re-exported via `cli-commands.ts`; `upgrade-to-enterprise` wired
  directly in `cli-main.ts`; `collab` is a separate binary, not a family.
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
  ON `autodiscover` (`defaultsettings.ts`), so with shipped defaults ALL kinds
  — including libs — autoload with NO opt-in. `autodiscover_libs` only matters
  when an app explicitly sets master `autodiscover:false`: it re-enables libs
  alone. Production configs SHOULD set exactly the flags they need and `false`
  for the rest (least privilege: every auto-imported package runs code at boot).
- **Order/concurrency:** libs → handlers → commands → devCommands fire as four
  independent non-awaited chains — NO guaranteed order.
- **Failure semantics:** ALL FOUR chains attach warn-and-continue catches
  (including commands) — a broken `qcobjects-command` logs a warning and boot
  continues; nothing aborts boot. (The inner rethrow in `loadCommands` is
  swallowed by the outer catch.)
- **Registry:** `CONFIG.backend` publishes `libs`, `handlers`, `commands`,
  `devCommands` — possibly still EMPTY at read time (fire-and-forget loaders).
  The raw `dependencies`/  `devDependencies` name lists are NEVER published
  (their memo closures return `[]` permanently — dead code). Introspection MUST
  read the four kind keys and tolerate emptiness, never re-scan `node_modules`.
  (`backend.plugins = commands + devCommands` is also written, but races the
  fire-and-forget loaders — do not rely on it at boot.)
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
- **Single-push rule:** `syncGit` pushes exactly once per release either way —
  with `--npm` the tag is created by `npm version` (manual `git tag -a`
  skipped), without `--npm` by `git tag -a` — then one `git push && git push
  --tags`. There is no double tag push in current code.
  GitHub Actions (tag-triggered publish) repos MUST still set
  `"postversion": "git push"` (branch only) to keep release pushes minimal.
- Tests: jasmine 3.7, single `spec/testsSpec.ts` asserting `qcobjects` version
  parity between `peerDependencies` and `devDependencies`; SDK mocked via
  `tsconfig.jasmine.json` mapping; `stopSpecOnExpectationFailure`,
  `failSpecWithNoExpectations`, `random:false`.
- Lint is permissive (`recommendedTypeChecked` with core `no-unsafe-*`,
  `no-explicit-any`, `no-unused-vars` off); ignores cover `**/*.js`,
  `spec/**/*`, `src/*.js`, `src/**/*.js`, `node_modules…`.
  (The `tsconfig.jasmine.json` SDK-mock mapping points at
  `spec/mocks/qcobjects-sdk.mock.ts`, which does NOT exist — dangling.)

## Verification

- Fresh `qcobjects create myapp --pwa && qcobjects-server` serves the PWA shell
  on configured ports with zero manual edits.
- `npm test` (eslint + jasmine) green; `tsc` declaration build emits `public/types`.
- `v-patch --git` (no `--npm`) produces exactly one tag push via `git tag -a`
  (single push, no duplicate CI).
