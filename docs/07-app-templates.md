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
