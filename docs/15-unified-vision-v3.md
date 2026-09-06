# 15 — Unified Vision v3.0: The Sovereign Polyglot Ecosystem

## Purpose

Declare where QCObjects is going and the transitional phases to get there.
This spec governs direction: when it conflicts with specs 01–14, this spec
wins and the affected spec MUST be updated in the same PR.

Sources: the three roadmap drafts in `roadmap/` (base vision, CosmosDB-lib
structural note, consolidated update), unified here as the single normative text.

## Vision

The transition to v3.0 is not a version bump. It formalizes QCObjects as
**polyglot, sovereign infrastructure**: evolving from a web framework into a
universal execution platform that orchestrates diverse runtimes,
native-performance handlers, and cloud-native integrations — managed from a
single decoupled CLI, with zero vendor lock-in and local-first AI readiness.

## Phase 1 — Structural Sovereignty (v3.0)

- License migration: full LGPLv3 → MIT transition across the ecosystem
  (mechanism: [09-license](./09-license.md)).
- Unified pipeline: decommission version-specific branches; single
  `development` flow with tag-based release channels (`latest`, `lts`, `beta`)
  per [08-ci-conventions](./08-ci-conventions.md).
- Metadata-driven ecosystem: `config.json` as universal source of truth;
  `$ENV()` / `$config()` injection standardized across core, SDK, CLI, and
  every handler (proven pattern: CosmosDB connector, PHP integration).
- Rollout order: CLI re-architecture first (developer entry point) → core +
  SDK refactor → cascade to `qcobjects-commands`, plugins, handlers, web
  components, `create-qcobjects` templates, eslint configs.

## Phase 2 — Server Abstraction Layer / High-Performance Engine (v3.1)

- Interface-driven server: decouple the CLI from the Node.js `http2` module
  via interchangeable **Server Adapters**.
- Launch `qcobjects-server-bun` alongside `qcobjects-server-node`; engine
  switches by configuration flag for sub-millisecond startup and massive
  concurrency gains.
- Zero-bloat core: core owns MVC routing + microservice orchestration only;
  network transport belongs to the chosen adapter.
- Config service evolution: `$ENV()`/`$config()` patterns extended to complex
  dependency injection across all plugins.

## Phase 3 — Polyglot Execution Layer (v3.2+)

On the dynamically loaded, npm-based handler architecture, support external
runtimes as native microservices, orchestrated by the CLI through Runtime Bridges:

- Legacy & bridge validation (`qcobjects-handler-php`): formalize PHP
  interoperability for legacy monolith migration.
- Performance edge (`qcobjects-handler-wasm`): WebAssembly modules compiled
  from Rust, AssemblyScript, or Go for CPU-bound tasks, cryptography, and
  secure edge execution.
- AI & data intelligence bridge (`qcobjects-handler-fastapi`): Python FastAPI
  persistent sidecars integrating PyTorch, LangChain, HuggingFace while web
  and routing stay in QCObjects.
- Sovereign AI: host local-first LLM inference engines and high-speed data
  processors on this layer (Code-Inference, WideLlama/OpenClaw-class tools,
  `qcobjects-enterprise`, `collab-server` for private networks).

## Architectural invariants (all phases)

- Decoupled handlers and adapters: independent npm packages, no hard
  dependencies, keyword-based auto-discovery.
- `config.json` remains the source of truth for runtime behavior,
  credentials, and routing.
- Best tool per domain: JS/TS for web, Rust/Wasm for speed, Python for data
  intelligence — one unified deployment pipeline.

## End of Life (v2.x)

All v2.x lines (`v2.3`, `v2.4-beta`, `v2.4-ts`, `v2.5-beta`) are designated
EOL and preserved as archive tags only: no public security patches, features,
or community support. Migration to the v3.0 track is strongly recommended.
Extended support and guided migration are available via enterprise SLA.

## Planning standard

Future plans MUST cite the phase (P1/P2/P3) they belong to, list the specs
they touch, and state acceptance per the Verification sections of those specs.
A plan that changes direction MUST update this spec first.

## Verification

- P1 exit: every repo MIT-licensed, single-branch pipeline live, templates
  re-verified on the v3.0 line.
- P2 exit: Bun adapter serves the reference app from a config flag flip.
- P3 exit: PHP + Wasm + FastAPI demo routes all green on one CLI-orchestrated app.
