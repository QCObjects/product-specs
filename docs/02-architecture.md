# 02 — Architecture

## Purpose

Pin the layered architecture shared by every QCObjects repo and app, including
the N-Tier/micro-service doctrine and the backend routing contract.

Sources: `QCObjects` README §§ N-Tier, Micro-services, backend settings/routing,
microservice class (`v2.5.142`); `qcobjects-new-app` `config.json` (`v2.4.40-ts`).
Definitions below are authoritative.

## Scope

Layers, their responsibilities, and the contracts between them.
Component-level detail lives in 03–05.

## The five layers (normative, read bottom-up)

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

## Rendering model: CSR + opt-in SSR (normative)

QCObjects renders on the client by default AND supports server-side rendering
through the CLI server — same component pipeline both sides. Sources:
`qcobjects-cli` `src/main-file.ts` (`FileDispatcher`), `src/defaultsettings.ts`.

- **CSR (default):** components build in the live browser DOM (template XHR,
  `{{}}` binding, routing against `location`, shadow roots). `Component.ts`
  carries `isBrowser` guards for DOM-only behaviors (effects, fullscreen,
  shadow attachment).
- **SSR (opt-in):** `FileDispatcher` serves `.html`/`.tpl.html` files by
  instantiating a real `Component` server-side in Node —
  `New(Component, {name:"static_source", template: source, tplsource:"inline",
  data:{…}, done({component}){ body = component.parsedAssignmentText }})` —
  and sending `parsedAssignmentText` as the response body. Template binding,
  `$…()` processors, and the template handler all run server-side exactly as
  in the browser; no browser DOM is needed for this inline path.
- **Gate:** SSR applies only when `CONFIG.get("useTemplate")` is true AND the
  extension is `.html`/`.tpl.html` (CLI default: `useTemplate=false`).
  Otherwise the file streams unrendered with its mime type + `cacheControl`.
  Apps that want SSR MUST set `useTemplate:true` in `config.json`.
- **Scope note:** SSR covers template+data rendering (the `parseTemplate` /
  handler path). Browser-only behaviors (shadow DOM attachment, effects,
  client routing, XHR-loaded external templates) still execute client-side on
  hydration. `publish:static` remains a file copier, not a prerenderer.
- Apps SHOULD still ship a crawlable static shell (meta/OG tags, `404.html`,
  sitemap) for crawlers that don't execute JS when SSR is off.

## N-Tier doctrine (from the core README)

QCObjects targets professional Multitier/N-Tier environments for scalability
and reliability. Reference background (study only):
Multitier Architecture (Wikipedia), 3-Tier Architecture (tonymarston.net),
Multi-Tier Application (techopedia), N-Tier concepts (guru99).

- Presentation (components/views), logic (controllers/services), data
  (services/microservices) MUST stay separable; a component MUST be usable with
  a different service, and a service with a different component.
- Routing (`hash` | `pathname` | `search`, see spec 03) belongs to presentation;
  persistence and auth belong to services/microservices.

## Micro-service doctrine

A microservice compacts a backend fragment callable remotely, so a high-level
service splits into small completable tasks (reference: microservices.io,
Wikipedia Microservices).

- **Definition rule:** inside a microservice package, a `Microservice` class
  extending `BackendMicroservice` is REQUIRED. The HTTP/2 server calls `post()`
  (or the verb method) only on POST requests to the configured path, answering
  e.g. JSON-RPC 2.0 envelopes.
- **Canonical example** (signup saver, from core README `v2.5.142`):

```javascript
'use strict';
const fs = require('fs');

Package('cl.quickcorp.backend.signup',[
  Class('Microservice',BackendMicroservice,{
    body:{ "jsonrpc": "2.0", "result": "", "id": 1 },
    saveToFile: function (filename,data){
      logger.debug('Writing file: '+filename);
      fs.writeFile(filename, data, (err) => {
        if (err) throw err;
        console.log('The file has been saved!');
      });
    },
    post:function (data){
      let submittedDataPath = CONFIG.get('dataPath'); // filled out from qcobjects-server
      let filename = submittedDataPath+'signup/signup'+Date.now().toString()+'.json';
      this.saveToFile(filename,data);
    }
  })
]);
```

