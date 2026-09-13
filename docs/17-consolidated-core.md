# 17 — Consolidated Core (Core + SDK + CLI in one `qcobjects` package)

## Purpose

Declare the structural end-state of the framework cursor: the three framework
packages (`qcobjects`, `qcobjects-sdk`, `qcobjects-cli`) become one repository
(`QCObjects`) publishing one npm package (``qcobjects``), on the path to v3.0.

This is the **normative** companion to
[ADR-0001](./adr/0001-consolidated-core.md); the ADR records *why* (the
decision), this spec records *what* (the contract). Both were introduced in the
same planning PR.

## Scope

The three framework packages and their consolidation into `qcobjects`.
Add-on packages (handlers, libs, commands, admin) stay decoupled — see
[16-addons](./16-addons.md) — and are explicitly **out of scope** for this merge.

## Status

Transitional. Version `2.6.5-unified` is a **patch** release on the v2.x line,
**not** v3.0. The v3.0 track described in
[15-unified-vision-v3](./15-unified-vision-v3.md) is unchanged in its end-state;
this spec only changes *how the framework packages are packaged* on the way there.

## Consolidation contract (normative)

### Repository and package shape

- Core, SDK, and CLI source MUST live in one `QCObjects` repository under a
  single `src/` tree.
- The repository MUST publish exactly ONE npm package: `qcobjects`.
- The consolidated release line MUST be `qcobjects@2.6.5-unified`. Before this,
  `2.6.2` (core), `2.6.0` (SDK), and `2.6.2` (CLI) were the last independently
  published lines; after this, all three are superseded by the single package.

### Deprecation of SDK and CLI packages

- `qcobjects-sdk` and `qcobjects-cli` MUST be deprecated on npm and archived
  (archive/tag-only) in their repositories.
- Each archived repo MUST retain its history and a README pointer to
  `QCObjects`, stating the content now ships in `qcobjects`.
- No new publishes MUST occur from the SDK or CLI repositories after the
  consolidation cut-over.

### Version and publish discipline

- The consolidation MUST NOT publish to npm until it is complete and verified
  (see Verification below). Do not dispatch `npmpublish` early.
- No `3.0` release is implied or triggered by this change.
- SSH-only auth; never push to the read-only `QuickCorp` org.

### API compatibility

- Existing public surfaces MUST keep working: component classes (``Component``,
  ``Controller``, ``View``, ``VO``, ``DDO``), packaging (``Package``,
  ``Import``/``Export``), routing, loaders, SDK widgets/controllers/effects,
  and CLI commands (`create`, `publish`, `generate-sw`, `launch`, etc.).
- `config.json` shape and `$ENV()`/`$config()` injection MUST remain the source
  of truth for runtime behavior.

### Browser-safety (the driving bug)

A consolidated package MUST eliminate core dependency pollution, concretely:

- Every published distribution (ESM, CJS, IIFE browser bundle) MUST contain
  **zero eager top-level `node:*` / `types` imports** (the `2.6.2` lazification
  work, now guaranteed to reach consumers because there is only one package).
- The template and reference app MUST serve/bundle green against the single
  `qcobjects` package with no `Dynamic require of "node:*" is not supported`.

## Relationship to the v3.0 roadmap (normative)

- The decoupling mandate in [15-unified-vision-v3](./15-unified-vision-v3.md)
  § "Architectural invariants" is **relaxed for the three framework packages**:
  core, SDK, and CLI consolidate into one. Decoupling remains **mandatory for
  add-on packages** (handlers, adapters, plugins, libs, commands), which MUST
  stay independent npm packages with keyword auto-discovery.
- Phase 1 "rollout order" in spec 15 is updated accordingly: the step that
  treats core and SDK as separately refactored/released packages is replaced by
  the single-package consolidation (see spec 15 § Phase 1).

## Rollout order (binding)

1. **Plan** — this spec + ADR-0001 + spec 02/15 amendments committed to
   `product-specs` (this PR).
2. **Consolidate** — merge `qcobjects-sdk` and `qcobjects-cli` sources into the
   `QCObjects` repo `src/` (history-preserving), reconciling the `package.json`
   (`scripts`, `bin`, keywords, `exports`) into one manifest.
3. **Verify locally** — build + test + browser-bundle gate green (see Verification).
4. **Cut-over** — publish `qcobjects@2.6.5-unified`; then deprecate/archive
   `qcobjects-sdk` and `qcobjects-cli`.
5. **Cascade** — point `qcobjects-new-app` (and other consumers) at the single
   `qcobjects` package; re-verify install + serve + build.

## Verification

- [ ] `npm run build` rc=0 and `npm test` green in the consolidated package.
- [ ] ESM/CJS/IIFE dists contain 0 eager top-level `node:*`/`types` imports
      (`grep` proof on the emitted files).
- [ ] `npm view qcobjects` shows `2.6.5-unified`; `npm view qcobjects-sdk` and
      `npm view qcobjects-cli` show deprecated/archived.
- [ ] Template + reference app serve/browser-bundle green with a single
      tag-pinned `qcobjects` dependency.
- [ ] Spec 02 and 15 updated in the same PR and no longer describe SDK/CLI as
      independently published framework packages.