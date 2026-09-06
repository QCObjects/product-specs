# 13 — Diagrams

## Purpose

Visual system maps. Sources live in `diagrams/*.mmd`; rendered views are the
Mermaid blocks below (GitHub renders them natively).

## Scope

Ecosystem map, request lifecycle, and release flow. Per-component internals
belong in code comments, not here.

## Ecosystem map

```mermaid
flowchart TB
    CLI[qcobjects-cli<br/>scaffold · serve · build · publish] --> APP[App<br/>PWA shell + packages]
    APP --> CORE[qcobjects core<br/>Class · MVC · routing · loaders]
    APP --> SDK[qcobjects-sdk<br/>components · controllers · views]
    SDK --> CORE
    CLI --> SRV[Server adapters<br/>node · bun -P2-]
    SRV --> H[Handlers / microservices<br/>static · php · wasm · fastapi]
    CFG(config.json<br/>$ENV / $config] -.-> CLI
    CFG -.-> SRV
    CFG -.-> H
```

Source: [`diagrams/ecosystem.mmd`](https://github.com/QCObjects/product-specs/blob/development/diagrams/ecosystem.mmd)

## Request lifecycle

```mermaid
sequenceDiagram
    participant B as Browser PWA
    participant S as Server adapter
    participant R as backend.routes
    participant M as Microservice/handler
    B->>S: HTTP/2 request
    S->>R: match path regex
    R->>M: dispatch by microservice name
    M-->>S: response + headers + cors
    S-->>B: response (cache per cacheControl)
```

## Release flow

```mermaid
flowchart LR
    F[feature/*] --> D[development]
    D --> M[main]
    M --> T[tag vX.Y.Z -suffix]
    T --> N[npm dist-tag<br/>latest · beta · lts]
```

## Component tree (Nested Components Stack)

```mermaid
flowchart TB
    G[global.componentsStack] --> C1[Component main<br/>cached · MainController]
    C1 --> T1[template main.tpl.html<br/>{{data}} bindings]
    C1 --> C2[Component grid<br/>GridController 2x2]
    C2 --> S1[subcomponent card xN<br/>DataGridController mapping]
    C1 --> C3[Component signup-form<br/>FormField · FormController]
    C3 --> SVC[SignupClientService<br/>JSONService POST]
    SVC --> MS[Microservice<br/>org.quickcorp.backend.signup]
```

## Normative

- Diagrams MUST match the specs: any layer/contract change MUST update the
  `.mmd` source and the block in this file in the same PR.
- New diagrams MUST be added as `.mmd` sources first, embedded second.

## Verification

- `npx --yes @mermaid-js/mermaid-cli -i diagrams/ecosystem.mmd` renders without errors.
