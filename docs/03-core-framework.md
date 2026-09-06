# 03 — Core Framework (`qcobjects`)

## Purpose

Specify the core package: what it exports, how classes/packages work, and the
complete essentials reference — so the core README can be summarized without loss.

Source: `QCObjects` README §§ Essentials, List/Math, Reference (`v2.5.142`).
Code pins: `https://github.com/QCObjects/QCObjects/blob/v2.5.142/src/<File>.ts`.
Definitions below are authoritative.

## Scope

Repo `QCObjects/QCObjects`, npm `qcobjects` (`v2.5.142`).
Build/test detail: [14-build-scripts-blueprint](./14-build-scripts-blueprint.md).

## Distribution contract (normative)

- Entry points MUST be: `main → public/cjs/index.cjs`,
  `module → public/esm/index.mjs`, `browser → public/browser/QCObjects.js`,
  `types → public/types/index.d.ts`, with `exports` maps for `.`, `./package.json`,
  `./tsconfig*`, `./*.js|cjs|mjs`, and wildcard `./*`.
- Core MUST NOT embed: HTTP servers, CLI parsing, PWA shell, or widget CSS.
- Every public symbol MUST ship type declarations under `public/types/`.
- Specs in `spec/` (jasmine: `testsSpec`, `testsConfigSpec`,
  `testsClassFactorySpec`, `testsGlobalFeaturesSpec`, `testsTypeSpec`);
  `npm test` MUST run lint + full suite green.

## Class system (normative)

- `Class(name, definition)` / `Class(name, Parent, definition)` declares;
  `InheritClass` is the common base; `_new_` is the constructor hook.
- `New(ClassRef, props)` instantiates; getters in `props` execute once.
- `_super_(SuperName, method).call(this, params)` reaches the parent implementation.
- `ClassFactory(name)` returns the factory from the class queue or a package;
  last same-name declaration wins the bare reference — use fully-qualified
  `ClassFactory('org.pkg.Name')` when extending across packages to protect scope.
- `Package(name, [classes])` defines; `Package(name)` retrieves (promise-based,
  scope-oriented loading).
- `Import('dotted.package'[, ready][, external])` loads `<package>.js` from
  `relativeImportPath` (or `remoteImportsPath` when external); `.js` extension
  is mandatory and unchangeable (security).
- `Export(symbol)` lifts a local to top-level scope.
- `[el].Cast(TargetClass)` merges another type's properties (e.g. a `div` cast
  to a QCObjects class gains a `body`).
- `Tag(selector)` returns a mappable/sortable/filterable element list.
- `Ready(fn)` runs after QCObjects init + `window.onload`; dynamic `<component>`
  loads do NOT trigger Ready — use controller `done()` there.
- `GLOBAL.set/get` reaches the global scope store.
- `waitUntil(effect, condition)` runs once when true (use sparingly).

**Canonical class example:**

```javascript
Class('MyClassName',InheritClass,{
  propertyName1:0,
  propertyName2:'',
  classMethod1: function (){ return this.propertyName1; }
});
var o = New(MyClassName,{ propertyName1:1, propertyName2:"some value" });
```

## CONFIG & processors (normative)

- `CONFIG.set(key, value)` / `CONFIG.get(key)`; `useConfigService=true` loads
  `config.json` from the app basePath via `ConfigService`
  (`ConfigService.configFileName='config.json'` by default).
- Encrypted `config.json`: encrypt at the config tool (domain + content), paste
  ciphertext back; decoding is transparent.
- **Processors** (`Processor.setProcessor(fn)`, non-arrow functions so `this`
  is the handler): `$ENV(VAR)` (Node/CLI/Collab only), `$config(key)` (all envs),
  plus custom `$NAME(args)`:

```json
{ "domain": "localhost", "env1": "$ENV(ENV1)",
  "customSettings": { "value1": "$config(domain)" } }
```

```javascript
let SERVICE_HOST = function (arg){
  var h = this;
  return (new URL(h.processors.ENV(arg))).host;
};
Processor.setProcessor(SERVICE_HOST); // enables "$SERVICE_HOST(SERVICE_URL)"
```

## Component model (normative)

**Class properties:** `domain`, `basePath` (auto); `templateURI` (use
`ComponentURI({COMPONENTS_BASE_PATH, COMPONENT_NAME, TPLEXTENSION, TPL_SOURCE})`);
`tplsource` (`default`|`none`); `url`, `name`, `method` (default `GET`); `data`
(`{{prop}}` binding; needs `rebuild()` to refresh); `reload` (replace vs append);
`cached` (load template once; static default or per-instance); `routingWay`
(`hash`|`pathname`|`search`, set globally via CONFIG), `validRoutingWays`,
`routingNodes`, `routings`, `routingPath`, `routingSelected`; `subcomponents`;
`body` (setting it triggers the routings builder).

**Methods:** `set/get`, `rebuild()` (via componentLoader), `Cast()`, `route()`,
`fullscreen()/closefullscreen()`, `css(obj)`, `append(child)`, `attachIn(selector)`.

**`<component>` tag attributes:** `name`; `cached="true"` (only `"true"` counts);
`data-*` one-way mock bindings (NOT bidirectional); `controllerClass`;
`viewClass`; `componentClass`; `effectClass`; `template-source` (`none`|`default`);
`tplextension` (default `html`).

