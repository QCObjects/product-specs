# 08 — CI Conventions

## Purpose

Pin how every repo in the ecosystem builds, tests, releases, and protects branches.

## Scope

Applies to `qcobjects`, `qcobjects-sdk`, `qcobjects-cli`, `qcobjects-new-app`,
templates, handlers, and this specs repo. v3.0 pipeline direction is normative
in [15-unified-vision-v3](./15-unified-vision-v3.md); this spec pins the mechanism.

## Normative

- Branching MUST be: single `development` branch for integration, `main` as
  release digest. Version-specific branches (e.g. `v2.3`) MUST NOT be created;
  old tracks survive only as archive tags.
- Topic branches MUST be named `feature/*`, `fix/*`, or `bugfix/*` from
  `development`. Direct commits to `main`/`development` are forbidden except
  for empty-repo bootstrap. Never rebase.
- Releases MUST be tag-driven: push of `v*.*.*` triggers build → test → npm
  publish. Tag suffixes select the npm dist-tag: `-beta` → `beta`,
  `-lts` → `lts`, otherwise `latest`.
- CI on every repo MUST run, at minimum: `npm run lint` (eslint, zero warnings),
  `npm test` (jasmine suite green), `npm run build` (all three distributions).
  Publish jobs MUST rebuild from the tag ref, never reuse PR artifacts.
- PRs to `main` MUST originate from `development` (enforced by workflow).
  PRs to `development` SHOULD reference the spec file(s) they affect.
- Protected branches MUST require the CI checks green before merge.

## Verification

- Pushing a `v9.9.9-beta` test tag on a fork publishes only under `beta`.
- Branch protection shows required checks: lint, test, build.
