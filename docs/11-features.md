# 11 — Features

## Purpose

Catalogue user-visible features with their status per repo, so plans can see
what exists, what is partial, and what is roadmap.

## Scope

Features across core, SDK, CLI, templates. Status values: `stable`, `beta`,
`planned` (with target phase from spec 15), `eol`.

## Normative

| Feature | Core | SDK | CLI | Templates | Status |
|---|---|---|---|---|---|
| Class system + MVC primitives | ✅ | — | — | — | stable |
| Package/Import/Export + routing | ✅ | — | — | — | stable |
| Storage cache, i18n, Crypt | ✅ | — | — | — | stable |
| Grid/List/Slider/Splash/Notifications/Modal components | — | ✅ | — | demo | stable |
| Form validation, Swagger controllers | — | ✅ | — | demo | stable |
| Cloud-auth session (token) | — | ✅ | — | demo | beta |
| Scaffold (`create`), HTTP/HTTPS/HTTP2 + GAE servers | — | — | ✅ | — | stable |
| Collab server, shell, createcert | — | — | ✅ | — | stable |
| esbuild + tsc builds, publish-static | — | — | ✅ | ✅ | stable |
| PWA shell (manifest, SW, offline) | — | — | template | ✅ | stable |
| `$ENV()`/`$config()` injection | ✅ | ✅ | ✅ | ✅ | stable |
| PHP handler bridge | — | — | ✅ | — | beta |
| Deno support (`mod.ts`) | ✅ core | — | ✅ | — | beta |
| MIT license line | — | — | — | — | planned (P1) |
| Bun server adapter | — | — | — | — | planned (P2 SAL) |
| Wasm handler (Rust/AssemblyScript) | — | — | — | — | planned (P3) |
| FastAPI sidecar bridge | — | — | — | — | planned (P3) |
| Local-first LLM orchestration | — | — | — | — | planned (P3) |

- Any new feature MUST enter this table as `planned` with its phase before
  code lands; status MUST advance to `beta`/`stable` in the shipping PR.
- `eol` features MUST stay listed for one major line with a migration pointer.

## Verification

- Each `stable` cell has a passing test or demo route proving it.
- No shipped feature is missing from the table.
