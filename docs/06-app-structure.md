# 06 — Resulting App Structure

## Purpose

Pin the layout every QCObjects app MUST follow — with the complete boot
sequence, script catalogue, and environment contract — so tooling, docs, and
deployments can assume it.

Sources: `qcobjects-new-app` README + `config.json` + `package.json`
(`v2.4.40-ts`); core README §§ Start Coding (5 steps).
Code pins: `https://github.com/QCObjects/qcobjects-new-app/blob/v2.4.40-ts/<path>`.
Definitions below are authoritative.

## Scope

As produced by `qcobjects create` / `src/templates/*` and exemplified by
`qcobjects-new-app` (`v2.4.40-ts`).

## Layout (normative)

```
myapp/
  config.json          # runtime truth: domain, ports, routes, paths ($ENV/$config).
                       # NOTE: the CLI loads ONLY config.json — a shipped
                       # config.yaml is inert (see § Production patterns)
  package.json         # scripts below; main public/js/init.js
  backend/app.js       # one-line entry: require("qcobjects-cli/qcobjects-http2-server")
  src/
    index.html         # shell: <script type="module" src="js/init.js">
    404.html  robots.txt  humans.txt  manifest.json  sw.js  favicon.ico
    css/               # flat component files (button.css, card.css, modal.css),
                       # desktop/, mobile/, theme/{basic,cyan,redlight,xtra}
    img/               # icons/ (no screenshots/ dir in template)
    js/
      config.ts        # CONFIG settings binding
      init.ts          # boot sequence (Init component)
      customWidgets.ts # app widgets registration (RegisterWidget calls)
      packages/        # .ts Package() namespaces:
                       # com.qcobjects.services.*, com.qcobjects.installer,
                       # org.<app>.{components,controllers,effects,models,views}
                       # (third-party libs MAY be vendored under
                       # packages/thirdparty/libs/<lib>/ + SourceJS chain —
                       # observed in reference apps, absent from template)
    templates/
      components/      # hero/ lives HERE (*.tpl.html), not under css/
  public/              # build output only (parcel/esbuild distDir)
  spec/ + support/     # jasmine specs mirroring src/
```

- `src/js/init.ts` MUST boot exactly one root component; feature code MUST live
  under `src/js/packages/<org>.<app>.*` namespaces.
- `public/` MUST be generated artifacts only — never hand-edited.
- PWA files (`manifest.json`, `sw.js`, `robots.txt`, `404.html`) MUST exist in
  every production app; `sw.js` MUST NOT cache authenticated API responses.
