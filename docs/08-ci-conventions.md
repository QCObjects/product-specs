# 08 — CI Conventions

## Purpose

Pin how every repo in the ecosystem builds, tests, releases, and protects branches.

Sources: `npmpublish.yml` (core/CLI), CLI README §§ CI Pipeline Considerations,
repo AGENTS.md CI notes. v3.0 direction: [15-unified-vision-v3](./15-unified-vision-v3.md).

## Scope

Applies to `qcobjects`, `qcobjects-sdk`, `qcobjects-cli`, `qcobjects-new-app`,
templates, handlers, and this specs repo.

## Branching & tags (normative)

- Single `development` integration branch; `main` is the release digest.
  Version-specific branches (e.g. `v2.3`) MUST NOT be created; old tracks survive
  only as archive tags.
- Topic branches `feature/*`, `fix/*`, `bugfix/*` from `development`. No direct
  commits to `main`/`development` except empty-repo bootstrap. Never rebase.
- Releases are tag-driven: push of `v*.*.*` triggers build → test → npm publish.
  Suffix selects npm dist-tag: `-beta` → `beta`, `-lts` → `lts`, else `latest`.

## Publish pipeline (normative, `npmpublish.yml` as observed)

```yaml
# jobs: build (checkout tag ref, node 22, npm-upgrade step, npm i --legacy-peer-deps,
#         [new-app only: npm run build], npm test)
# then publish-npm (needs build; OIDC id-token:write; rebuild; suffix routing)
if [[ ref == *-beta ]]; then npm publish --tag beta
elif [[ ref == *-lts ]]; then npm publish --tag lts
else npm publish; fi
```
NOTE: the CLI `build` job runs NO build step (install + test only); only the
template workflow builds between install and test. Both include an
`npm install -g npm@latest` OIDC-support step first.

- Publish jobs MUST rebuild from the tag ref, never reuse PR artifacts; auth
  MUST be OIDC (`id-token: write`), never long-lived tokens.
- `ci.yml` / `codeql-analysis.yml` in some repos still carry placeholder TODO
  steps and are NOT runnable — repos MUST either implement or delete them;
  the `docs` + `publish` + branch-protection jobs are the binding ones.
- CI on every repo SHOULD run, at minimum: `npm run lint` (eslint, zero warnings),
  `npm test` (jasmine green), `npm run build` (all three distributions) — TARGET
  STATE: `ci.yml` is still a placeholder echo in the CLI repo, so today only
  `npmpublish.yml` (+ `main-pr-source.yml`) binds. Wire the checks before
  strengthening this to MUST.
- PRs to `main` MUST originate from `development` (workflow-enforced). PRs to
  `development` SHOULD reference affected spec files. Protected branches MUST
  require CI green before merge.
- **Containerized tests:** repos MAY run the suite in Docker via a compose
  `sut` service (`build: .`, `command: npm test` — reference: web-dev site
  repo). Compose-run and runner-run results MUST agree; on disagreement the
  compose run governs (it matches production).

## Version-tag interplay (normative)

- Tags are cut by the CLI itself (`v-patch --git --npm`); the pushed tag is what
  CI publishes — so a bad tag publishes a bad release. `v-sync` reconciles
  `VERSION` from `git describe` after release merges.
- GitHub Actions repos SHOULD use `"postversion": "git push"` (branch only).
  KNOWN VIOLATION: the canonical template still ships
  `"postversion": "git push && git push --tags"` despite having tag-triggered
  publish — fix the template or accept duplicate trigger runs.

## Verification

- Pushing a `v9.9.9-beta` test tag on a fork publishes only under `beta`.
- Branch protection shows required checks: lint, test, build (+ docs/schemas/mkdocs here).
