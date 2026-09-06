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

## TypeScript posture (normative)

- **Runtime requires no transpiler:** apps MAY be pure `.js` — the browser bundle
  runs as-is; `Class()`/`Package()`/`Import()` work in plain JavaScript with zero
  build step (see [06-app-structure](./06-app-structure.md) boot sequence).
- **Transpilers allowed:** apps MAY be authored in TypeScript — templates ship
  `src/js/*.ts` + `*.d.ts` alongside compiled output and a binding `build:ts`
  script; `tsc` declaration builds are part of every repo's pipeline
  (see [14-build-scripts-blueprint](./14-build-scripts-blueprint.md)).
- **Framework authored in TypeScript:** core `src/` is 78 `.ts` files / 0 `.js`
  (SDK: 26 `.ts` / 0 `.js`) at `v2.5.142`/`v2.5.105`; every repo carries
  `tsconfig.json` + `tsconfig.d.json` + `tsconfig.jasmine.json` and ships
  first-party declarations under `public/types/` (the `types` + `exports`
  contract above). Type coverage MUST NOT regress: new public API without
  declarations fails the release.
- **Deno:** the CLI is Deno-compatible (`deno.json` + `mod.ts`, strict
  compiler options) — types flow to Deno consumers via the same declarations.

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

## Component inheritance (normative)

Sources: `src/Class.ts`, `src/InheritClass.ts`, `src/super.ts`, `src/is_a.ts`,
pinned at `https://github.com/QCObjects/QCObjects/blob/v2.5.142/src/Class.ts`.

- **Two equivalent modes.** Factory: `Class('Child', Parent, definition)` builds
  `class extends Parent` with the parent's `__definition` merged in
  (`LegacyCopy`, `__instanceID` stripped so IDs stay unique). Native:
  `class Child extends Parent {…}`. Both produce a real prototype chain; both
  register by name and resolve via `ClassFactory`. Mixed hierarchies (factory
  parent + native child and vice versa) MUST work.
- **Construction:** `InheritClass`'s constructor copies `__definition`, binds
  function props from the init object to the instance, and assigns a read-only
  `__instanceID`. Factory classes run the `_new_` hook; native classes use
  `constructor(o)` + `super(o)` — pass the init object up in both modes or
  inherited fields stay unset.
- **`_super_` is registry-based, not chain-based:** `_super_('Parent','m')`
  returns `ClassFactory('Parent')['m']`. It therefore works across packages but
  REQUIRES the parent to be registered under exactly that name — renaming or
  late-loading the parent breaks the call. Prefer native `super.m()` inside
  native classes; reserve `_super_` for factory definitions and cross-package
  reaches.
- **Type checks:** `is_a(obj, typeName)` checks `hierarchy()` membership, then
  `__getType__`/`ObjectName`, then `typeof`. Use it instead of `instanceof`
  across package boundaries (duplicate module copies break `instanceof`).
- **Rules:** names MUST NOT be forbidden words (`Class()` throws); every class
  SHOULD extend `InheritClass` (directly or transitively) so `__instanceID`,
  `__classType`, and `hierarchy()` exist; overrides MUST call the parent
  implementation unless intentionally replacing it.

**Canonical class example:**

```javascript
Class('MyClassName',InheritClass,{
  propertyName1:0,
  propertyName2:'',
  classMethod1: function (){ return this.propertyName1; }
});
var o = New(MyClassName,{ propertyName1:1, propertyName2:"some value" });
```

## Native `class` / `new` interop (normative)

Recent framework versions accept native ES class syntax everywhere the
factory syntax works — detection via `__is_raw_class__` (public API:
a function whose source starts with `class`), pinned at
`https://github.com/QCObjects/QCObjects/blob/v2.5.142/src/is_raw_class.ts`.

- **Declare natively:** `class Main extends InheritClass {}` is a first-class
  class definition; `New(Main, {})` instantiates it (covered by `testsSpec`:
  `__instanceID` is a number, `__classType` is `"Main"`).
- **Package natively:** `Package('org.pkg',[class Card extends Component {...}])`
  registers each class with namespace stamping; a single class may also be
  passed directly — `Package('org.pkg', MyClass)` sets
  `__definition.__namespace` + `__namespace` and registers it.
- **Resolve natively:** `ClassFactory('org.pkg.Name')` returns the native class
  from the package (last registered wins the bare reference, same rule as above).
- **Instantiate natively:** `New()` is defined as `new __class__(args)`, so the
  native `new` operator works too — `new FormController(o)`, `new Move()`,
  `new i18n_messages_es()` (all used across the SDK sources, which are themselves
  written in native class syntax).
- **Introspection:** `__getType__` names raw classes via `constructor.name`;
  `LegacyCopy` copies them branch-aware. Native and factory classes MAY be mixed
  freely in one package.
