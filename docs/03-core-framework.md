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
- `Package(name, [classes])` defines and registers; a bare `Package(name)` call
  with no classes throws (retrieval is synchronous `ClassFactory(name)`, which
  throws when the name is missing).
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
- `waitUntil(func, exp)` runs `func` once when `exp()` turns true (use sparingly).

## Object inheritance (normative)

Concept 1 of 3 (see also: Component inheritance; Nested components routing).
Applies to every QCObjects object — components, controllers, services, views,
models, effects, and plain classes alike. Sources: `src/Class.ts`,
`src/InheritClass.ts`, `src/super.ts`, `src/is_a.ts`,
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

## Component inheritance (normative)

Concept 2 of 3: how inheritance specializes *components* specifically — the
template/class/pairing rules that don't apply to plain objects.

- **Canonical pattern:** subclass a framework component to specialize it —
  `FormField extends Component`; `ButtonField/InputField/TextField/EmailField
  extends FormField` (each fixing a `fieldType` selector: button/input/textarea);
  `GridComponent`,
  `SliderComponent`, splash variants. Prefer subclassing over configuring the
  base with flags.
- **`name` rule:** a subclass inherits the parent's `name` unless it overrides
  it — and `name` drives `templateURI`. A subclass that renders different markup
  MUST set its own `name` (else it silently reuses the parent's template); a
  subclass that only changes behavior SHOULD keep the parent's `name` to reuse
  its template. Unnamed components log a build warning.
- **Pairing inheritance:** `controllerClass`/`viewClass`/`effectClass` pairings
  and `cached`/`tplsource`/`tplextension` settings inherit with the subclass —
  override only what changes (e.g. `SlideItemComponent` fixes
  `effectClass="Fade"`; `GridItemComponent` fixes an inline template).
- **`subcomponentClass` specialization:** spawning parents fix a default child
  in their constructors (`SlideListComponent` and `GridComponent` default to
  `GridItemComponent`) — subclass the parent and fix a narrower
  `subcomponentClass` to specialize a list/grid without touching its logic.
  `DataGridController` only READS the attribute (logs when absent); it sets
  no default.
- **Shadowed inheritance:** `shadowed` inherits; a shadowed subclass of a
  non-shadowed parent (or vice versa) MUST be a conscious choice — mixed trees
  route templates into different roots (`shadowRoot` vs body).

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
  native `new` operator works too — e.g. `New(Move,{…})`, `new Fade(…)`,
  `new i18n_messages({})` (the forms actually used across the SDK sources,
  which are themselves written in native class syntax).
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
`\$name((.*))` and invoked positionally AFTER the component instance, i.e.
`fn(componentInstance, arg1, arg2, …)` with the comma-split args SPREAD
(this is the contract every processor body assumes: `mapper(componentInstance,
componentName, valueName)`, `layout(componentInstance, layoutname, cssfile)`,
`MAILCHIMP_API` joining three env names).
Sources: `src/Processor.ts`, `src/defaultProcessors.ts` (`setDefaultProcessors`),
pinned at `https://github.com/QCObjects/QCObjects/blob/v2.5.142/src/Processor.ts`.

⚠️ KNOWN REGRESSION at the pinned tag: the TypeScript migration (commit
`9b0dbfc`) changed `execute()` from `[component, ...args.split(",")]` (spread,
correct) to `[component, args?.split(",")]` (single array) — so multi-arg
processors (`$mapper`, `$layout`, `$component`, `$MAILCHIMP_API`) receive all
args bundled in one array and misbehave. The contract above is normative;
the code MUST be fixed back to spread (one-line fix in `execute()`).

Default meta processors (always registered):

- `$mapper(componentName,valueName)` — renders a list value (component `data`/
  prop, else global) as `<quick-component name="…" data-k="v" …>` items, one per
  element with its keys as `data-*` attributes.
- `$layout(portrait|landscape, cssfile)` — emits orientation/aspect-ratio
  `@import` rules for the CSS file (mobile-first portrait set + landscape set).
- `$component(name=…, componentClass=…, …)` — emits a
  `<component name="…" componentClass="…" …>` tag declaration.
- `$quick_component(name=…, componentClass=…, …)` — same for `<quick-component>`.
- `$repeat(length, text)` — repeats `text` over `range(length)` (inclusive:
  `range(3)` yields 4 items), substituting only the FIRST `{{index}}` per copy
  (non-global replace).
- `$ENV(VAR)` / `$config(key)` resolve in the same pass where applicable
  (Node/CLI/Collab for `$ENV`; everywhere for `$config`).

Rules: custom meta processors register via `Processor.setProcessor(fn)` with
non-arrow functions (`this` is the handler); names MUST be alphanumeric;
processors MUST be pure string transforms (no DOM writes — return markup);
templates SHOULD prefer `$component`/`$mapper` over hand-concatenated tags.
Multi-arg form is supported — args arrive positionally after the component
instance (production proof: `$MAILCHIMP_API(KEY,SERVER,KEY_LIST)` joins three
env vars with `-`, registered inside the mailchimp lib package itself).
**Processors travel with packages:** an add-on that needs custom placeholders
MUST register them in its own module (lib/handler entry), never ask the app to
register them — the mailchimp lib's `api/*.js` registering `MAILCHIMP_API` at
import time is the canonical pattern.

## Component model (normative)

**Class properties:** `domain`, `basePath` (auto); `templateURI` (use
`ComponentURI({COMPONENTS_BASE_PATH, COMPONENT_NAME, TPLEXTENSION, TPL_SOURCE})`);
`tplsource` (`default`|`none`|`inline`|`external`); `url`, `name`, `method` (default `GET`); `data`
(`{{prop}}` binding; needs `rebuild()` to refresh); `reload` (replace vs append);
`cached` (load template once; static default or per-instance); `routingWay`
(`hash`|`pathname`|`search`, set globally via CONFIG), `validRoutingWays`,
`routingNodes`, `routings`, `routingPath`, `routingSelected`; `subcomponents`;
`body` (plain property — assigning it does NOT rebuild routings; the routings
builder runs from construction and the route flow).

**Methods:** `set/get`, `rebuild()` (via componentLoader), `Cast()`, `route()`,
`fullscreen()/closefullscreen()`, `css(obj)`, `append(child)`, `attachIn(selector)`.

**`<component>` tag attributes:** `name`; `cached="true"` (only `"true"` counts);
`data-*` one-way mock bindings (NOT bidirectional); `controllerClass`;
`viewClass`; `componentClass`; `effectClass`; `template-source` (passed through
as-is — `default`|`none`|`inline`|`external`);
`tplextension` (default `html`).

```html
<component name="main"></component>  <!-- loads ./templates/main[.tplextension] -->
```

**Loaders:** `componentLoader(instance, load_async)` → Promise
(`successStandardResponse{request, component}` / `failStandardResponse{component}`);
instance `__buildSubComponents__(true)` (or exported `buildComponents(element)`)
rebuilds the subtree — there is NO `[element].buildComponents()` element method
(usually automatic anyway).

**MVC:** `Controller` (base; `done()` fires per component load — the hook for
dynamic components), `View`, `VO` (value object), `DDO` (dynamic data object).

## Loading transport: XHR vs `fetch` (normative)

Components and services load over different transports by purpose. Sources:
`src/componentLoader.ts`, `src/serviceLoader.ts`, pinned at
`https://github.com/QCObjects/QCObjects/blob/v2.5.142/src/componentLoader.ts`.

- **Component templates over HTTP(S) → XHR.** `componentLoader` opens an async
  `XMLHttpRequest` with `component.method` (default `GET`), sends the stringified
  `data` as the body, sets `Content-Type: text/html` (skipped on PhoneGap), and
  treats status `200` as success. The `xhr` travels in the standard response as
  `request`, so `done({request, component})` can inspect status/headers.
  Success stores `responseText` as `component.template` (cached when
  `cached:true`), then feeds the component; any other status rejects.
- **`file:` URLs → `fetch`.** XHR cannot reliably read `file:` everywhere, so
  `file:`-scheme template URLs use `fetch(url).then(response.text())` when
  `"fetch" in top` (sync-XHR fallback otherwise). This is the local-preview /
  hybrid-app path — same feed pipeline after the text arrives.
- **Services → one `serviceLoader`, four legs** (dispatch on `service.kind`,
  then runtime — callers never choose; full detail in
  [02-architecture](./02-architecture.md) § `serviceLoader` dispatch detail):
  `rest` + browser → XHR (async forced; headers loop skipping functions;
  `withCredentials`; `200` → `done`, else `fail()` when defined — WARNING: with
  no `fail()` method the promise NEVER settles, it does not reject);
  `rest` + Node → built-in http/https/http2 leg (`useHTTP2` flag, chunk
  accumulation); `mockup`/`local` → no-network `service.mockup()`/
  `service.local()` with `{request: null, …}`; unknown kind → resolved no-op.
  Standalone `serviceLoaderNode` helpers (e.g. the OpenAI package's
  native-https one) parallel the built-in Node leg and MUST keep its shape.
  Test doubles MUST use `kind:"mockup"` (not stub URLs) so tests never touch
  the network.
- **Cache short-circuit:** cached GET components skip the network entirely via
  `ComplexStorageCache` (`alternate` path); non-GET always hits the network.
- Rules: custom loaders MUST preserve the `{request, component|service}`
  standard-response shape; MUST NOT switch template transport to `fetch` for
  HTTP(S) (progress/status semantics live on the `xhr`); services MUST define
  `fail()` whenever non-200 is a reachable outcome; test doubles MUST use
  `kind:"mockup"` (not stub URLs) so tests never touch the network.

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

Concept 3 of 3 (see also: Object inheritance; Component inheritance).
Every component owns its routing table, and subcomponents own theirs —
routing is recursive down the Nested Components Stack. Sources:
`src/Component.ts` (`_generateRoutingPaths`), `src/routings.ts`, pinned at
`https://github.com/QCObjects/QCObjects/blob/v2.5.142/src/routings.ts`.

- **Declaration:** routings are literal `<routing>` child elements of the component
  body, e.g. `<routing path="/one" name="page-one">` (reference: view-stack
  widget example). Each node's attributes become the routing object (`path`,
  `name`, optional per-routing `tplextension`, plus any custom attributes).
  Paths accumulate into `component.routingPaths` and the global `routingPaths`
  registry. (Correction: earlier text said “attribute marker” — the mechanism is
  `querySelectorAll("routing")`, i.e. real elements.)
