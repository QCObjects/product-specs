# 05 — CLI (`qcobjects-cli`)

## Purpose

Specify the CLI: the developer's single entry point for scaffold, serve,
build, and publish.

## Scope

Repo `QCObjects/qcobjects-cli`, npm `qcobjects-cli` (v2.5.158 at time of writing).
Sources in `src/*.ts`, binaries in `bin/` (`qcobjects-cli.js`).

## Normative

- Binaries MUST expose at least: `qcobjects` (main), `qcobjects-server`
  (HTTP/HTTPS/HTTP2), `qcobjects-collab`, `qcobjects-shell`,
  `qcobjects-createcert`, plus GAE server variants.
- Command families in `src/cli-commands*.ts` MUST include: project create/scaffold,
  `serve` (dev, HTTP/2 default), `build` (esbuild + TypeScript declaration paths),
  `publish-static`, `version`, `shell`, and enterprise/collab commands.
  Jira commands are optional extensions, MUST NOT gate core flows.
- Servers (`main-http-server`, `main-http2-server`, `main-http-gae-server`,
  `qcobjects-http2-server`, …) MUST read all behavior from `config.json`:
  ports (`serverPortHTTP/HTTPS`), `documentRoot`, `backend.routes`,
  TLS material via `$config(domain)`-derived filenames.
- Scaffolding templates MUST live in `src/templates/{apps,pwa}` and produce the
  layout specified in [06-app-structure](./06-app-structure.md).
- The CLI MUST support Deno (`mod.ts` + `deno.json`) alongside Node (`bin/`,
  `public/cjs|esm`), with shared `src/` logic.
- `createcert` MUST generate self-signed local TLS material; production
  deployments MUST use externally provisioned certificates.

## Verification

- Fresh `qcobjects create myapp && cd myapp && qcobjects-server` serves the
  PWA shell on the configured ports with zero manual config edits.
- `npm test` (eslint + jasmine) green; `tsc` declaration build emits `public/types`.
