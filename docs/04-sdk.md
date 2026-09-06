# 04 — SDK (`qcobjects-sdk`)

## Purpose

Specify the SDK: the reusable MVC building blocks apps compose from.

## Scope

Repo `QCObjects/qcobjects-sdk`, npm `qcobjects-sdk` (v2.5.105 at time of writing).
Sources in `src/ts/org.qcobjects.*.ts`, templates in `src/templates`.

## Normative

- The SDK MUST export, at minimum, these modules (CJS + ESM + browser + types):
  `controllers`, `controllers.grid|slider|form|list|swagger`, `views`,
  `components`, `components.grid|list|slider|splashscreen|notifications`,
  `modal.controllers`, `effects`, `tools.canvas|layouts`,
  `i18n_messages`, `models`, `cloud.auth.session.usertoken|data`, and the
  `QCObjects-SDK` bundle.
- Component modules MUST pair a `Component` subclass with its default
  `Controller` and template; grids/lists MUST support paged data sources.
- `controllers.form` MUST provide validation hooks; `controllers.swagger`
  MUST render API docs from an OpenAPI descriptor.
- Cloud-auth session modules MUST store tokens via the core storage cache,
  MUST NOT log or persist raw passwords.
- The SDK MUST depend on `qcobjects` core and MUST NOT depend on
  `qcobjects-cli` or any server code.
- Visual assets live under `src/css` and `src/templates`; class logic MUST NOT
  inline large CSS blobs — reference the stylesheet instead.

## Verification

- Demo app renders each exported component with only core + SDK installed.
- `npm run build` regenerates `build/` + `public/`; `npm test` (eslint + jasmine) green.