(Note: `Date.now().toString()` in source; shape above.)

## `BackendMicroservice` base API (normative)

Source: `src/BackendMicroservice.ts`, pinned at
`https://github.com/QCObjects/QCObjects/blob/v2.5.142/src/BackendMicroservice.ts`.

- **Construction:** `New(MicroserviceClass, {domain, basePath, body, stream, request})`;
  the constructor stores all five, defaults `body` to `null`, runs `cors()`,
  and wires dispatch. `stream`/`request`/`route`/`headers` stay available as
  instance fields for the whole call.
- **Verb dispatch:** stream `"data"` events route to `post(data)`; all other
  request methods dispatch to same-named methods — `get`, `head`, `put`,
  `delete`, `connect`, `options`, `trace`, `patch`. Override exactly the verbs
  the route serves; default verb methods log and call `done()`.
- **Answering:** set `this.body` (object, e.g. a JSON-RPC 2.0 envelope
  `{jsonrpc:"2.0", result, id}`) then call `this.done()`. Never write the raw
  stream unless implementing a custom transport.
- **`cors()` semantics** (driven by `route.cors`):
  `allow_origins` (`"*"` or list; mismatch empties the body and finishes —
  fail-closed); `allow_credentials` (default `"true"`);
  `allow_methods` (default `GET, OPTIONS, POST`); `allow_headers` (default `*`).
  With no `route.cors` at all, validation is skipped (log only) — routes that
  need browsers MUST declare `cors`.
- The `com.qcobjects.backend.microservice.static` built-in serves
  `redirect_to` file targets — use it for static routes instead of custom code.

## Secret-hiding proxy pattern (normative, reference: `qcobjects-openai-api`)

Third-party APIs with secret keys MUST be integrated browser → same-origin
proxy → vendor, never browser → vendor. The OpenAI/Azure packages prove the shape:

- **Browser side:** a `Service` subclass (`ProxyOpenAIService`) POSTs to a
  same-origin route (`/api/openai`, `external:false`, `cached:false`), carrying
  only the model payload (`model`, `messages`, `temperature`) — NO key, NO
  `Authorization` header.
- **Server side:** a `BackendMicroservice.post()` builds the vendor client
  service (key from `CONFIG.get("OPENAI_API_KEY")`, i.e. `$ENV(...)` resolved
  only in Node), executes it via `serviceLoaderNode`, sets `this.body` to the
  vendor response, and `done()`s. Errors become `this.body` + `done()` (fail
  closed with a body, not a stream write).
- **Rules:** vendor keys MUST live only in server-side `$ENV` settings;
  browser code MUST NOT contain, import, or receive keys; proxy routes SHOULD
  keep `cached:false`; streaming/SSE responses are NOT covered by this pattern
  (request/response JSON only — verify before promising streams).

## Service composition — BFF aggregation (normative, reference: store app)

Beyond proxying, a microservice verb method MAY run one or more client `Service`
subclasses through `serviceLoader` and reshape their responses into `body`
(backend-for-frontend aggregation). Production proof (Printful catalog):

- `Microservice.get()` instantiates `PrintfulService` (a `Service` subclass
  whose `_new_` sets `Authorization: Basic <process.env KEY>` — `process.env`
  exists only in Node, so the class is backend-only by construction),
  `serviceLoader(New(PrintfulService,{data:null}))`, then
  `microservice.body = JSON.parse(service.template)` + `done()`.
