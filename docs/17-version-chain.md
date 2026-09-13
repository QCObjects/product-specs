# 17 — Version Chain (framework idempotency under multi-copy resolution)

## Purpose

State the root cause of the historic `--legacy-peer-deps` workaround, define the
**framework-idempotency** property the three framework packages must uphold, and
propose a **coordinated release train** ("version chain") that prevents version
pollution and dependency-circle collapse without merging the packages.

## Scope

The three framework packages — `qcobjects` (core), `qcobjects-sdk`, and
`qcobjects-cli` — and how their versions and dependency declarations relate
across the ecosystem. Add-on packages ([16-addons](./16-addons.md)) are affected
by the same resolution mechanics and are covered where noted, but their
lifecycle stays in spec 16.

## The problem, named (normative diagnosis)

The framework is **stateful and single-instance by construction**: `qcobjects`
holds global singletons (the `_top` context, `Package`/`ClassFactory` registries,
`CONFIG` shared state, `ComplexStorageCache`, the `__qcobjects_sdk__` ready
flag). A correct app therefore requires **exactly one copy** of each framework
package in its dependency graph. This is the **framework-idempotency**
property: *"whatever a consumer declares, the resolved graph must collapse N
declarations of `qcobjects` / `qcobjects-sdk` into one instance."*

The current design does **not** enforce idempotency; it *hopes* for it. The
declaration model is peer-to-peer:

- `qcobjects-cli` declares `qcobjects` and `qcobjects-sdk` as **peer
  dependencies** so the *app* selects the framework version (the CLI ships no
  copy).
- Add-ons ([16-addons](./16-addons.md)) are expected to do the same.

The failure is that **npm's treatment of peer dependencies changed across
npm/Node versions and is not uniform**:

1. npm 7+ **auto-installs** peers — a peer dep can be materialised as its own
   resolved copy.
2. In some resolutions, a package declaring `qcobjects`/`qcobjects-sdk` as a
   **direct dependency** (not a peer) hoists its own copy **above** the app's,
   so the app's declared instance is shadowed.
3. In others, peers are resolved *as if* dependencies, producing **duplicate
   copies** rather than deduplicating to the app's.

Every one of these yields **two `qcobjects` instances** (or two `qcobjects-sdk`)
in one graph. Because the framework is stateful, two instance means a Component
built by the app's core can be invisible to a wrapper/controller resolved from an
add-on's core — registries diverge, shared state splits, and the ready/startup
flags misfire.

### Why `--legacy-peer-deps` masked, not fixed, this

The historical instruction `npm i --legacy-peer-deps` (present across
[05-cli](./05-cli.md), [08-ci-conventions](./08-ci-conventions.md),
[10-contribution-pipeline](./10-contribution-pipeline.md), and repo AGENTS
files) does **not** make the framework idempotent. It only tells npm to *stop
installing* the peer — which prevents the duplicate copy in *some* cases while
leaving the underlying single-instance-vs-N-copies assumption unenforced. It is
a per-command workaround, not a property.

### Rejected framing