- New code SHOULD prefer native `class`/`extends` syntax; the `Class()` factory
  remains supported for cross-browser legacy paths and dynamic definitions.

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

## Template meta processors `$…(…)` (normative)

Separate from CONFIG processors: `$name(args)` placeholders inside component
templates (and any string in processed config objects, via `processObject`
recursion) are expanded by `Processor.process(template, component)` — matched by
`\$name((.*))` and invoked as `fn(componentInstance, ...args.split(","))`.
Sources: `src/Processor.ts`, `src/defaultProcessors.ts` (`setDefaultProcessors`),
pinned at `https://github.com/QCObjects/QCObjects/blob/v2.5.142/src/Processor.ts`.

Default meta processors (always registered):

- `$mapper(componentName,valueName)` — renders a list value (component `data`/
  prop, else global) as `<quick-component name="…" data-k="v" …>` items, one per
  element with its keys as `data-*` attributes.
- `$layout(portrait|landscape, cssfile)` — emits orientation/aspect-ratio
  `@import` rules for the CSS file (mobile-first portrait set + landscape set).
- `$component(name=…, componentClass=…, …)` — emits a
  `<component name="…" componentClass="…" …>` tag declaration.
- `$quick_component(name=…, componentClass=…, …)` — same for `<quick-component>`.
- `$repeat(length, text)` — repeats `text` `length` times, substituting
  `{{index}}` per occurrence (built on `range(length)`).
- `$ENV(VAR)` / `$config(key)` resolve in the same pass where applicable
  (Node/CLI/Collab for `$ENV`; everywhere for `$config`).

Rules: custom meta processors register via `Processor.setProcessor(fn)` with
non-arrow functions (`this` is the handler); names MUST be alphanumeric;
processors MUST be pure string transforms (no DOM writes — return markup);
templates SHOULD prefer `$component`/`$mapper` over hand-concatenated tags.

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

## Smart widgets (normative)

Smart widgets let a component be declared as a native custom element instead of
a `<component>` tag. Source: `src/WidgetsFactory.ts`, pinned at
`https://github.com/QCObjects/QCObjects/blob/v2.5.142/src/WidgetsFactory.ts`.

- `RegisterWidget(name)` / `RegisterWidgets(...names)` define real custom
  elements via `customElements.define(name, class extends _ComponentWidget_)`.
  Widget names MUST contain a hyphen (custom-elements requirement).
- Register widgets in app code (`customWidgets.ts`), e.g.
  `RegisterWidget("signup-form")`, then declare
  `<signup-form componentClass="..." controllerClass="...">` directly in HTML.
- On upgrade, the widget's light-DOM children are cloned into the component
  body and `data-*` attributes are forwarded onto the body as `data-*` —
  so slots (`<h1 slot="title">`) and bindings flow through untouched.
- All tag attributes (`name`, `cached`, `controllerClass`, `componentClass`,
  `effectClass`, `template-source`, `tplextension`, `data-*`) work identically
  on widget tags and `<component>` tags.
- Browser-only: `RegisterWidget` throws
  `"RegisterWidget is not implemented for non browser ecosystems yet."` outside browsers.
- New components SHOULD ship a widget name (hyphenated component name) alongside
  the `<component>` form; templates SHOULD demonstrate the widget form.

## Nested components routing (normative)

Every component owns its routing table, and subcomponents own theirs —
routing is recursive down the Nested Components Stack. Sources:
`src/Component.ts` (`_generateRoutingPaths`), `src/routings.ts`, pinned at
`https://github.com/QCObjects/QCObjects/blob/v2.5.142/src/routings.ts`.

- **Declaration:** routings are child elements of the component body carrying a
  `routing` attribute marker; each routing node's attributes become the routing
  object (notably `path`, plus any custom attributes). Paths accumulate into
  `component.routingPaths` and the global `routingPaths` registry.
- **Matching:** `path` is a regex where `{param}` segments become named capture
  groups; `__valid_routings__(routings, routingPath)` filters matches and
  reverses — later declarations win. `__routing_params__(routing, routingPath)`
  extracts the params object.
- **Selection:** `routingSelected` is read-only (setting it only logs); force a
  rebuild with `route()`. The current path resolves per `routingWay`
  (`hash` | `pathname` | `search`, from CONFIG, validated against
  `validRoutingWays`); location changes re-trigger matching.
- **Nesting:** setting `body` triggers the routings builder for that component;
  `__buildSubComponents__` then builds each subcomponent, which builds its own
  routings in turn — so a route selects a chain of component + subcomponents,
  each rendering its matched template. Shadowed components route into their
  `shadowRoot` (`<slot>` content follows the same rules).
- Components that never declare `routing` children match nothing and render
  their default template unconditionally.
- New routable components MUST declare explicit `path`s (no catch-all reliance)
  and MUST list valid `routingWay`s they support.

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
