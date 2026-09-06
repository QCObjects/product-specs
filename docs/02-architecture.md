# 02 — Architecture

## Purpose

Pin the layered architecture shared by every QCObjects repo and app.

## Scope

Layers, their responsibilities, and the contracts between them.
Component-level detail lives in 03–05.

## Normative

The stack MUST be read bottom-up as five layers:

1. **Core (`qcobjects` npm package)** — class system (`Class`, `InheritClass`,
   `ClassFactory`, `New`), MVC primitives (`Component`, `Controller`, `View`,
   `VO`, `DDO`), packaging (`Package`, `Import`/`Export`), routing
   (`routings.ts`), and loaders (component, service, SDK). No transport, no CLI.
2. **SDK (`qcobjects-sdk`)** — elementary Controllers, Views, Components
   (grid, list, slider, splashscreen, notifications, modal, i18n, effects,
   cloud-auth session). Depends on core; MUST NOT depend on the CLI.
3. **CLI / Runtime (`qcobjects-cli`)** — scaffolding (`templates/apps`,
   `templates/pwa`), dev servers (HTTP, HTTPS, HTTP/2, GAE variants),
   `qcobjects-server` / `qcobjects-collab` / `qcobjects-shell` entry points,
   build commands (esbuild, TypeScript), publish-static. Orchestrates; does
   not implement UI widgets.
4. **Handlers / Microservices** — backend route targets addressed by name in
   `config.json` (e.g. `com.qcobjects.backend.microservice.static`), plus
   language bridges (PHP handler today; Wasm / FastAPI per v3.2+).
   MUST be independently installable npm packages, auto-discovered by keyword.
5. **Apps (`qcobjects-new-app` and derivatives)** — PWA shell (`index.html`,
   `manifest.json`, `sw.js`), `src/js/{config,init,packages}`, static assets.
   Apps consume layers 1–3; MUST NOT fork them.

Cross-cutting rules:

- Layer N MAY depend on layers below it, MUST NOT depend on layers above it.
- All runtime behavior MUST resolve through `CONFIG` + `config.json`
  (`relativeImportPath`, `componentsBasePath`, `documentRoot`, `backend.routes`).
- Browser, ESM, and CJS distributions MUST all be published
  (`public/browser`, `public/esm`, `public/cjs` + `public/types`).

## Verification

- `npm ls` in any app shows exactly one `qcobjects` + one `qcobjects-sdk` copy.
- Deleting a handler package degrades only its routes; the server still boots.