- `index.html` SHOULD carry a `Content-Security-Policy` meta tag; ship WIDE-OPEN
  only for local/dev, and lock it down for production (reference: view-stack
  tutorial's permissive CSP with explicit lockdown note).

## Boot sequence (normative, the `init.js` CONFIG block)

```javascript
CONFIG.set("sourceType", "module");
CONFIG.set("relativeImportPath", "js/packages/");
CONFIG.set("componentsBasePath", "templates/components/");
CONFIG.set("delayForReady", 1);          // wait before first ready (incl. imports)
CONFIG.set("preserveComponentBodyTag", false);
CONFIG.set("useConfigService", false);   // true => load settings from config.json
CONFIG.set("routingWay","pathname");     // 'hash' | 'pathname' | 'search' (template default: pathname)
CONFIG.set("useLocalSDK",true);          // local SDK vs sdk.qcobjects.dev
CONFIG.set("tplextension","tpl.html");   // main => main.tpl.html
CONFIG.set("asynchronousImportsLoad",true);
CONFIG.set("serviceWorkerURI","/sw.js"); // auto-registered for offline
CONFIG.set("overrideComponentTag",true); // load-bearing for template resolution
Component.cached = true;                 // template-level caching default
```

## The 5 coding steps (normative tutorial contract)

1. **Main import file** (`js/packages/<org>.js`): license header + `Import(...)`
   lines + root `Package('<org>',[ Class('FormValidator',Object,{}) ])`.
2. **Services** (`<org>.services.js`): `Package('<org>.service',[ Class('FormSubmitService',JSONService,{ name, external:true, cached:false, method:'POST', withCredentials:false, url, _new_ (drop charset), done (super), fail }) ])`.
3. **Components** (`<org>.components.js`): `Class('MyCustomComponent',Component,{
   name, cached:false, controller:null, view:null, templateURI:ComponentURI({...}) })`.
4. **Controllers** (`<org>.controller.js`): `Class('MainController',Controller,{
   _new_ (logger.debug init) })`, feature controllers keep `component` ref from
   `_new_(o)` and mark `body.setAttribute('loaded',true)` in `done()`.
5. **HTML shell**: set CONFIG keys, then `Import('<org>')` (resolves
   `js/packages/<org>.js` via `relativeImportPath`).

Every generated source file MUST carry the license header (LGPLv3 text on the
v2.x line; MIT on v3.0+ — see [09-license](./09-license.md)).

## npm scripts contract (normative — as shipped in the template)

Every app `package.json` MUST provide: `test` (eslint+jasmine),
`lint`, `sync` (`git add . && git commit -am`), `preversion` (`npm i --upgrade`
+ test), `postversion` (`git push && git push --tags` — NOTE: full push, unlike
the CLI's branch-only rule in [08-ci-conventions](./08-ci-conventions.md)),
`coverage` (nyc),
`start` (`createcert` + `serve`), `serve`/`server` (`qcobjects-server`),
`start:dev` (watch build+serve), `collab` (`qcobjects-collab`),
`shell` (runs `qcobjects shell` — NOTE: no such CLI subcommand exists; the
working entrypoint is the `qcobjects-shell` BINARY),
`createcert`, `v-patch|v-minor|v-major`, `qcobjects` (local CLI),
`http-server` (local test), `gae-server` (App Engine), `build`
(= `publish:web`, the full chain below — NOT a bare TS step) / `build:ts`
(`npm test && npx tsc`), `prepare` (husky).
(No `publish:local` script exists — use `publish:web` / `publish:static`.)

## Environment & deploy (normative)

`.env` (never committed) SHOULD be accompanied by a committed `.env.example`
defining:
`ENGINE_NAME` (e.g. `sqlite3`), `DATABASE_NAME` (e.g. `admin.db`),
`DEFAULT_USER`, `DEFAULT_PASSWORD`, `MICROSOFT_API_KEY`, `GOOGLE_API_KEY`
(the `$ENV(…)` names above match the template `config.json`; the template
ships NO `.env*` file itself — add the example per app).
Netlify one-click deploy supported; live demo at `https://newapp.qcobjects.dev`;
Docker: `docker run -p 8080:8080 -p 8443:8443 qcobjects/qcobjects-newapp`
→ `https://127.0.0.1:8443/`.

## Production patterns (reference — observed in production apps, NOT in the pinned template)

The items below describe patterns proven in production reference apps
(`v2.4`-line commercial apps, jobs template). They are ADOPTABLE, not template
contract — the pinned `v2.4.40-ts` template contains NONE of: `config-debug.json`,
`config-prod.json`, `app.yaml`, `qcobjects.service`, `src/templates/email/`,
seed CSV/JSON pairs (only `Dockerfile`, `docker-compose.yml`, `src/_redirects` overlap).

- **Multi-env configs:** apps MAY ship `config.json` + `config-debug.json` +
  `config-prod.json` variants (same shape, different `$ENV` bindings/ports);
  the deploy step selects which file becomes the effective `config.json`
  (copy/symlink at deploy time — there is no framework `--config` flag).
  Secrets MUST differ per environment; never reuse prod credentials in debug.
- **Data seeding:** list/data-driven apps MAY ship seed pairs — a source CSV
  plus its converted `data/*.json` (reference: jobs template's
  `datamercadopublico.csv` + `data/mercadopublico.json`). The JSON is what the
  app loads; the CSV is the editable source of record. Regeneration MUST be
  scripted (`csv→json` step documented in README), never hand-edited JSON
  drifting from its CSV.
- **Web publish chain:** production `publish:web` runs staged —
  `build:static` (copy `src/` → `build/`) → `build:ts` (test + `tsc`) →
  `publish:static` (`build/` → `public/`, excluding `js`, with `minify:css`
  nested inside) → `publish:esbuild` (bundle to `public/js`) →
  `generate-sw` (terminal stage). `prestart` SHOULD run the publish
  chain so servers never boot stale artifacts.
- **Deploy targets:** beyond Netlify/Docker — `app.yaml` (App Engine,
  `gae-server`), `qcobjects.service` (systemd unit), `Dockerfile` +
  `docker-compose.yml` (container; base `qcobjects/qcobjects`), `_redirects`
  (host redirect rules), cloud aliases (`azure-server`, `aws-server`,
  `do-server` all delegate to `npm start`). Multi-target apps MUST keep one
  canonical `publish:web` that every target invokes.
- **Email templates:** transactional mail lives in `src/templates/email/*.tpl.html`
  (one template per audience, e.g. user + backoffice notifications), rendered
  server-side through the newsletter/contactform handlers with subjects from
  `$ENV(...)` settings — never hardcode recipients, subjects, or keys.
- **Backend entry:** production backends expose a one-line `backend/app.js`
  (`require("qcobjects-cli/qcobjects-http2-server")`); all behavior stays in
  `config.json` routes, never in the entry file.
- **Quality gates:** `lighthouse` script with budgets SHOULD run against the
  local TLS server before release; `spec/` + `coverage/` MUST stay green.
- **OAuth redirect target:** OAuth-style flows land on a minimal static page
  (`auth_redirect.html`: shell + CSP + `<noscript>`, no app boot) that captures
  the provider response (reference: academy app). Keep it dependency-free —
  it MUST NOT import the framework (loads before auth exists).

## Electron desktop shell (reference: `qcobjects-electron` line)

Desktop apps wrap the same web tree in an Electron shell — three files at the
app root plus packaging metadata:

- **`main.js` (required):** creates `BrowserWindow` (800×600 baseline),
  `webPreferences: {nodeIntegration: true, preload: <preload.js>}`,
  `loadFile('index.html')`, macOS `window-all-closed`/`activate` lifecycle.
  `nodeIntegration:true` is REQUIRED — it enables QCObjects features in the
  window (notably `file:` template loading through the `fetch` path, see
  [03-core-framework](./03-core-framework.md) § Loading transport).
  `require('qcobjects')` in the main process.
- **`preload.js` (required):** `require('qcobjects')` in the preload
  (Chrome-extension-equivalent sandbox); debug logger enablement.
- **`renderer.js`:** stock Electron renderer stub (no Node APIs; bridge via preload).
- **`package.json`:** `"main": "main.js"`, `"start": "electron ."`, `electron`
  dependency (reference pins v8 line — use a maintained Electron on new apps).
  Publish via the template `publish:electron` npm script (the CLI has no native
  electron target).
- The SAME `src/` tree (components, templates, PWA assets) ships inside the
  shell — no app-code fork between web and desktop; only the shell trio +
  packaging differ.

## Hybrid packaging — PhoneGap/Cordova (reference: `qcobjects-phonegap-app`)

Hybrid apps ship the same web tree inside a Cordova shell:

- **`config.xml` (required):** widget descriptor — `id` (reverse-DNS app id),
  `version`, `<content src="index.html"/>`, per-`platform` icons and splash
  screens (android densities + iOS sizes), preferences
  (e.g. `DisallowOverscroll`, `android-minSdkVersion`).
- **`www/` tree:** mirrors the web `src/` tree (css, js, templates, assets) as
  the device web root; `platforms/` (per-OS Cordova build code) and `plugins/`
  (vendored `cordova-plugin-*` with `plugin.xml`) are generated/vendored
  alongside — never hand-edit generated platform code.
- **Boot:** wait for the `deviceready` event before QCObjects init (Cordova APIs
  don't exist before it). The framework detects the shell via
  `is_phonegap = typeof cordova !== "undefined"` (`src/platform.ts`) and adapts
  transport accordingly (no `Content-Type` header on XHR — see
  [03-core-framework](./03-core-framework.md) § Loading transport).
- `res/` icons + `.pgbomit` mark PhoneGap-Build assets, as in the base layout.

## Verification

- `npm run build` from clean checkout reproduces `public/` byte-equivalent config.
- `npx eslint "src/**/*.ts"` passes; jasmine `spec/` suite passes.
- Fresh stamp serves the boot CONFIG above with zero edits.
