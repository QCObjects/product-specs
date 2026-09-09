# 04 — SDK (`qcobjects-sdk`)

## Purpose

Specify the SDK: the reusable MVC building blocks apps compose from — with the
complete component/controller/effect catalogue, so the SDK README can be
summarized without loss.

Source: `qcobjects-sdk` README (`v2.5.105`).
Code pins: `https://github.com/QCObjects/qcobjects-sdk/blob/v2.5.105/src/ts/<module>.ts`.
Definitions below are authoritative.

## Scope

Repo `QCObjects/qcobjects-sdk`, npm `qcobjects-sdk` (`v2.5.105`).
Sources in `src/ts/org.qcobjects.*.ts`, templates in `src/templates`.

## Install & load (normative)

- `npm install qcobjects-sdk@v2.5.105` (pin per release).
- Straight HTML: the SDK browser bundle (`QCObjects-SDK` bundle /
  `public/browser/index.js`), not the core `QCObjects.js` CDN file.
- NOTE (binding): the SDK is NOT bundled inside the core runtime — with
  `useSDK:true` (core default) it is auto-loaded from `remoteSDKPath`
  (`https://sdk.qcobjects.dev/`) in browsers, or `require("qcobjects-sdk")`
  from `node_modules` in Node. Install separately only when default paths fail.
- The SDK MUST depend on `qcobjects` core and MUST NOT depend on
  `qcobjects-cli` or any server code.

## Package namespaces (normative — ClassFactory/Import MUST use these exact names)

- `org.qcobjects.form.components`: `ShadowedComponent`, `ButtonField`,
  `InputField`, `TextField`, `EmailField`, `ModalEnclosureComponent`,
  `ModalComponent`, `SwaggerUIComponent`.
- `org.qcobjects.base.components`: `FormField`.
- `org.qcobjects.components.grid`: `GridItemComponent`, `GridComponent`.
- `org.qcobjects.components.list`: `ListItemComponent`, `ListComponent`.
- `org.qcobjects.components.slider`: `SlideListComponent`, `SlideItemComponent`,
  `SliderComponent`.
- `org.qcobjects.components.splashscreen`: `VideoSplashScreenComponent`,
  `CubeSplashScreenComponent`; `org.qcobjects.components.base`:
  `SplashScreenComponent`.
- `org.qcobjects.controllers`, `.grid`, `.list`, `.slider`, `.form`,
  `.swagger`, `org.qcobjects.modal.controllers`: as named per file.
- `org.qcobjects.modal.effects`: `ModalFade`, `ModalMoveDown`, `ModalMoveUp`
  (a registered package — importable via ClassFactory).
- There is NO `Package("org.qcobjects.components")` — bare `ClassFactory` /
  `Import` on that name fails.

## Module export table (normative)

The SDK MUST export, at minimum (CJS + ESM + browser + types):
`controllers`, `controllers.grid|slider|form|swagger`, `views`,
`components`, `components.grid|list|slider|splashscreen|notifications`,
`modal.controllers`, `effects`, `tools.canvas|layouts`,
`i18n_messages`, `models`, `cloud.auth.session.usertoken|data`, and the
`QCObjects-SDK` bundle.
(Gap on record: `org.qcobjects.controllers.list` exists in `src/` but has NO
`./js/org.qcobjects.controllers.list` subpath export in `package.json` —
`ListController` resolves via ClassFactory/package, not via deep import.)

## Components catalogue

- **ShadowedComponent** — Shadow-DOM custom components.
  `<component componentClass="ShadowedComponent">` or
  `<my-custom-widget componentClass="ShadowedComponent">`.
  Widgets register via `RegisterWidget("signup-form")`.
- **FormField** — generic form behavior with *reverse* data binding (no
  observable overhead): put `data-field="<prop>"` on inner DOM tags; set
  `componentClass="FormField"` on the tag; read `componentInstance.data`.
  `executeBindings()` matches `data-field`s to `data` fields; triggered by
  `change`, `blur`, `focus`, `keydown` inside the body.
- **ButtonField** (`<button>` body) / **InputField** (`<input>` body) /
  **TextField** (`<textarea>` body) / **EmailField** (`<input>` body) — all
  FormField sub-definitions, same usage with their tag default body.
- **GridComponent** (reserved name `"grid"`, inline `<p>Loading...</p>`
  template) forces `controllerClass="DataGridController"` in its constructor —
  pair it with `DataGridController`, not `GridController` (the CSS-only
  variant); `rows`/`cols` attrs; subcomponents recommended as cells.
- **GridItemComponent** (name `"grid-item"`, shadowed, inline template
  `<img src="{{image}}"/><p>{{description}}</p>`) — the default cell used when
  a grid-like controller needs a `subcomponentClass` and none is given.