- **Rules:** composing services MUST be `Service` subclasses (never raw
  `https` calls — keeps headers, `done`/`fail`, and both transport legs);
  secrets MUST come from `process.env`/`$ENV` (never literals, never client
  reachable); each upstream failure MUST map to a `body` + `done()` (or a
  deliberate non-200), never an unhandled rejection; aggregation of N
  upstreams SHOULD `Promise.all` them and merge, not chain sequentially.

## Front-end vs back-end services (normative)

Two different classes, two runtimes, one contract shape:

| | Front-end (`Service`/`JSONService`) | Back-end (`BackendMicroservice`) |
|---|---|---|
| Where it runs | Browser, via XHR `serviceLoader` (or Node via `serviceLoaderNode`) | CLI server, dispatched from `backend.routes` |
| Definition | `Class('X',Service\|JSONService,{name,url,method,…})` | `Class('Microservice',BackendMicroservice,{get/post/…})` in a route package |
| Trigger | Component/controller calls `serviceLoader(New(X))` | HTTP verb on the route path |
| Input | `service.data` (bound params) | Request stream data / route params |
| Output | `service.template` + `JSONresponse`, `done`/`fail` | `this.body` + `done()` |
| Secrets | MUST NOT hold keys (proxy instead) | MAY hold keys via `$ENV`/`process.env` |
| Response as UI | Binds `{{}}` into templates directly | Never touches DOM — returns data/envelopes |

- The two sides meet ONLY at HTTP route boundaries (proxy + BFF patterns
  above) sharing the `{request, service|component}` standard-response shape —
  see [03-core-framework](./03-core-framework.md) §§ Services, Loading transport.
- A `Service` subclass executed via `serviceLoaderNode` inside a microservice
  verb method is still a FRONT-end class reused server-side (Printful proof) —
  classify by definition site, not execution site.
- Rules: front-end services MUST NOT embed secrets or absolute vendor URLs
  (same-origin proxy paths only); back-end services MUST NOT import browser
  globals (`document`, `window`, `location`); shared DTO shapes SHOULD be
  documented once (in the route's spec entry) and referenced from both sides.

## Backend routing contract (`config.json`)

- Every route REQUIRES `path` + `microservice` (package string as indexing point).
  Optional: `name`, `description`, `redirect_to`, `responseHeaders`, `cors`.
- `path` is matched as a regex string (e.g. `"^/demo-tests/QCObjects-SDK.js$"`).
- Unmatched paths fall back to static-file serving from `documentRoot` if the
  file exists — so the server handles static AND dynamic from one table.
- Server-side `config.json` MAY carry: `documentRoot`, `basePath`, `projectPath`,
  `domain`, `dataPath`, TLS material (`private-key-pem`, `private-cert-pem`),
  ports. Full field catalogue: [12-schemas](./12-schemas.md).

```json
{
  "documentRoot": "/home/qcobjects/projects/mynewapp/",
  "relativeImportPath": "js/packages/",
  "basePath": "/home/qcobjects/projects/mynewapp/",
  "projectPath": "/home/qcobjects/projects/mynewapp/",
  "domain": "mynewapp.qcobjects.com",
  "dataPath": "/etc/qcobjects/data/",
  "private-cert-pem": "/etc/letsencrypt/live/mynewapp.qcobjects.com/fullchain.pem",
  "private-key-pem": "/etc/letsencrypt/live/mynewapp.qcobjects.com/privkey.pem",
  "backend": { "routes": [
    { "path": "/createaccount", "microservice": "org.quickcorp.backend.signup", "responseHeaders": {} }
  ]}
}
```

## Certificates

- `qcobjects-createcert` generates self-signed local TLS material (dev only).
- Production MUST use externally provisioned certificates (e.g. LetsEncrypt +
  Certbot paths wired via `private-key-pem`/`private-cert-pem`).

## Verification

- `npm ls` in any app shows exactly one `qcobjects` + one `qcobjects-sdk` copy.
- Deleting a handler package degrades only its routes; the server still boots.
- `schemas/examples/backend-config.json` validates against `schemas/config.schema.json`.
