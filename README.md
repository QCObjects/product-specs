# QCObjects Product Specs

[![docs](https://github.com/QCObjects/product-specs/actions/workflows/docs.yml/badge.svg)](https://github.com/QCObjects/product-specs/actions/workflows/docs.yml)
[![pages](https://github.com/QCObjects/product-specs/actions/workflows/pages.yml/badge.svg)](https://github.com/QCObjects/product-specs/actions/workflows/pages.yml)
<!-- TODO provisioning: point RTD badge at the project readthedocs URL after RTD import -->
[![Documentation Status](https://readthedocs.org/projects/qcobjects-product-specs/badge/?version=latest)](https://readthedocs.org/dashboard/)

**Source of truth** for the QCObjects ecosystem: framework, SDK, CLI, resulting app
structure, templates, conventions, and the unified v3.0 vision.

**Published docs:** [GitHub Pages](https://qcobjects.github.io/product-specs/) (from `main`)
· local preview via `mkdocs serve` · Read the Docs (live after importing
`qcobjects-product-specs` at readthedocs.org with `.readthedocs.yaml`).

One document per product foundation. Each spec is normative (RFC 2119 `MUST` /
`SHOULD` / `MAY`) with an explicit Verification section so future plans can be
checked against it. Definitions live inline; code references are pinned to
release tags (`v2.5.142` core, `v2.5.105` SDK, `v2.5.158` CLI, `v2.4.40-ts`
template) because code drifts and the spec is the truth.

## Index

| # | Spec | What it pins down |
|---|------|-------------------|
| 01 | [Product Vision](docs/01-product-vision.md) | Sovereign, polyglot, local-first platform vision |
| 02 | [Architecture](docs/02-architecture.md) | MVC + microservice + handler + adapter layers |
| 03 | [Core Framework](docs/03-core-framework.md) | `QCObjects` repo: classes, packages, exports |
| 04 | [SDK](docs/04-sdk.md) | `qcobjects-sdk`: controllers, views, components |
| 05 | [CLI](docs/05-cli.md) | `qcobjects-cli`: commands, servers, scaffolding |
| 06 | [App Structure](docs/06-app-structure.md) | Resulting app layout produced by the CLI |
| 07 | [App Templates](docs/07-app-templates.md) | `qcobjects-new-app` approach, PWA shell |
| 08 | [CI Conventions](docs/08-ci-conventions.md) | Branching, tags, release channels, workflows |
| 09 | [License](docs/09-license.md) | LGPLv3 → MIT migration path |
| 10 | [Contribution Pipeline](docs/10-contribution-pipeline.md) | How changes flow across the multi-repo ecosystem |
| 11 | [Features](docs/11-features.md) | Feature catalog with status per repo |
| 12 | [Schemas](docs/12-schemas.md) | `config.json`, `$ENV()`/`$config()` conventions (+ `schemas/`) |
| 13 | [Diagrams](docs/13-diagrams.md) | System maps (Mermaid sources in `diagrams/`) |
| 14 | [Build Scripts Blueprint](docs/14-build-scripts-blueprint.md) | esbuild/parcel/tsc pipelines per repo |
| 15 | [Unified Vision v3.0](docs/15-unified-vision-v3.md) | Sovereign Polyglot Ecosystem + transitional phases |

> Specs 01–14 describe the **current** framework. Spec 15 describes **where it is going**
> and how to plan the transition. When they conflict, 15 governs the direction and
> the affected 01–14 spec MUST be updated in the same PR.
