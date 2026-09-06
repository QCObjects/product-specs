# 07 — App-Templates Approach

## Purpose

Specify how app templates (`qcobjects-new-app`, CLI `templates/pwa|apps`)
deliver the out-of-box experience.

## Scope

Template repos and the CLI scaffolding that stamps them out. PWA runtime
semantics belong to 06; visual SDK widgets belong to 04.

## Normative

- Templates MUST be runnable immediately after `npm install`:
  `npm start` (cert + serve) with no extra configuration.
- Templates MUST demonstrate, at minimum: boot (`init`), one data-bound
  component, one backend route call, offline shell (service worker), and
  one form with validation.
- Template `config.json` MUST use `$ENV(...)` for every secret/host and
  `$config(...)` for derived paths; a committed `.env.example` MUST document
  each variable.
- CLI `src/templates/pwa` is the minimal shell; `src/templates/apps` are
  fuller starters. Both MUST track the [06-app-structure](./06-app-structure.md)
  layout; any layout change MUST update templates in the same release.
- Templates MUST pin `qcobjects` + `qcobjects-sdk` to a tested minor range
  and MUST be re-verified (install + serve + build) on every core/SDK minor bump.
- `qcobjects-new-app` doubles as the integration testbed: demo routes under
  `demo-tests/` MAY exist there but MUST NOT leak into `src/templates`.

## Verification

- CI stamps each template into a temp dir, runs install + serve smoke test +
  build, and all three succeed.
- `npm audit` on a freshly stamped template shows zero critical vulnerabilities.