- **ListComponent + ListItemComponent + ListController** — vertical-list analogue
  of the grid trio: `ListController` drives `ListItemComponent` instances inside
  `ListComponent` from `data[]`.
- **SliderComponent** (name `"slider"`, shadowed) + **SlideListComponent**
  (name `"slidelist"`, inline `<p>Loading...</p>`; forces
  `controllerClass="DataGridController"` and defaults
  `subcomponentClass="GridItemComponent"`) + **SlideItemComponent**
  (name `"slider_item"`, `Fade` effect; inline `qcoSlides` template binding
  `{{slideNumber}} {{__dataLength}} {{image}} {{title}} {{label}} {{link}}
  {{category}}`, with `slideNumber = __dataIndex + 1`).
- **ModalEnclosureComponent / ModalComponent** — modal shells (pair with
  ModalController).
- **SwaggerUIComponent** (+ SwaggerUIController) — injects Swagger-UI DOM.
- **VideoSplashScreenComponent** — video splash: first tag in the document,
  `data-background`, `data-video_mp4|_webm|_ogg`, `duration` (example `"5000"`;
  absent-attribute default is `1000`),
  `<img slot="logo">`; main component follows with `splashscreen` attr
  (`<layout-basic splashscreen name="main" cached=true ...>` in widget syntax).
- **SplashScreenComponent** — base splash (extended by video + cube variants).
- **CubeSplashScreenComponent** — 3D spinning-cube splash (shadowed, inline
  template with `spin` keyframes).
- **NotificationComponent** — notification shell. Drift note: registered under
  the legacy `org.quickcorp.components.notifications` package (the i18n loader
  also references an `org.quickcorp.*` namespace); rename to `org.qcobjects.*`
  when touched.
- Visual assets live under `src/css` + `src/templates`; class logic MUST NOT
  inline large CSS blobs.

## Controllers catalogue

- **GenericController** — empty `Controller` extension point; extend it (instead
  of raw `Controller`) when a controller needs no built-in behavior yet.
- **GridController** — CSS-only grid variant (does NOT pair with
  `GridComponent`, which forces `DataGridController` — see above).
- **DataGridController** — maps `data[]` onto `subcomponentClass` instances
  (e.g. profile cards: `CardComponent` template `card.tpl.html` with
  `{{profilePicture}} {{name}} {{email}}`; list shell `loading_list.tpl.html`).
- **ListController** — with ListComponent/ListItemComponent (see above).
- **SliderController** — autoplay for SliderComponent: `duration` default 7100ms,
  `slideIndex`, `interval`; API `plusSlides(n)`, `plusSlidesAndStop(n)`,
  `currentSlide(n)`, `stop()`; shadow-aware (`shadowRoot` when shadowed, else
  `body`); registers itself globally as `slider_<instanceID>`.
