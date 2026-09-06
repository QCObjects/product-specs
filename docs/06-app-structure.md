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
  config.json          # runtime truth: domain, ports, routes, paths ($ENV/$config)
  config.yaml          # MAY exist; if both JSON+YAML present, YAML wins (dismiss JSON)
  package.json         # scripts below; main public/js/init.js
  backend/             # microservice packages live here
  src/
    index.html         # shell: <script type="module" src="js/init.js">
    404.html  robots.txt  humans.txt  manifest.json  sw.js  favicon.ico
    css/               # components/hero, desktop/, mobile/, theme/{basic,cyan,redlight,xtra}
    img/               # icons/, screenshots/
    js/
      config.ts        # CONFIG settings binding
      init.ts          # boot sequence (Init component)
      customWidgets.ts # app widgets registration (RegisterWidget calls)
      packages/        # Package() namespaces: org.myapp.*
    templates/
      components/      # hero/, pages/
  public/              # build output only (parcel/esbuild distDir)
  spec/ + support/     # jasmine specs mirroring src/
```

- `src/js/init.ts` MUST boot exactly one root component; feature code MUST live
  under `src/js/packages/<org>.<app>.*` namespaces.
- `public/` MUST be generated artifacts only — never hand-edited.
- PWA files (`manifest.json`, `sw.js`, `robots.txt`, `404.html`) MUST exist in
  every production app; `sw.js` MUST NOT cache authenticated API responses.

## Boot sequence (normative, the `init.js` CONFIG block)

```javascript
CONFIG.set("sourceType", "module");
CONFIG.set("relativeImportPath", "js/packages/");
CONFIG.set("componentsBasePath", "templates/components/");
CONFIG.set("delayForReady", 1);          // wait before first ready (incl. imports)
CONFIG.set("preserveComponentBodyTag", false);
CONFIG.set("useConfigService", false);   // true => load settings from config.json
CONFIG.set("routingWay","hash");         // 'hash' | 'pathname' | 'search'
CONFIG.set("useLocalSDK",true);          // local SDK vs sdk.qcobjects.dev
CONFIG.set("tplextension","tpl.html");   // main => main.tpl.html
CONFIG.set("asynchronousImportsLoad",true);
CONFIG.set("serviceWorkerURI","/sw.js"); // auto-registered for offline
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

## npm scripts contract (normative)

Every app `package.json` MUST provide: `test` (eslint+jasmine),
`lint`, `sync` (`git add . && git commit -am`), `preversion` (`npm i --upgrade`
+ test), `postversion` (`git push && git push --tags`), `coverage` (nyc),
`start` (`createcert` + `serve`), `serve`/`server` (`qcobjects-server`),
`start:dev` (watch build+serve), `collab` (`qcobjects-collab`), `shell`,
`createcert`, `v-patch|v-minor|v-major`, `qcobjects` (local CLI),
`http-server` (local test), `gae-server` (App Engine), `build`/`build:ts`
(TypeScript), `publish:local`, `prepare` (husky).

## Environment & deploy (normative)

`.env` (never committed; `.env.example` committed) MUST define:
`ENGINE_NAME` (e.g. `sqlite3`), `DATABASE_NAME` (e.g. `admin.db`),
`DEFAULT_USER`, `DEFAULT_PASSWORD`, `MICROSOFT_API_KEY`, `GOOGLE_API_KEY`.
Netlify one-click deploy supported; live demo at `https://newapp.qcobjects.dev`;
Docker: `docker run -p 8080:8080 -p 8443:8443 qcobjects/qcobjects-newapp`
→ `https://127.0.0.1:8443/`.

## Verification

- `npm run build` from clean checkout reproduces `public/` byte-equivalent config.
- `npx eslint "src/**/*.ts"` passes; jasmine `spec/` suite passes.
- Fresh stamp serves the boot CONFIG above with zero edits.
