# product-specs — AGENTS.md

**Docs repo, not code.** Structured product specifications for the QCObjects ecosystem.
This repo is the source of truth for framework, SDK, CLI, app structure, and the v3.0 roadmap.

## Structure

- `docs/` — one Markdown file per product foundation (see `docs/README.md` index)
- `schemas/` — JSON Schemas for config files (`config.json`, `package.json` conventions)
- `diagrams/` — Mermaid sources (`.mmd`) + rendered SVGs
- No build, no dependencies, no runtime code

## Conventions

- Every doc has: Purpose, Scope, Normative statements (`MUST`/`SHOULD`/`MAY`), Verification
- Normative language follows RFC 2119
- Diagrams as Mermaid code blocks (renderable on GitHub); keep `.mmd` source in `diagrams/`
- Cross-link docs relatively (`./architecture.md`, not absolute URLs)
- English for all specs; keep filenames kebab-case

## Git workflow

Topic branches from `development` (`feature/<topic>`), PR into `development`,
promotion `development` → `main` via release PR. Never rebase. SSH only.

## Verification

- Markdown link check in CI (`docs/**/*.md` must have no broken relative links)
- After editing, run: `npx --yes markdown-link-check docs/*.md` (or open the CI run)
