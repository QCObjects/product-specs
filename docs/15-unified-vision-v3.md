# 15 — Unified Vision v3.0: The Sovereign Polyglot Ecosystem

## Purpose

Declare where QCObjects is going and the transitional phases to get there —
with per-repo exit criteria and migration checklists, so standard plans can be
derived mechanically.
This spec governs direction: when it conflicts with specs 01–14, this spec
wins and the affected spec MUST be updated in the same PR.

Sources: the three roadmap drafts in `roadmap/` (base vision, CosmosDB-lib
structural note, consolidated update), systematized here in full as the single
normative text. The drafts may be archived; this file is the record.

## Vision

The transition to v3.0 is not a version bump. It formalizes QCObjects as
**polyglot, sovereign infrastructure**: evolving from a web framework into a
universal execution platform that orchestrates diverse runtimes,
native-performance handlers, and cloud-native integrations — managed from a
single decoupled CLI, with zero vendor lock-in and local-first AI readiness.

Why it protects current investment: handler/adapter decoupling means existing
`config.json` shapes, `$ENV()`/`$config()` usage, and component code keep
working while engines and runtimes swap underneath (proven by the CosmosDB
connector pattern: same config shape, cloud-native backend).

## Phase 1 — Structural Sovereignty (v3.0)

Goal: remove legal and pipeline friction for enterprise adoption.

- **License migration:** full LGPLv3 → MIT across the ecosystem
  (mechanism: [09-license](./09-license.md)). Per-repo checklist: `LICENSE.txt`
  swap → `package.json` field → source headers → `CHANGELOG.md` → header-lint green.
- **Unified pipeline:** decommission version-specific branches; single
  `development` flow with tag-based channels (`latest`, `lts`, `beta`) per
  [08-ci-conventions](./08-ci-conventions.md). Archive old tracks as tags.
- **Metadata-driven ecosystem:** `config.json` as universal source of truth;
  `$ENV()` / `$config()` injection standardized across core, SDK, CLI, and
  every handler.
- **Rollout order (binding):**
  1. `qcobjects-cli` re-architecture first (developer entry point).
  2. Core (`qcobjects`) + `qcobjects-sdk` refactor + MIT.
  3. Cascade: `qcobjects-commands`, plugins, handlers, web components,
     `create-qcobjects` templates, `eslint-config-qcobjects` (+ TypeScript twin).
- **P1 exit criteria:** every repo MIT-licensed with header-lint green;
  single-branch pipeline live with tag-channel publish proven (`latest` + one
  `-beta`); templates re-verified (install + serve + build) on the v3.0 line;
  this spec 15 + specs 08/09/10 updated to past tense.

## Phase 2 — Server Abstraction Layer / High-Performance Engine (v3.1)

Goal: runtime-agnostic transport with sub-millisecond startup and massive
concurrency gains.

- **Interface-driven server:** decouple the CLI from Node.js's native `http2`
  module via interchangeable **Server Adapters** implementing one contract:
  `listen(config)` / `route(request)` → `backend.routes` dispatch /
  `static(request)` fallback / `close()`. Core owns MVC routing + microservice
  orchestration only; transport belongs to the adapter (zero-bloat core).
- **Launch `qcobjects-server-bun`** alongside `qcobjects-server-node`;
  engine switches by configuration flag (e.g. `"serverAdapter": "bun"|"node"`).
- **Config service evolution:** `$ENV()`/`$config()` patterns extended to
  complex dependency injection across all plugins.
- **P2 exit criteria:** reference app serves identically on both adapters from
  a config-flag flip; adapter contract spec added to [02-architecture](./02-architecture.md);
  performance delta (startup p50, concurrent connections) published in the release notes.

## Phase 3 — Polyglot Execution Layer (v3.2+)

Goal: external runtimes as native microservices, orchestrated by the CLI
through Runtime Bridges (all npm-installed, keyword-discovered, per spec 02).

| Bridge | Package | Workload |
|---|---|---|
| Legacy validation | `qcobjects-handler-php` | Legacy monolith migration interop |
| Performance edge | `qcobjects-handler-wasm` | CPU-bound, crypto, secure edge (Rust/AssemblyScript/Go → Wasm) |
| AI & data intelligence | `qcobjects-handler-fastapi` | Persistent Python sidecars (PyTorch, LangChain, HuggingFace) |

- **Sovereign AI:** host local-first LLM inference + high-speed data processors
  on this layer (Code-Inference, WideLlama/OpenClaw-class tools,
  `qcobjects-enterprise`, `collab-server` for private networks).
- Granular stream/timeout/resource control at the server level keeps stability
  during heavy inference or polyglot workloads.
- **P3 exit criteria:** PHP + Wasm + FastAPI demo routes green on one
  CLI-orchestrated app; handler authoring guide published; [11-features](./11-features.md)
  rows flipped to `stable`.

## Architectural invariants (all phases, normative)

- Decoupled handlers and adapters: independent npm packages, no hard
  dependencies, keyword-based auto-discovery.
- `config.json` remains the source of truth for runtime behavior,
  credentials, and routing.
- Best tool per domain: JS/TS for web, Rust/Wasm for speed, Python for data
  intelligence — one unified deployment pipeline.

## End of Life (v2.x, normative)

| Line | State | Support |
|---|---|---|
| `v2.3.x`, `v2.4.x-beta`, `v2.4.x-ts`, `v2.5.x-beta` lines (dotted versions, e.g. `v2.3.1`, `v2.4.1-beta`) | EOL, archive tags only (CLI also carries `archive/v2.4-beta`, `archive/v2.4-ts` prefixed tags) | No patches, features, or community support |
| v3.0 track | Current | Full maintenance |
| Legacy enterprise | SLA only | Custom patching / guided migration via consulting contracts |

Migration to the v3.0 track is strongly recommended (MIT + unified pipeline +
adapters). Open-source sustainability: Open Collective / GitHub Sponsors.

## Planning standard (normative)

Future plans MUST cite the phase (P1/P2/P3), list touched specs, state
acceptance per those specs' Verification sections, and name the exit criteria
above they advance. A plan that changes direction MUST update this spec first.

## Verification

- P1 exit: all criteria in Phase 1 list green.
- P2 exit: Bun adapter serves the reference app from a config flag flip.
- P3 exit: PHP + Wasm + FastAPI demo routes all green on one CLI-orchestrated app.
