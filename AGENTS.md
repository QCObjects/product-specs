# product-specs — AGENTS.md

**Docs repo, not code.** Structured product specifications for the QCObjects ecosystem.
This repo is the source of truth for framework, SDK, CLI, app structure, and the v3.0 roadmap.

## Structure

- `docs/` — one Markdown file per product foundation (nav in `mkdocs.yml`)
  + `docs/index.md` (site home), `docs/requirements.txt` (MkDocs deps)
- `schemas/` — JSON Schemas + `schemas/examples/*.json` fixtures (CI-validated)
- `diagrams/` — Mermaid sources (`.mmd`) + rendered SVGs
- `mkdocs.yml` — MkDocs Material site config (`strict: true`)
- `.readthedocs.yaml` — RTD indexing config (MkDocs backend)
- No build, no dependencies, no runtime code

## Conventions

- Every doc has: Purpose, Scope, Normative statements (`MUST`/`SHOULD`/`MAY`), Verification
- Normative language follows RFC 2119
- Definitions live inline in specs; code links to QCObjects code repos MUST be
  pinned to release tags (e.g. `.../blob/v2.5.142/src/...`), never floating
  `development` links. Self-repo living refs (this repo's own files) MAY track
  `development`.
- Diagrams as Mermaid code blocks (renderable on GitHub); keep `.mmd` source in `diagrams/`
- Cross-link docs relatively (`./architecture.md`, not absolute URLs)
- English for all specs; keep filenames kebab-case

## Git workflow

Topic branches from `development` (`feature/<topic>`), PR into `development`,
promotion `development` → `main` via release PR. Never rebase. SSH only.
Pages deploys from `main`; RTD tracks the default branch.

## Verification

```bash
npx --yes markdown-link-check docs/*.md README.md AGENTS.md
for f in schemas/examples/*.json; do npx --yes ajv-cli validate -s schemas/config.schema.json -d "$f" --strict=false; done
pip install -r docs/requirements.txt && mkdocs build --strict
mkdocs serve   # local preview at http://127.0.0.1:8000
```
