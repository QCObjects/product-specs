# 10 — Contribution Pipeline

## Purpose

Define how a change travels from idea to every affected repo and npm package.

Sources: CLI README §§ versioning workflow, repo AGENTS.md command tables.
Repo-local PR mechanics: [08-ci-conventions](./08-ci-conventions.md).

## Scope

Multi-repo flow across core → SDK → CLI → templates → handlers → apps.

## The chain (normative)

- Every change MUST start from the spec: if no spec covers it, the PR MUST
  update (or add) the spec in this repo first, then reference it
  (`spec: docs/NN-name.md`) from the code PR.
- Propagation order MUST be: core → SDK → CLI → templates/handlers → apps.
  A downstream repo MUST NOT adopt an API its upstream has not tagged.
- Cross-repo PRs MUST link each other (upstream tag ↔ downstream bump) and
  MUST state the rollback tag if the chain breaks.
- Version bumps use the CLI itself, never manual `package.json` edits:

```shell
qcobjects v-patch --git --npm -m "fix: resolve timeout issue"  # patch + publish path
qcobjects v-minor --git -m "feat: add collaboration endpoints"
qcobjects v-sync -m "sync after release merge"                  # reconcile from git describe
qcobjects v-changelog > CHANGELOG.md                            # release notes from annotated tags
```

- `CHANGELOG.md` in each touched repo MUST gain an entry in the same PR
  (generate the body with `v-changelog`, curate before committing).
- Local dev loop per repo (binding): `npm i --legacy-peer-deps` → `npm run lint` →
  `npm test` (lint + jasmine) → `npm run build` (types → code → browser).
- When [15-unified-vision-v3](./15-unified-vision-v3.md) conflicts with an older
  spec, the PR MUST update the older spec in the same chain — never ship code
  against a stale spec.
- **Periodic accuracy audit (quarterly):** link-check and builds catch rot, not
  drift — every quarter, re-verify each spec's normative claims against the
  pinned source tags (class/param/flag/default level, as in the inaugural
  audit that fixed ~60 items). Audit findings land as a `fix/accuracy-audit`
  PR; verified code bugs found this way are filed upstream with file:line
  evidence instead of being silently spec'd around.

## Verification

- For any merged feature, `git log` shows: spec PR + upstream tag + downstream
  bump + changelog entries, in order.
- `v-changelog` output covers every annotated tag since the last minor.
