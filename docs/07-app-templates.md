# 07 — App-Templates Approach

## Purpose

Specify how app templates (`qcobjects-new-app`, CLI `templates/pwa|apps`)
deliver the out-of-box experience — with the complete template contract.

Sources: `qcobjects-new-app` README + `CONFIG.md` + scripts (`v2.4.40-ts`).
Definitions below are authoritative.

## Scope

Template repos and the CLI scaffolding that stamps them out. PWA runtime
semantics: [06-app-structure](./06-app-structure.md); widgets: [04-sdk](./04-sdk.md).

## Template contract (normative)

- Templates MUST be runnable immediately after `npm install`:
  `npm start` (cert + serve) with no extra configuration.
- Templates MUST demonstrate, at minimum: boot (`init`), one data-bound
  component, one backend route call, offline shell (service worker), and
  one form with validation.
- Template `config.json` MUST use `$ENV(...)` for every secret/host and
  `$config(...)` for derived paths; a committed `.env.example` MUST document
  each variable.
- CLI `src/templates/pwa` is the minimal shell; `src/templates/apps` are
  fuller starters. Both MUST track the [06-app-structure](./06-app-structure.md)
  layout; any layout change MUST update templates in the same release.
- Templates MUST pin `qcobjects` + `qcobjects-sdk` to a tested minor range
  and MUST be re-verified (install + serve + build) on every core/SDK minor bump.
- `qcobjects-new-app` doubles as the integration testbed: demo routes under
  `demo-tests/` MAY exist there but MUST NOT leak into `src/templates`.
- CSS theme matrix ships with every template:
  `css/theme/{basic,cyan,redlight,xtra}` + `desktop/` + `mobile/` variants;
  new themes MUST follow the same directory shape.
- Template hero/pages components (`templates/components/{hero,pages}`) MUST
  keep `name` ↔ `*.tpl.html` file correspondence (`tplextension: tpl.html`).

## Template catalogue (normative)

| Template (npm) | `create` flag | Source | Purpose | Status |
|---|---|---|---|---|
| `qcobjectsnewapp` (`v2.4.40-ts`) | `--pwa` / default | `QuickCorp/qcobjects-new-app` (public) | Reference PWA starter + integration testbed (`demo-tests/`) | stable, canonical |
| `qcobjects-ecommerce-amp` (`v0.0.7`) | `--amp` | private GitLab | AMP storefront starter | stable |
| `qcobjectsnewphp` (`v1.0.35`) | `--php` | private GitLab | PHP-backend PWA starter | stable |
| CLI `src/templates/pwa` + `src/templates/apps` | built-in | `qcobjects-cli` repo | Minimal embedded shell (`sw.js`, `spa-local.*`) — fallback when npm is unreachable | stable |
| `create-qcobjects` (`v2.0.13`) | `npx` initializer | `QCObjects/create-qcobjects` (private) | Standalone creation tool | stable |
| any npm package | `--custom <name>` | author-provided | Custom layouts per [05-cli](./05-cli.md) § Custom templates | stable mechanism |
| `QCObjects-App-Templates/*` boilerplates | `--custom <name>` | `QCObjects-App-Templates` org (public): `qcobjects-swipper-template` (swiper/slider showcase), `qcobjects-boilerplate-{pwa,tailwind,tabs-spa,dashboard,hello-world}-template` (all `v1.0.0`) | Minimal starters by concern | stable, convention-compliant |

- New official templates MUST enter this table (flag, source, purpose, status)
  in their release PR and MUST satisfy the Template contract above.
- Private-source templates MUST still publish versioned npm tarballs so
  `create` works without repo access; their sources MAY stay private.
- `--custom` names MUST use the `-template` suffix convention
  (`qcobjects-<name>-template`, kind infixes preserved:
  `qcobjects-handler-<name>-template`, etc. — see [05-cli](./05-cli.md)).
  The `QCObjects-App-Templates/*` boilerplates were renamed into compliance
  (bare names → `-template` suffix); GitHub redirects preserve old URLs.

## CSS framework interoperability (normative)

The framework is CSS-agnostic: it ships plain CSS (SDK `src/css`, template
`css/` matrix) and composes with any CSS system at two layers.

- **Light DOM (page shell, non-shadowed components):** any global stylesheet
  works unchanged — link Foundation, Bootstrap, Tailwind builds, or hand CSS in
  `index.html` as usual (reference demos exist for Foundation, Materialize, and
  raw CSS).
- **Shadow DOM (shadowed components):** page CSS cannot cross the boundary —
  each shadowed template MUST carry its own `<style>` importing what it needs
  (`<style>@import url("css/components/….css")</style>`); chained imports
  (e.g. a component CSS importing a compiled Tailwind build) resolve inside the
  shadow root (see [03-core-framework](./03-core-framework.md) § Component
  authoring rules).
- **Preprocessors (SCSS/Sass, Tailwind, PostCSS):** build-time concerns owned by
  the app, NOT the framework — no framework package depends on them. Apps MAY
  compile `scss/ → css/` and Tailwind sources into `src/css` before the standard
  build (reference: `qcobjects-web-2025` runs `sass` + `tailwindcss` ahead of
  `build:assets`); compiled output MUST land in the served CSS tree, never
  source `.scss` files.
- **Theme matrix:** every template ships `css/theme/{basic,cyan,redlight,xtra}`
  + `desktop/` + `mobile/` variants and `css/components/` per-component styles;
  new themes MUST follow the same directory shape. Switching themes MUST be a
  CSS swap only — no component or template changes.

## Configuration precedence (normative, from template README + CONFIG.md)

1. Open `config.json` (or `config.yaml`/`config.yml`).
2. If JSON **and** YAML both present → **YAML wins, JSON dismissed**.
3. Field meanings per `CONFIG.md` (migrated in full to [12-schemas](./12-schemas.md)):
   General (`devmode`: info|debug|warn|error; `autodiscover[_commands|_handlers]`;
   `documentRoot` e.g. `"$config(projectPath)public/"`; `documentRootFileIndex`;
   `cacheControl`; `relativeImportPath`; `serverPortHTTP/HTTPS`; `useLocalSDK`;
   `useLegacyHTTP`; `private-key-pem`/`private-cert-pem` e.g.
   `"$config(domain)-privkey.pem"`; `enableShellCommands`),
   Backend (`db_engine{name,databaseName}`, `auth{enabled,defaultUser,defaultPasswd,
   microsoftapikey,googleapikey}`, `routes[]{name,description,path,microservice,
   redirect_to,responseHeaders,cors.allow_origins}`),
   Package (`package{source{backend,frontend},build,dist}`).

## Verification

- CI stamps each template into a temp dir, runs install + serve smoke test +
  build, and all three succeed.
- `npm audit` on a freshly stamped template shows zero critical vulnerabilities.
- `schemas/examples/app-config.json` (derived from the template config) validates.
