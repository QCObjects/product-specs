# 11 — Features

## Purpose

Catalogue user-visible features with their status per repo — plus every delivery
channel — so plans see what exists, what is partial, and what is roadmap.

Sources: core README §§ Installing, Demo, PWA features; SDK/CLI/new-app READMEs.

## Scope

Features across core, SDK, CLI, templates. Status: `stable`, `beta`,
`planned` (target phase from spec 15), `eol`.

## Feature matrix (normative)

| Feature | Core | SDK | CLI | Templates | Status |
|---|---|---|---|---|---|
| Class system + MVC primitives | ✅ | — | — | — | stable |
| Native `class`/`extends` + `new` interop | ✅ | ✅ | — | ✅ | stable |
| Package/Import/Export + routing | ✅ | — | — | — | stable |
| Template meta processors (`$mapper/$layout/$component/$repeat`) | ✅ | — | — | ✅ | stable |
| Smart widgets (custom elements) | ✅ | ✅ | — | ✅ | stable |
| Nested component routing (`{param}`) | ✅ | — | — | ✅ | stable |
| Storage cache, i18n, Crypt, codecs | ✅ | — | — | — | stable |
| Array/Collection/math helpers | ✅ | — | — | — | stable |
| Transpiler-free runtime + TS authoring + first-party types | ✅ | ✅ | ✅ | ✅ | stable |
| ShadowedComponent + RegisterWidget | — | ✅ | — | demo | stable |
| Form/DataGrid/Modal/Swagger controllers | — | ✅ | — | demo | stable |
| Grid/List/Slider/Splash/Notifications | — | ✅ | — | demo | stable |
| 15-effect catalogue + Timer | — | ✅ | — | demo | stable |
| CanvasTool, BasicLayout, GridView | — | ✅ | — | demo | stable |
| Cloud-auth session (token) | — | ✅ | — | demo | beta |
| Scaffold (`create`), HTTP/HTTPS/HTTP2 + GAE | — | — | ✅ | — | stable |
| Collab server, shell, createcert | — | — | ✅ | — | stable |
| esbuild + tsc builds, publish-static | — | — | ✅ | ✅ | stable |
| Synced semver (`v-*`) + changelog | — | — | ✅ | ✅ | stable |
| PWA shell (manifest, SW, offline, lazy-src) | — | — | template | ✅ | stable |
| `$ENV()`/`$config()` + custom processors | ✅ | ✅ | ✅ | ✅ | stable |
| Keyword autoload (libs/handlers/commands) | — | — | ✅ | ✅ | stable |
| Encrypted `config.json` | ✅ | — | ✅ | ✅ | stable |
| Backend routes + `BackendMicroservice` | ✅ | — | ✅ | ✅ | stable |
| SSR via FileDispatcher (`useTemplate`) | — | — | ✅ | ✅ | stable |
| PHP handler bridge | — | — | ✅ | — | beta |
| Deno support | — | — | ✅ (`deno.json` + `mod.ts` in CLI) | — | beta (no `mod.ts` entry point ships in core at `v2.5.142`) |
| MIT license line | — | — | — | — | planned (P1) |
| Bun server adapter | — | — | — | — | planned (P2 SAL) |
| Wasm handler (Rust/AssemblyScript) | — | — | — | — | planned (P3) |
| FastAPI sidecar bridge | — | — | — | — | planned (P3) |
| Local-first LLM orchestration | — | — | — | — | planned (P3) |

## Delivery channels (normative — every channel below MUST keep working)

- **npm:** `npm install qcobjects-cli -g && npm install qcobjects --save`
  (`qcobjects-sdk@v2.4` line as documented).
- **CDN:** `cdn.qcobjects.dev/QCObjects.js` (dev) · jsDelivr
  (`cdn.jsdelivr.net/npm/qcobjects/QCObjects[.min].js`) · UNPKG
  (`unpkg.com/qcobjects@latest/QCObjects.js`) · CDNJS
  (`cdnjs.../qcobjects/[VERSION]/QCObjects[.min].js`).
  ESM form supported: `<script type="module">import
  "https://cdn.qcobjects.dev/QCObjects.js"</script>` (reference: effects demo).
  `useSDK:true` + `useLocalSDK:false` loads the SDK remote instead of vendored.
- **Docker:** `quickcorp/qcobjects-playground` (playground) ·
  `qcobjects/qcobjects-newapp` (app, ports 8080/8443).
- **One-step scripts:** Ubuntu 18.x / RHEL8 / Raspbian 9 / macOS installers from
  `cdn.qcobjects.dev` (fresh-machine only warning applies); Windows = NodeJS +
  `npm i qcobjects-cli -g` + `qcobjects create --pwa`.
- **Cloud:** DigitalOcean 1-Click Droplet; AWS AMI + PIB (Marketplace listing).
- **Editors:** Atom `qcobjects-syntax`; VS Code `Quickcorp.QCObjects-vscode`.
- **Demos:** `newapp.qcobjects.dev` (PWA) + Foundation/Materialize/raw-CSS samples.
- **Desktop (Electron):** same `src/` tree in a shell trio (`main.js` with
  `nodeIntegration:true` + `preload.js` + `renderer.js`, `package.json`
  `"main": "main.js"`, `electron` dep); publish via the template's
  `npm run publish:electron` script (a wrapper — the CLI has no native
  `electron` publish target).
- **Hybrid mobile (PhoneGap/Cordova):** `res/` icons + `.pgbomit`, same web tree.

## Rules (normative)

- New features MUST enter the table as `planned` with phase before code lands;
  status advances in the shipping PR.
- `eol` features stay listed one major line with a migration pointer.
- Dropping a delivery channel REQUIRES a major bump + migration note.

## Verification

- Each `stable` cell has a passing test or demo route proving it.
- No shipped feature is missing from the table; every channel installs/serves.
