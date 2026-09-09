# 16 — Add-ons (handlers, libs, commands, admin)

## Purpose

Catalogue the official add-on packages: what each does, which autoload keyword
it carries, and its lifecycle status — so apps discover backend capability
without forking the framework.

Sources: `package.json` keywords/versions read live from each repo
(2026-09-06); README purpose lines. Definitions below are authoritative;
per-release detail lives in each repo.

## Scope

Official `QCObjects/*` add-ons only. Community packages follow the same
keyword contract ([05-cli](./05-cli.md) § autoload) but are listed elsewhere.

## How add-ons load (normative)

- Every add-on is an independent npm package carrying its role in `package.json`
  `keywords`: `qcobjects-handler` (backend route handlers), `qcobjects-lib`
  (libraries), `qcobjects-command` (CLI commands), `qcobjects-admin-lib`
  (admin storage backends). The CLI autoloader picks them up per
  [05-cli](./05-cli.md) — no core changes needed to adopt one.
- Add-ons MUST follow the unified pipeline (single `development`, tag releases
  per [08-ci-conventions](./08-ci-conventions.md)) and MUST NOT pin conflicting
  `qcobjects` majors.

## Built-in vs Add-on (normative)

The following capabilities are **built into the framework** and always available
without installing any add-on package:

| Capability | Role | Notes |
|---|---|---|
| `qcobjects` core | library | Framework foundation: class system, components, routing, loaders, processors |
| `qcobjects-sdk` | library | Controllers, views, components, effects, cloud auth, i18n, models |
| `create`, `publish`, `generate-sw`, `launch`, `upgrade-to-enterprise` | CLI commands | Registered in `cli-main.ts`, no extra install |
| `com.qcobjects.backend.microservice.static` | handler | Serves `QCObjects.js`, `QCObjects-SDK.js`, `/qcobjects-sdk/*` with CORS `*`; injected by `defaultsettings.ts` when no routes exist |

**Everything else** (payment handlers, email/SMS libs, admin panels, custom commands, storage backends) MUST be adopted as add-on packages via the keyword autoload contract. The catalogue below lists only those add-on packages.

## Catalogue (normative)

| Add-on | Role / keywords | Purpose | Status |
|---|---|---|---|
| [qcobjects-handler-hello-world](https://github.com/QCObjects/qcobjects-handler-hello-world) | handler (`qcobjects-handler` + `qcobjects-api`; package `v1.0.0`, no git tags) | Minimal starter handler template — copy it to author a new handler | stable reference |
| [qcobjects-handler-webpayplus](https://github.com/QCObjects/qcobjects-handler-webpayplus) | handler | Transbank WebPay Plus flow (`/checkout/webpay/init`, `/checkout/webpay/result`) | stable |
| [qcobjects-handler-openapi](https://github.com/QCObjects/qcobjects-handler-openapi) | handler | Generic Open API request handler | stable |
| [qcobjects-handler-contactform](https://github.com/QCObjects/qcobjects-handler-contactform) | handler | Contact-form endpoint (`/rest/contactform`) → email + Mailchimp subscriber notification | stable |
| [qcobjects-handler-mockup](https://github.com/QCObjects/qcobjects-handler-mockup) | handler | Mock backend services for development/test | stable |
| [qcobjects-admin](https://github.com/QCObjects/qcobjects-admin) | handler (`qcobjects-handler` + `qcobjects-api`, package `v1.0.1`) | Admin panel for QCObjects apps. MUST be uninstalled before production deploys | stable, dev-only |
| [qcobjects-admin-lib-db-sqlite3](https://github.com/QCObjects/qcobjects-admin-lib-db-sqlite3) | admin storage (`qcobjects-admin-lib`) | SQLite3 backend for `qcobjects-admin` | stable |
| [qcobjects-lib-cosmosdb](https://github.com/QCObjects/qcobjects-lib-cosmosdb) | data lib | Microsoft CosmosDB adapter; configures via `$ENV(...)` (the pattern that proved cloud-native readiness for the v3 roadmap) | stable |
| [qcobjects-lib-sendemail](https://github.com/QCObjects/qcobjects-lib-sendemail) | data lib | Email sending via NodeMailer + Gmail (building block behind contact-form notifications) | stable |
| [qcobjects-lib-mailchimp-api](https://github.com/QCObjects/qcobjects-lib-mailchimp-api) | data lib | Mailchimp list subscription via the official API (building block behind contact-form notifications) | stable |
| [qcobjects-openai-api](https://github.com/QCObjects/qcobjects-openai-api) | AI proxy (`qcobjects-handler` + `qcobjects-api`, `v1.0.14`) | OpenAI chat-completions via the secret-hiding proxy pattern: browser `Service` → same-origin route → `BackendMicroservice` + `serviceLoaderNode` with server-side key. Ships browser UI component/controller + Node services + per-subpath `.cts/.mts/.ts` triplets | stable |
| [qcobjects-azure-openai-api](https://github.com/QCObjects/qcobjects-azure-openai-api) | AI proxy (`qcobjects-handler` + `qcobjects-api`, `v1.0.22`) | Same proxy architecture against Azure OpenAI endpoints (sibling of `qcobjects-openai-api`) | stable |
| [qcobjects-command-publish-static](https://github.com/QCObjects/qcobjects-command-publish-static) | command (`qcobjects-command`, `v1.0.4`) | **SUPERSEDED**: standalone `publish:static` command — now built-in to current `qcobjects-cli` (`src/cli-commands-publish-static.ts`). Do NOT install on new projects; kept for legacy CLI lines only | superseded |

## Rules (normative)

- New official add-ons MUST enter this table (role, keywords, purpose, status)
  in the same PR that publishes them.
- `superseded` add-ons stay listed one major CLI line with their replacement
  named, then move to an archived section.
- The admin panel MUST NEVER ship to production (`npm uninstall` before release
  builds; CI SHOULD fail the build if it is present in `dependencies`).

## Singleton consumption (normative, reference: github-chatapp-test + openai-api)

Beyond tag, widget, and loader instantiation, an add-on MAY export a
pre-instantiated component singleton the app appends directly:

- The package constructs at module scope (`export const chatbotComponent =
  new ChatBotComponent({name:"chatbot"})`, shadowed) and exports it alongside
  imperative helpers (`sendMessage()`, `closeChatbot()`) that attach the
  controller (`new ChatbotController({component})`) on demand.
- The app consumes it in boot code (`import chatbotComponents from
  "qcobjects-openai-api/components"`, then `document.body.append(
  chatbotComponents.chatbotComponent.body)` on `DOMContentLoaded`) — no
  `<component>` tag, no registry, no loader round-trip.
- Rules: singleton components MUST be self-sufficient (own template/handler
  inline or bundled — never depend on app-side `componentsBasePath`); controller
  attachment MUST be idempotent (re-calling helpers replaces, never duplicates);
  apps MUST NOT mutate the singleton's `name` (shared identity across imports).

## Verification

- Each `stable` row installs cleanly alongside the pinned core/SDK/CLI and its
  keyword autoloads under default flags.
- No shipped official add-on is missing from the table.
