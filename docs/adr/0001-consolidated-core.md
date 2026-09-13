# ADR-0001 — Consolidate Core, SDK, and CLI into a single `qcobjects` package

- **Status:** Proposed
- **Date:** 2026-09-13
- **Supersedes:** the package-decoupling mandate in
  [15-unified-vision-v3](../15-unified-vision-v3.md) § "Architectural
  invariants" (relaxed for framework packages only, NOT for add-ons).
- **Affected specs:** 02, 03, 04, 05, 15, 16, 17.

## Context

QCObjects is currently split across three framework packages — `qcobjects`
(core), `qcobjects-sdk`, and `qcobjects-cli` — each versioned and published
independently. This produced a concrete failure: the `qcobjects-new-app`
template pins `dependencies.qcobjects: ">=2.5.142 <3.0.0"`, which resolved to
`2.5.142`, a build that still carries an eager top-level `require("node:process")`.
The browser bundle fix shipped in core `2.6.2` never reached the template, so
the template serves a blank page (`Dynamic require of "node:process" is not
supported`).

The root cause is not a single stale range. It is that **the framework's three
layers version and publish independently, so a fix in one layer can only reach
a consumer if every intermediate dependency resolves to the matching release.**
That coupling is *core dependency pollution*: drift between core, SDK, and CLI
versions, each with its own peer range matrix, each a moving target.

## Decision

**Consolidate `qcobjects` (core), `qcobjects-sdk`, and `qcobjects-cli` into one
repository (`QCObjects`) emitting a single npm package (`qcobjects`).**

- The consolidated code lives in one `src/` tree and ships as
  `qcobjects@2.6.5-unified` — a **patch** bump, not `3.0`.
- `qcobjects-sdk` and `qcobjects-cli` npm packages are deprecated and archived;
  no further publishes occur from either.
- Git history is preserved (merge/transfer, not a squash copy).
- The public API (component classes, `config.json` shape, `$ENV()`/`$config()`
  injection, CLI commands) is preserved for existing consumers.

## Consequences

### Positive

- One package, one version, one publish: a core fix propagates to every
  consumer the moment they resolve a single `qcobjects` range. The template
  failure above becomes structurally impossible.
- No peer-range matrix across core/SDK/CLI; no inter-package drift.
- Simpler install surface for apps and add-ons (one `qcobjects` dependency).
- SDK and CLI code, colocated with core, can share internals without a
  publish round-trip.

### Negative / risks

- Single larger package; consumers who depended on `qcobjects-sdk` or
  `qcobjects-cli` alone must migrate their dependency declaration.
- Deprecation must be communicated clearly and mechanically (README pointers,
  `npm deprecate`, archive notices).
- Larger blast radius on any one publish — offsets require the stricter
  release gate already proven in the v2.5.6-ts / v2.6.2 work.

### Neutral / non-goals

- Add-on packages (`qcobjects-handler-*`, `qcobjects-lib-*`,
  `qcobjects-command-*`, `qcobjects-admin-*`) remain **decoupled** and
  keyword-discovered. This decision does NOT collapse the add-on ecosystem.

## Alternatives considered

1. **Keep three packages, tighten the range matrix** (pin exact versions).
   Rejected: does not fix the structural drift; exact pinning makes the matrix
   brittle and unsolvable in the general case.
2. **npm/pnpm workspaces monorepo with three packages.**
   Rejected for now: preserves the three-surface install and the drift; adds
   workspace tooling cost without eliminating the version coupling.
3. **Single repo, single package (this ADR).**
   Accepted: directly eliminates the coupling and matches "all contained into
   one QCObjects repo."

## Decision record

- The **normative** consolidation contract lives in
  [17-consolidated-core](../17-consolidated-core.md).
- The architecture diagram (layer model) is updated in
  [02-architecture](../02-architecture.md) to reflect the merged framework
  package and to reference this ADR.

## Verification

- [ ] `qcobjects@2.6.5-unified` publishes from `QCObjects` only.
- [ ] `qcobjects-sdk` and `qcobjects-cli` are `deprecated`/`archived` on npm
      with README pointers to `QCObjects`.
- [ ] Spec 15, 02, 04, 05 updated in the same planning PR to no longer describe
      SDK/CLI as independently published framework packages.