```html
<component name="main"></component>  <!-- loads ./templates/main[.tplextension] -->
```

**Loaders:** `componentLoader(instance, load_async)` → Promise
(`successStandardResponse{request, component}` / `failStandardResponse{component}`);
`[element].buildComponents()` rebuilds the subtree (usually automatic).

**MVC:** `Controller` (base; `done()` fires per component load — the hook for
dynamic components), `View`, `VO` (value object), `DDO` (dynamic data object).

## Services (normative)

**`Service` props:** `domain`, `basePath` (auto); `url` (absolute or basePath-
relative; `external:true` + serviceLoader for off-origin); `name` (descriptive,
non-unique); `method` (`GET`/`POST`/`PUT`/…); `data` (`{{prop}}` response binding);
`cached` (false = always reload). Methods: `set/get`.

**`serviceLoader(instance)`** → Promise; typical definition:

```javascript
Class('MyTestService',Service,{
  name:'myservice', external:true, cached:false, method:'GET',
  headers:{'Content-Type':'application/json'},
  url:'https://api.github.com/orgs/QuickCorp/repos', withCredentials:false,
  done:()=>{ /* service loaded */ }
});
```

**`JSONService`** extends with `JSONresponse`; override `done` via
`_super_('JSONService','done').call(this,result)`; drop `headers.charset` when
the endpoint dislikes it. **`ConfigService`** loads `config.json`.
**`SourceJS`/`SourceCSS`** inject non-package dependencies from controllers
(`controller.dependencies.push(New(SourceJS,{external, url, done}))`).

## Component authoring rules (normative)

Harvested from the field-verified scaffolding recipe (2.4 line; key APIs
re-confirmed in `v2.5.142` source: `hostElements`/`subtags`, shadow-root
handling, tag filter `quick-component:not([loaded]),component:not([loaded])`
in `src/tag_filter.ts`).

- **Widget vs generic:** a smart widget (`<greeting-component>`) is a shell —
  its constructor creates a child generic node (`<quick-component name="…">`)
  and copies attributes onto it; the generic node is the real renderer.
  Prefer plain `<quick-component name>` with an explicit `componentClass`
  string over widget tags nested under shadowed layouts (the widget transform
  can drop `componentClass` there, misnaming the component and 404ing its template).
- **Class resolution:** the class comes from the element's `componentClass`
  attribute (default: base `Component`); `name` drives only `templateURI` and
  registration. Inline `template`/`data` patterns REQUIRE the real class via
  `componentClass` or those fields are ignored. One namespace per component
  (`Package("com.x.card",[Card])`, referenced as `…card.Card`).
- **`data-*` merging:** the base constructor merges element `data-*` attributes
  into `data` — do NOT also declare a `data` field on a class that wants those
  values (the field initializes after `super()` and clobbers them).
- **Interaction:** implement widget behavior in `done()` (fires after build with
  a live `shadowRoot`); query via `this.hostElements(selector)` (shadow-aware:
  `shadowRoot` when shadowed, else `body`; `subtags` is the same getter).
  Page-level delegated listeners see retargeted `event.target` (the host) —
  use `event.composedPath()[0]` + `getRootNode()` there instead.
- **Shadow CSS:** page CSS cannot reach shadow roots — each template carries
  `<style>@import url("css/components/….css")</style>` and the imported file
  chains further imports (e.g. compiled Tailwind); the browser resolves the
  chain inside the shadow root.
- **Blank component triage:** 404 on the `.tpl.html` XHR (template must exist
  under the served root) is the #1 cause; enable `logger.debugEnabled` and look
  for `template source … is default|inline`, `type for … is Component`
  (base-class fallback), `LOADING COMPONENT DATA`, and `Something wrong loading
  the component`.

## Effects, Timer, codecs (normative)

- Custom effects extend `Effect` and override `apply`, delegating via
  `_super_('Fade','apply').apply(this,arguments)`; engine runs on
  `requestAnimationFrame` and mutates CSS smartly.
- `Timer.thread({duration, timing(fraction,elapsed), intervalInterceptor(progress)})`
  emulates threads (modern browsers only).
- `_Crypt`: `New(_Crypt,{string,key})._encrypt()/._decrypt()`, or static
  `_Crypt.encrypt(text,key)` / `_Crypt.decrypt(cipher,key)`.
- `ComplexStorageCache({index, load, alternate})` + `getCached(id)` for
  localStorage object caching.
- `asyncLoad(fn, args)` runs once after the async queue, before Ready.
- `ArrayList` (`New(ArrayList,[...])`), `ArrayCollection` (`{source}`),
  `.unique()`, `.table()` (shell only), `.sort()`, `.sortBy(prop)`,
  `.matrix(n[,v])`, `.matrix2d`, `.matrix3d`, `range(n|a,b)`,
  `.sum()`, `.avg()`, `.min()`, `.max()`.

## Verification

- `node -e "require('qcobjects')"` and `import 'qcobjects'` both resolve.
- `npx tsc --noEmit -p tsconfig.d.json` passes; jasmine suite passes.
- Every symbol above resolves at the pinned tag; drift opens a spec-update PR.
