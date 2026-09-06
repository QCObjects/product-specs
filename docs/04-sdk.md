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

- `npm install qcobjects-sdk@v2.4` (line as documented; pin per release).
- Straight HTML: `https://cdnjs.cloudflare.com/ajax/libs/qcobjects/2.4.20/QCObjects.js`.
- NOTE (binding): the SDK dependency ships inside the QCObjects runtime by
  default — install separately only when default paths fail.
- The SDK MUST depend on `qcobjects` core and MUST NOT depend on
  `qcobjects-cli` or any server code.

## Module export table (normative)

The SDK MUST export, at minimum (CJS + ESM + browser + types):
`controllers`, `controllers.grid|slider|form|list|swagger`, `views`,
`components`, `components.grid|list|slider|splashscreen|notifications`,
`modal.controllers`, `effects`, `tools.canvas|layouts`,
`i18n_messages`, `models`, `cloud.auth.session.usertoken|data`, and the
`QCObjects-SDK` bundle.

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
- **GridComponent** (reserved name `"grid"`) + **GridController** → CSS grid;
  `rows`/`cols` attrs; subcomponents recommended as cells; provide
  `grid.tpl.html` (`<p>Loading grid...</p>`).
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
  `data-background`, `data-video_mp4|_webm|_ogg`, `duration="5000"`,
  `<img slot="logo">`; main component follows with `splashscreen` attr
  (`<layout-basic splashscreen name="main" cached=true ...>` in widget syntax).
- **SplashScreenComponent** — base splash (extended by video + cube variants).
- **CubeSplashScreenComponent** — 3D spinning-cube splash (shadowed, inline
  template with `spin` keyframes).
- **NotificationComponent** — notification shell. Drift note: registered under
  the legacy `org.quickcorp.components.notifications` package — the only SDK
  module still on the old namespace; rename to `org.qcobjects.*` when touched.
- Visual assets live under `src/css` + `src/templates`; class logic MUST NOT
  inline large CSS blobs.

## Controllers catalogue

- **GenericController** — empty `Controller` extension point; extend it (instead
  of raw `Controller`) when a controller needs no built-in behavior yet.
- **GridController** — with GridComponent (see above).
- **DataGridController** — maps `data[]` onto `subcomponentClass` instances
  (e.g. profile cards: `CardComponent` template `card.tpl.html` with
  `{{profilePicture}} {{name}} {{email}}`; list shell `loading_list.tpl.html`).
- **ListController** — with ListComponent/ListItemComponent (see above).
- **SliderController** — autoplay for SliderComponent: `duration` default 7100ms,
  `slideIndex`, `interval`; API `plusSlides(n)`, `plusSlidesAndStop(n)`,
  `currentSlide(n)`, `stop()`; shadow-aware (`shadowRoot` when shadowed, else
  `body`); registers itself globally as `slider_<instanceID>`.
- **ModalController** — modal behavior.
- **FormValidations** — `FormValidations.getDefault(name)` default validators.
- **FormController** — 3-step forms: (1) `serviceClass` string (resolved via
  ClassFactory, may be fully qualified), (2) `formSettings`
  (`backRouting` on fail / `loadingRouting` while calling / `nextRouting` on OK;
  defaults `'#'` / `'#loading'` / `'#signupsuccessful'`), (3) `validations`
  (`field(){ return (fieldName, dataValue, element)=>bool }`).
  `formSaveTouchHandler` submits on click/touch of any `.submit` element —
  override to change. Safe-extension pattern: extend `Controller`, keep a
  `defaulController = new FormController(o)` wired in `_new_(o)`, delegate in
  `done()` (see README signup example: `SignupClientService extends JSONService`
  POST + `SignupFormController` + shadowed `signup-form` template with slots).
- **SwaggerUIController** — with SwaggerUIComponent.

## Effects catalogue (all `requestAnimationFrame`-based, CSS-smart)

`Move.apply(el,x1,y1,x2,y2)`; `MoveXInFromRight/Left.apply(el)`;
`MoveYInFromBottom/Top.apply(el)`; `RotateX/Y.apply(el,aFrom,aTo)`,
`RotateZ`, `Rotate` (3D parallel, degrees 0–360);
`Fade.apply(el,alphaFrom,alphaTo)` (0–1);
`Radius.apply(el,rFrom,rTo)`; `Resize.apply(el,sFrom,sTo)` (1 = normal);
`WipeLeft/Right/Up/Down.apply(el,sFrom,sTo)`.
Batch via `Tag(...).map(el => (new X()).apply(el, …))`.

Modal presets (`org.qcobjects.modal.effects`, internal — imported by the
components module, no separate package export): `ModalFade extends Fade`
(500ms), `ModalMoveUp extends Move` (800ms), `ModalMoveDown extends Move` (300ms).

## Models, cloud session, tools, views, i18n

- `org.qcobjects.models.Contact extends VO` — canonical value-object example;
  extend `VO` (not plain objects) for model data.
- `org.qcobjects.cloud.auth.session.data.SessionData` and
  `...session.usertoken.SessionUserToken` (both `InheritClass`) — token/data
  holders for cloud-auth sessions; MUST go through the core storage cache,
  MUST NOT log or persist raw passwords.
- `org.qcobjects.tools.canvas.CanvasTool`, `org.qcobjects.tools.layouts.BasicLayout`.
- `org.qcobjects.tools.Process extends Timer` — named thread entry.
- `org.qcobjects.views.GridView` (generic grid view).
- `org.qcobjects.i18n_messages.i18n_messages` — subclass per lang
  (`class i18n_messages_es extends i18n_messages` with `messages:[{en,es}…]`),
  instantiate and attach in the package
  (`_i18n_messages_es: new i18n_messages_es()`).

## Verification

- Demo app renders each exported component with only core + SDK installed.
- `npm run build` regenerates `build/` + `public/`; `npm test` (eslint + jasmine) green.
- Full signup-form example (service + controller + shadowed templates) works verbatim.