- **Matching:** `path` is a regex where `{param}` segments become named capture
  groups; `__valid_routings__(routings, routingPath)` filters matches and
  reverses — later declarations win. `__routing_params__(routing, routingPath)`
  extracts the params object.
- **Selection:** `routingSelected` is read-only (setting it only logs); force a
  rebuild with `route()`. The current path resolves per `routingWay`
  (`hash` | `pathname` | `search`, from CONFIG, validated against
  `validRoutingWays`); location changes re-trigger matching.
- **Name → template switch (`_reroute_`):** for every selected routing, the
  component rebuilds `templateURI` from `routing.name` via `ComponentURI`
  (base path + name + `tplextension` — per-routing override or the component's),
  clears the body and `rebuild()`s (`reload=true` is set by the static `route()`
  wrapper, not by `_reroute_` itself). So `name` picks the
  template file (`page-one` → `page-one.html`) while `path` picks when.
- **Consuming the selection:** `routingSelected` is an array — read the current
  view with `.pop().name` (reference pattern: inside `addComponentHelper` after
  `__promise__` resolves, branch notifications/effects on the name). Dynamic
  `{param}` values come from `__routing_params__`; with `assignRoutingParams`
  set they merge into template `data` at `parseTemplate` time.
- **Defaults & navigation:** declare catch-alls as empty/last paths (`/`, ``);
  with `routingWay:"pathname"`, plain `<a href="/one">` anchors drive the
  switch — no router calls needed. Pair routed views with `effectClass`
  (e.g. a `TransitionEffect` of Fade+Move) for animated transitions.
- **Nesting:** setting `body` triggers the routings builder for that component;
  `__buildSubComponents__` then builds each subcomponent, which builds its own
  routings in turn — so a route selects a chain of component + subcomponents,
  each rendering its matched template. Shadowed components route into their
  `shadowRoot` (`<slot>` content follows the same rules).
- Components that never declare `routing` children match nothing and render
  their default template unconditionally.
- New routable components MUST declare explicit `path`s (no catch-all reliance)
  and MUST list valid `routingWay`s they support.
- **Custom routing management (escape hatch):** canonical routing above covers
  standard cases, but a component class MAY implement its own routing entirely —
  reference `example2-routing.html`: a `RoutingComponent` builds `routings` from
  `<routing>` nodes in `_new_`, overrides `_reroute_()` (exact-match on
  `document.location[routingWay]`, template switch, body clear + `rebuild()`),
  exposes `route()` sweeping `GLOBAL.componentsStack` by `__classType`, and is
  driven by a `popstate` listener. Custom routers MUST reuse the `<routing>`
  declaration shape and the `routingSelected`/`templateURI`/`rebuild()` protocol
  above so nested children keep working; custom matching semantics MUST be
  documented on the class.

## Template handlers (normative)

Every component renders its template through a handler class — swappable per
component, which is the framework's other-framework-interop seam. Sources:
`src/Component.ts` (`parseTemplate`), `src/DefaultTemplateHandler.ts`, pinned at
`https://github.com/QCObjects/QCObjects/blob/v2.5.142/src/DefaultTemplateHandler.ts`.

- **Default:** `Component.templateHandler = "DefaultTemplateHandler"`.
  `parseTemplate` resolves the name via `ClassFactory` (fully-qualified custom
  names work), instantiates `New(HandlerClass, {component, template})`, merges
  `routingParams` into the data when the component sets `assignRoutingParams`,
  and returns `instance.assign(data)`.
- **Contract for custom handlers:** constructor takes `{component, template}`;
  `assign(data) -> string` returns the rendered markup. (`templateHandler` is a
  class field so the raw-passthrough `else` branch is unreachable on normal
  `Component` instances — passthrough applies only to non-`Component` callers.)
- **Default semantics** (`DefaultTemplateHandler.assign`): for each
  string/number datum, run it through `processObject` (meta processors resolve
  inside values too), `{{key}}` global-replace across the template, then
  `processObject` over the whole result. Non-object `data` skips binding with a
  debug line; processor failures throw naming the component.
- **Interop:** a custom handler MAY parse/emit any syntax — set
  `templateHandler` to a registered handler class name on exactly the components
  that need it (e.g. a React-rendered subtree, Mustache/Handlebars templates).
  Handler choice is per-component, so hybrid apps MUST document which components
  use non-default handlers and their syntax.

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
- **Third-party lib integration (reference: QR scanner app):** vendor the lib
  under `js/packages/thirdparty/libs/<lib>/` (with its LICENSE), then chain-load
  it from the controller via `loadDependencies(callback)`:
  `CONFIG.get("<lib>-path", "<vendored default>")` locates the base,
  `CONFIG.get("<lib>-external", false)` flips vendored vs CDN, and nested
  `New(SourceJS,{url, external, done})` pushes ordered dependencies (worker
  before lib), calling back when ready. Query live DOM through
  `component.shadowRoot.subelements(selector)` (`subelements` works on
  `ShadowRoot` directly). Headless `New(Component,{templateURI:"", body: el,
  tplsource:"none"})` MAY wrap raw elements as throwaway component instances
  for framework-flavored DOM utilities.

## Effects, Timer, codecs (normative)

- Custom effects extend `Effect` and override `apply`, delegating via
  `_super_('Fade','apply').apply(this,arguments)`; engine runs on
  `requestAnimationFrame` and mutates CSS smartly.

## Transition effects + `apply-effect-to` (normative)

Sources: `src/TransitionEffect.ts`, `src/Component.ts`
(`createEffectInstance`, `applyTransitionEffect`, `applyObserveTransitionEffect`),
pinned at `https://github.com/QCObjects/QCObjects/blob/v2.5.142/src/TransitionEffect.ts`.

- **Declaration:** `effectClass="<TransitionEffect subclass>"` on the component
  tag/body + `apply-effect-to="<mode>"` (absent = `"load"`). Only two modes exist:
  `load` (apply immediately at build) and `observe` (apply on first visibility).
  Any other value applies nothing.
- **`load` path** (`applyTransitionEffect`): resolves `effectClass` via
  `ClassFactory` (unknown name throws), requires a `TransitionEffect` subclass
  (anything else logs and skips), instantiates `New(Effect,{component})`, and
  calls `.apply(defaultParams)`.
- **`observe` path** (`applyObserveTransitionEffect`): watches
  `componentRoot` (`shadowRoot` when shadowed, else `body`) with an
  `IntersectionObserver`; on first intersect it applies once and unobserves.
  Without `IntersectionObserver`, it applies immediately (same as `load`).
  Browser-only.
- **`TransitionEffect` mechanics** (package
  `com.qcobjects.effects.transitions.base`): `effects[]` lists effect class
  names applied in order, each resolved via `ClassFactory` and invoked with the
  full param set (`alphaFrom/To`, `angleFrom/To`, `radiusFrom/To`,
  `scaleFrom/To`) — defaults `alpha 0→1`, `angle 180→0`, `radius 0→30`,
  `scale 0→1`, `duration` 385. `fitToHeight`/`fitToWidth` size the root from
  its `offsetParent`/bounding rect first; the root (or shadow host) is forced
  `display:block` before effects run.
- **Canonical example** (view transitions):
  `Class("MainTransitionEffect",TransitionEffect,{duration:2500,
  defaultParams:{alphaFrom:0, alphaTo:1}, effects:["Fade","MoveXInFromRight"],
  fitToHeight:true})` + `effectClass="MainTransitionEffect"
  apply-effect-to="observe"` — fade+slide-in the first time each view scrolls
  into view.
- Rules: effect names in `effects[]` MUST all resolve (one typo skips nothing —
  resolution throws); `observe` SHOULD be preferred for below-fold content,
  `load` for above-fold entrances; custom transitions MUST extend
  `TransitionEffect` (not raw `Effect`) to participate in this protocol.
- `Timer.thread({duration, timing(fraction,elapsed), intervalInterceptor(progress)})`
  emulates threads (modern browsers only).
- `_Crypt`: `New(_Crypt,{string,key})._encrypt()/._decrypt()`, or static
  `_Crypt.encrypt(text,key)` / `_Crypt.decrypt(cipher,key)`.
- `ComplexStorageCache({index, load, alternate})` + `getCached(id)` for
  localStorage object caching.
- `asyncLoad(fn, args)` runs once after the async queue, before Ready.
- `ArrayList` (`New(ArrayList,[...])`), `ArrayCollection` (array passed directly
  to `_new_`, not `{source}`),
  `.unique()`, `.table()` (unguarded `console.table` — works anywhere, not shell-only), `.sort()`, `.sortBy(prop)`,
  `.matrix(n[,v])`, `.matrix2d`, `.matrix3d`, `range(n|a,b)`,
  `.sum()`, `.avg()`, `.min()`, `.max()`.

## Verification

- `node -e "require('qcobjects')"` and `import 'qcobjects'` both resolve.
- `npx tsc --noEmit -p tsconfig.d.json` passes; jasmine suite passes.
- Every symbol above resolves at the pinned tag; drift opens a spec-update PR.