An earlier consolidation proposal (monorepo / single `qcobjects` package) was
**rejected** because merging the packages does not establish idempotency
cleanly: a merged framework package bloat every consumer (browser-only apps
would carry the CLI's native/CLI dependencies) and merely *relocates* the
resolution risk rather than removing it. A separate `qcobjects-build` package
was likewise **rejected** as it adds a fourth version surface without addressing
the cross-dependency cycle. The three packages remain separate.

## The three failure surfaces (normative)

1. **CLI:** dev/build-time only for apps, runtime for servers. Peers on core+SDK.
   Risk: a second core/SDK copy resolves inside the CLI's own tree when peers are
   auto-installed or treated as direct deps.
2. **Browser-only apps:** need core+SDK at runtime, the CLI only at build time.
   Risk: the build-time CLI install drags its peer resolution into the runtime
   graph (or vice versa), yielding a divergent core/SDK.
3. **Add-ons:** declare core+SDK as peers *or* direct deps. Risk: a direct-dep
   declaration shadows the app's instance in versions where hoisting wins.

In every surface the *same* invariant is violated: the single-instance
assumption is not enforced by the module system, and no mechanism collapses
duplicate declarations to one instance.

## Proposal — coordinated release train ("version chain")

The framework packages MUST be released in lockstep so that their versions are a
**single coordinated point** consumers resolve, and their declarations MUST be
shaped so resolution cannot fork. This is the **version chain**.

### Chain invariants (normative)

- **V1 — Lockstep versioning.** At each coordinated release, `qcobjects`,
  `qcobjects-sdk`, and `qcobjects-cli` ship the **same version number** (e.g.
  `2.7.0` for all three), released from one tag/one pipeline window. There is no
  independently-versioned framework package.
- **V2 — Exact-pin peer contracts, no ranges.** Framework packages MUST NOT
  declare open ranges (`>=2.5.142 <3.0.0`, `^2.5`, etc.) against each other.
  Peers/dependencies MUST pin the exact coordinated version (`2.7.0`) so every
  consumer resolves a single, known instance. Ranges are what allow resolution
  to fork; exact pins are what prevent it.
- **V3 — Single canonical consumer face.** An app selects the framework by
  pinning the chain version **once**; the CLI and add-ons treat core/SDK as
  peers pinned to that same chain version. No package in the chain is ever a
  *direct* dependency of another chain member (this is what triggers hoisting
  shadow-copies).
- **V4 — Idempotent single-instance enforcement.** Resolution MUST be verified
  in CI: an app (or a canary consumer fixture) MUST install with the framework
  appearing **exactly once** (`npm ls qcobjects` yields one entry, not a tree),
  and this MUST be asserted in the release gate (see Verification).

### What the chain changes vs today

| Today (specs 05/08) | Version chain |
|---|---|
| Each framework repo versions+publishes independently | One lockstep version for all three |
| `>=` ranges in peer/deps | Exact-pin peers only |
| `--legacy-peer-deps` required everywhere | Pin + single-instance CI gate; legacy flag tolerated but no longer the correctness mechanism |
| No cross-repo release coordination | One tag window releases all three, idempotency verified |

### Relationship to existing specs

- `05-cli.md` § "Core libraries" stays correct in *that* the CLI ships no copy
  and peers on core+SDK — but its range-based resolution guidance MUST be
  replaced with exact-pin peers.
- `08-ci-conventions.md` release model gains a **cross-repo lockstep** step: the
  coordinated release is one operation across three repos, not three independent
  tag pushes.
- `16-addons.md` keyword contract is unchanged; add-ons additionally MUST pin
  the chain version (not a range) to avoid shadow-copies.
- This spec does **not** propose merging packages (monorepo/single-package
  rejected above) and does **not** propose a new build package.

## Global-scope dependency (normative finding, carry-forward)

The framework's single-instance global context relies on a **writable,
redefinable `global` property on the ambient object** (`window` | `global` |
`self` | `top` | `globalThis`). This is a **non-portable assumption**:

- **ECMAScript** does not specify the ambient globals `window`/`global`/`self`;
  only `globalThis` (ES2020 § 18.6.1) is defined.
- **Node** exposes `globalThis.global` as a writable *data* property.
- **Browsers** expose `window.global` as a **read-only accessor** whose
  `configurable`/redefinability is *host-defined and context-dependent* — not
  guaranteed by any standard.

The concrete failure sequence observed (2026-09-13):

1. `set("global", window)` (i.e. `_top.global = window`) threw
   `TypeError: Cannot set property global ... only a getter` under the strict-mode
   esbuild bundle.
2. Replacing it with `Object.defineProperty(_top, "global", {
   writable:true, configurable:true, enumerable:true, value })` **passed** in a
   Chrome DevTools console but **failed** (`TypeError: Cannot redefine property:
   global`) in the same Chrome when run inside the built app — because the
   property's redefinability is not stable across the two evaluation contexts.

**Conclusion:** any mechanism that *writes or redefines* the ambient `global`
property is ECAM-non-compliant-by-omission and environment-volatile, and MUST
NOT be relied upon. The durable direction is to **remove the framework's global
scope dependency** — carry the unified context as an explicit module export /
owned object rather than a synthesized property on the ambient global — and
treat `window`/`global`/`self` as read-only detection sources only.

This is NOT resolved; it is a carry-forward blocker tracked alongside the open
design questions below.

## Open design questions (to resolve before codifying)

These MUST be answered before this proposal becomes binding; until then the
chain is a **proposal**, not a ratified contract:

1. **Lockstep mechanism:** one tag committed across three repos, or one
   orchestrator release job that tags all three? (Determines whether V1 is
   enforceable in CI or only by convention.)
2. **Exact-pin consequences:** exactly-pinned peers reintroduce the
   dependency-tree-collision the old era suffered (a reason `--legacy-peer-deps`
   was adopted). Does exact-pinning make `npm dedupe`/resolution reliably
   single-instance across the npm-versions matrix, or does a single-instance
   *enforcement layer* (e.g. a runtime identity check that two distinct
   `qcobjects` versions must not both load) become necessary?
3. **Rollout:** which consumers (templates, add-ons, enterprise) migrate first,
   and what is the compatibility window during which range-based consumers still
   resolve?
4. **Global-scope removal:** the single-instance `global` currently lives on the
   ambient object and requires a writable/redefinable `global` property — which
   is non-portable (see "Global-scope dependency" above). The fix direction is
   to remove the global-scope dependency (explicit module export / owned object),
   but the migration path for cross-frame consumers (`top`/`parent` reach) and
   legacy in-browser global access is not yet designed.

## Verification (normative, once ratified)

- [ ] A coordinated release of `2.7.0` publishes `qcobjects`, `qcobjects-sdk`,
      and `qcobjects-cli` all at `2.7.0` within one release window.
- [ ] A canary app pins the chain version and installs with **exactly one**
      `qcobjects` and one `qcobjects-sdk` in `npm ls` (no duplicated tree),
      asserted in a release gate.
- [ ] An add-on declaring core/SDK as exact-pin peers resolves to the app's
      single instance (no shadow-copy) across the npm-version matrix CI runs.
- [ ] `--legacy-peer-deps` is **not** required for a clean install of the
      pinned chain (it MAY remain tolerated as an option, but correctness no
      longer depends on it).
- [ ] The `global` context is no longer a synthesized property on the ambient
      object: `global` resolves identically across Node, browser DevTools, and
      the built strict-mode bundle (no `Cannot set/redefine property: global`).