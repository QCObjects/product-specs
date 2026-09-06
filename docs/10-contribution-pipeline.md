# 10 — Contribution Pipeline

## Purpose

Define how a change travels from idea to every affected repo and npm package.

## Scope

Multi-repo flow across core → SDK → CLI → templates → handlers → apps.
Repo-local PR mechanics defer to [08-ci-conventions](./08-ci-conventions.md).

## Normative

- Every change MUST start from the spec: if no spec covers it, the PR MUST
  update (or add) the spec in this repo first, then reference it
  (`spec: docs/NN-name.md`) from the code PR.
- Propagation order MUST be: core → SDK → CLI → templates/handlers → apps.
  A downstream repo MUST NOT adopt an API its upstream has not tagged.
- Cross-repo PRs MUST link each other (upstream tag ↔ downstream bump) and
  MUST state the rollback tag if the chain breaks.
- Version bumps use the CLI itself (`qcobjects v-patch|v-minor|v-major`);
  manual `package.json` version edits MUST NOT be committed.
- `CHANGELOG.md` in each touched repo MUST gain an entry in the same PR.
- When [15-unified-vision-v3](./15-unified-vision-v3.md) conflicts with an
  older spec, the PR MUST update the older spec in the same chain — never
  ship code against a stale spec.

## Verification

- For any merged feature, `git log` shows: spec PR + upstream tag + downstream
  bump + changelog entries, in order.