- **ModalController** — modal behavior.
- **FormValidations** — instance validators: `(new FormValidations(o)).getDefault()`
  returns a `(fieldName, dataValue, element)=>bool` checker (name/email regexes
  or the element's own `pattern` attribute).
- **FormController** — 3-step forms: (1) `serviceClass` string (resolved via
  ClassFactory, may be fully qualified), (2) `formSettings`
  (`backRouting` on fail / `loadingRouting` while calling / `nextRouting` on OK;
  defaults `'#'` / `'#loading'` / `'#signupsuccessful'`), (3) `validations`
  — an ARRAY-like keyed per field: `validations[fieldName](fieldName, dataValue,
  element)`, NOT a `field(){ return fn }` wrapper shape.
  `formSaveTouchHandler` submits on click/touch of any `.submit` element —
  override to change. ⚠️ KNOWN ISSUE: `done()` calls `this.onpress(".submit",…)`
  but `FormController` overrides `onpress` to `throw new Error("Method not
  implemented.")` — submitting through `done()` throws until this is fixed;
  the safe-extension pattern (extend `Controller`, keep a `defaulController =
  new FormController(o)` wired in `_new_(o)`, delegate in `done()`) is the
  workaround (see README signup example: `SignupClientService extends JSONService`
  POST + `SignupFormController` + shadowed `signup-form` template with slots).
- **SwaggerUIController** — with SwaggerUIComponent.

## Effects catalogue (all `requestAnimationFrame`-based, CSS-smart)

Calling form matters — static-only vs instance-only is per class:

- Static `X.apply(el, …)`: `Move`, `MoveXInFromRight/Left`, `MoveYInFromBottom/Top`,
  `RotateX`, `RotateY` — `(new Move()).apply` is `undefined` and throws.
- Both forms: `Fade` only (`Fade.apply(el,aFrom,aTo)` or `(new Fade()).apply(…)`).
- Instance-only `(new X()).apply(el, …)`: `RotateZ`, `Rotate` (3D parallel,
  degrees 0–360), `Radius`, `Resize` (1 = normal), `WipeLeft/Right/Up/Down`.
- Batch via `Tag(...).map(el => X.apply(el, …))` for static classes,
  `Tag(...).map(el => (new X()).apply(el, …))` for instance classes.
- **Effects-dispatch pattern (reference: effects demo app):** expose an
  `effects: {apply<Name>(el){…}}` map on the controller plus an
  `applyEffect(name)` dispatcher (`this.effects["apply"+name](el)`); generate
  trigger buttons with a custom meta processor emitting BOTH `ontouchstart`
  and `onclick` handlers (touch-first devices); register the controller in
  `global` (`global.set("mainControllerInstance", this)` in `done()`) so
  generated markup can reach it. Composed moves (slide/fall/rise) chain static
  `Move.apply` calls with measured offsets (`clientWidth`/`clientHeight`);
  rotates SHOULD set `transformOrigin` first.
- **`tplextension` is free-form:** any extension value works
  (`<name>.<tplextension>`) — the TEMPLATE HANDLER class must support it.
  Text formats are natively supported (`svg`, `md`, `txt` — reference: clickable
  octocat via `tplextension="svg"`); non-text formats REQUIRE a custom handler
  that parses them (see [03-core-framework](./03-core-framework.md) § Template handlers).

Modal presets (`org.qcobjects.modal.effects` — a REGISTERED package,
importable via `ClassFactory("org.qcobjects.modal.effects.ModalFade")`;
only the npm subpath export is absent): `ModalFade extends Fade`
(500ms), `ModalMoveUp extends Move` (800ms), `ModalMoveDown extends Move` (300ms).

## Models, cloud session, tools, views, i18n

- `org.qcobjects.models.Contact extends VO` — canonical value-object example;
  extend `VO` (not plain objects) for model data.

## Session handling (normative)

Sources: `src/ts/org.qcobjects.cloud.auth.session.{usertoken,data}.ts`, pinned at
`https://github.com/QCObjects/qcobjects-sdk/blob/v2.5.105/src/ts/org.qcobjects.cloud.auth.session.usertoken.ts`.
Status: `beta` (see [11-features](./11-features.md)) — API shape may still move.

- **Token issuance** (`SessionUserToken extends InheritClass`): one singleton per
  username in `global` under `userToken_<base64(username)>` (`getGlobalUser(...)`
  creates-or-returns). The token is `_Crypt.encrypt("userAgent|username|timestamp",
  origin-or-domain)` held in a `ComplexStorageCache` keyed by instance ID
  (first access encrypts via `load`, later accesses hit `alternate`/cache).
  Accessors: `getGlobalUser{,Token,Id,Priority}(username)`.
- **Login credentials:** `getLoginCredentialsToken(username, password)` =
  `_Crypt.encrypt(username+password, userToken)` — the password never travels or
  persists raw; only the derived credential token leaves the client.
- **Logout:** `closeGlobalSession(username)` clears the token cache, nulls the
  global slot, and resets `SessionUserToken.user` to `{}`. NOTE:
  `ComplexStorageCache.clear()` wipes ALL `cachedObject_*` keys — component and
  service caches go too, not just the token.
- **Session data** (`SessionData extends InheritClass`): `sessionStorage`-backed,
  keyed `session_<btoa(userToken)>` so each login's data is namespaced by its
  token. (The `index()` missing-import guard is unreachable behind the static
  import — treat the import as mandatory, not the error.)
  A session container MUST be set first
  (`setSessionContainer(...parts)`; `getSessionContainer()` throws when unset);
  `save(...)` stringifies `sessionData` into the slot, `getSessionData(...)`
  parses it back (`{}` when absent).
- **Rules:** session reads/writes MUST go through these classes (never raw
  `sessionStorage` keys); tokens MUST NOT be logged; credential tokens MUST be
  re-derived per login, never stored; closing a session MUST clear both the
  token cache and the `sessionStorage` slot.
- `org.qcobjects.tools.canvas.CanvasTool`, `org.qcobjects.tools.layouts.BasicLayout`.
- `org.qcobjects.tools.Process extends Timer` — registry-only anonymous class;
  ⚠️ its `thread()` override throws `Method not implemented`, so `start()`
  always throws. Do not use until fixed.
- `org.qcobjects.views.GridView` (generic grid view).
- `org.qcobjects.i18n_messages.i18n_messages` — subclass per lang
  (`class i18n_messages_es extends i18n_messages` with `messages:[{en,es}…]`),
  instantiate and attach in the package
  (`_i18n_messages_es: new i18n_messages_es()`).

## Verification

- Demo app renders each exported component with only core + SDK installed.
- `npm run build` regenerates `build/` + `public/`; `npm test` (eslint + jasmine) green.
- Full signup-form example (service + controller + shadowed templates) works verbatim.
