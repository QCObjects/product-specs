# 01 — Product Vision

## Purpose

Define what QCObjects is, who it serves, and the principles every future
decision MUST be judged against. This is the top of the authority chain:
when specs conflict, the vision wins.

## Scope

Covers the product identity across `qcobjects` (core), `qcobjects-sdk`,
`qcobjects-cli`, and app templates. Excludes release scheduling (see
[15-unified-vision-v3.md](./15-unified-vision-v3.md)).

## Normative

- QCObjects MUST remain a full-stack framework for micro-services and
  micro-frontends in an N-Tier architecture — not a single-layer UI library.
- The platform MUST preserve three sovereign guarantees:
  1. **No vendor lock-in** — apps run on own infrastructure with own config.
  2. **Local-first execution** — routing, rendering, and orchestration work
     without calling vendor clouds.
  3. **Metadata-driven behavior** — `config.json` is the source of truth;
     code MUST NOT hardcode credentials, paths, or routing.
- Secrets and environment values MUST be injected via `$ENV(VAR)` /
  `$config(key)` placeholders, never committed literally.
- Developer experience MUST stay "one CLI in, running app out":
  scaffold → serve (HTTP/2) → build → publish, all through `qcobjects-cli`.
- Every new capability SHOULD ship as a decoupled, keyword-discovered package
  (handler, server adapter, component set) rather than a core dependency.

## Verification

- Any feature proposal cites which guarantee it strengthens.
- `grep -r "process.env" src/` in core repos returns no direct credential reads
  outside the `$ENV()` resolver.
