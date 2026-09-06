# 01 — Product Vision

## Purpose

Define what QCObjects is, who it serves, and the principles every future
decision MUST be judged against. This is the top of the authority chain:
when specs conflict, the vision wins.

Source: `QCObjects` README (identity, principals, features, PWA adoption),
pinned at tag `v2.5.142`
(`https://github.com/QCObjects/QCObjects/blob/v2.5.142/README.md`).
Definitions below are authoritative; the README may be summarized in the future.

## Scope

Covers the product identity across `qcobjects` (core), `qcobjects-sdk`,
`qcobjects-cli`, and app templates. Excludes release scheduling (see
[15-unified-vision-v3.md](./15-unified-vision-v3.md)).

## Identity

- **Name:** QCObjects = **Q**uick **C**omponents and **O**bjects. The Q means Quick.
- **Tagline:** "An open-source framework that empowers full-stack developers to
  make micro-services and micro-frontends into an N-Tier architecture."
- **Thesis:** front-end and back-end coded together with a common syntax in pure
  JavaScript. Cross-browser, cross-platform, cross-frame. No TypeScript, no
  transpiler required to run; pure JavaScript with zero code dependencies.
  Transpilers are NOT required but ARE allowed: apps MAY be authored in
  TypeScript (templates ship `build:ts`), and the framework itself is now
  authored natively in TypeScript (see [03-core-framework](./03-core-framework.md)
  § TypeScript posture) with first-party type declarations.
- **Standard basis:** [ECMAScript® 2020 Language Specification](https://tc39.es/ecma262/)
  (ECMA-262). The `Class` factory (capital C) is the cross-browser declaration
  helper; recent versions ALSO accept native ES `class`/`extends` + `new`
  everywhere the factory works (see [03-core-framework](./03-core-framework.md)
  § Native interop) — new code SHOULD prefer native syntax.
- **Interop:** designed to compose with CSS frameworks (Foundation, Bootstrap)
  and mobile frameworks (PhoneGap, Onsen UI).

## Principals (normative design laws, 0–25)

The framework was built on these principals. A proposal that violates one MUST
justify the violation explicitly or be rejected.

0. Type in JavaScript to code a JavaScript application.
1. Everything is an object.
2. Every object has a definition.
3. On the front-end, any object can be stacked into the DOM or Virtual-DOM
   without re-declaring its definition.
4. Every object has a body.
5. A class SHOULD be the main definition of an object.
6. A class SHOULD be easy to type as an object itself.
7. Code SHOULD organise easily into packages.
8. Code SHOULD scaffold cleanly into a clean architecture.
9. A component is an entity with an object representation and a tag declaration;
   its content MUST be fillable remotely and locally. Its body is normally a
   stacked DOM element instance.
10. A component can attach/detach from the DOM without losing functionality.
11. A service call can be extended to scaffold its functionality.
12. Packages MUST be importable remotely.
13. Scaffolding MUST control server-side savings (no unnecessary remote calls)
    without boilerplate repetition.
14. N-Tier applications MUST be codeable in a single language/syntax.
15. Any template syntax/language MUST be applicable to a component.
16. An already-represented HTML tag MUST NOT need duplicate instance definitions.
17. The HTML shell stays clean; tag behaviour binds without affecting HTML syntax.
18. Execution order MUST be readable from the code; component rendering MUST
   expose execution control in as many layers as needed.
19. A layered pattern (MVC/MVCC) MUST be present for every component, whether
   or not every layer is explicitly defined.
20. Component behaviour MUST NOT be determined by its rendering process.
21. The DOM is split into a subjacent tree of attached elements: the QCObjects
   Nested Components Stack (`global.componentsStack`).
22. Component instances MUST be extendable with dynamic behaviour decoupled from
   the initial declaration.
23. Simultaneous visual effects/animations MUST apply easily to any DOM element.
24. Effects MUST be controllable from CSS or JavaScript without hurting performance.
25. Behaviour MUST be controllable into-the-box and out-of-the-box.

## Main features (binding catalogue)

- Built-in & custom templates for PWA and AMP.
- Revolutionary UI effects (see [04-sdk](./04-sdk.md) effects catalogue).
- Breakthrough backend micro-services (see [02-architecture](./02-architecture.md)).
- Objects- & Components-driven architecture; front-end + back-end full-stack.
- Recursive routing for components; built-in nested components management.
- Fully integrated MVC (Model, View, Controller); Dynamic Data Objects.
- N-Tier architecture concepts; one-step install (textfield / navigate-home).

## PWA adopted features (normative)

- **Prevent render-blocking resources:** implemented via the `Package` factory —
  imports resolve through packages, not blocking script tags.
- **On-demand resource load:** visual resources inside a component render only
  when the component builds itself; every component hangs on
  `global.componentsStack` (instance + subcomponents tree); rebuilds reload
  resources on demand.
- **Lazy image loading (since 2.1.251):** `lazy-src` attribute on `<img>` inside
  components; preloader in `src`, real image in `lazy-src`; Intersection Observer
  API when available; omit `lazy-src` for normal loading.

## Community & sustainability ( pointers, not norms)

- Demos: [PWA live demo app](https://github.com/QCObjects/qcobjects-new-app) (deployed
  at newapp.qcobjects.dev), Foundation/Materialize/raw-CSS
  samples, canvas manipulation example (see spec 07).
- DevBlog (Hashnode), explainer video, Product Hunt, CII Best Practices badge.
- Sponsorship/donations via the README links; Contributor Covenant
  (`CODE_OF_CONDUCT.md`); contributions per `CONTRIBUTING.md`.

## Normative (umbrella)

- QCObjects MUST remain a full-stack framework for micro-services and
  micro-frontends in an N-Tier architecture — not a single-layer UI library.
- The platform MUST preserve three sovereign guarantees:
  1. **No vendor lock-in** — apps run on own infrastructure with own config.
  2. **Local-first execution** — routing, rendering, and orchestration work
     without calling vendor clouds.
  3. **Metadata-driven behavior** — `config.json` is the source of truth;
     code MUST NOT hardcode credentials, paths, or routing.
- Secrets and environment values MUST be injected via `$ENV(VAR)` /
  `$config(key)` placeholders, never committed literally.
- Developer experience MUST stay "one CLI in, running app out":
  scaffold → serve (HTTP/2) → build → publish, all through `qcobjects-cli`.
- Every new capability SHOULD ship as a decoupled, keyword-discovered package
  (handler, server adapter, component set) rather than a core dependency.

## Verification

- Any feature proposal cites which guarantee it strengthens and which principal
  (0–25) it obeys.
- `grep -r "process.env" src/` in core repos returns no direct credential reads
  outside the `$ENV()` resolver.
