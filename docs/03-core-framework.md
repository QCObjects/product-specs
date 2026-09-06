# 03 — Core Framework (`qcobjects`)

## Purpose

Specify the core package: what it exports, how classes/packages work,
and what it deliberately does NOT do.

## Scope

Repo `QCObjects/QCObjects`, npm `qcobjects` (v2.5.142 at time of writing).
Build/test detail lives in [14-build-scripts-blueprint](./14-build-scripts-blueprint.md).

## Normative

- Package entry points MUST be: `main → public/cjs/index.cjs`,
  `module → public/esm/index.mjs`, `browser → public/browser/QCObjects.js`,
  `types → public/types/index.d.ts`, with `exports` maps for `.`, `./package.json`,
  `./tsconfig*`, `./*.js|cjs|mjs`, and wildcard `./*`.
- The class system (`Class.ts`, `InheritClass.ts`, `ClassFactory.ts`, `New.ts`,
  `RegisterClass.ts`, `NamespaceRef.ts`, `Package.ts`) MUST support:
  `Package()` namespacing, `Class()` definition with `_new_` constructor,
  single inheritance via `InheritClass`, and reflective lookup.
- MVC primitives MUST include at minimum: `Component` (+ `ComponentFactory`,
  `componentLoader`), `Controller`, `View` (+ `DocumentLayout`), `VO`, `DDO`,
  `Service` (+ `serviceLoader`), `Effect`/`TransitionEffect`.
- Client services MUST cover: `CONFIG`/`ConfigSettings` (settings resolution),
  `localStorage`/`ComplexStorageCache`, `asyncLoad`/`componentLoader`,
  `routing` (`routings.ts`), `i18n` messages, `Crypt`/`secretKey`,
  `Base64`/`Cast`/`DataStringify` codecs.
- Core MUST NOT embed: HTTP servers, CLI parsing, PWA shell, or widget CSS.
  Those belong to the CLI / SDK / app layers.
- Every public symbol MUST ship type declarations under `public/types/`.
- Specs live in `spec/` (jasmine: `testsSpec`, `testsConfigSpec`,
  `testsClassFactorySpec`, `testsGlobalFeaturesSpec`, `testsTypeSpec`);
  `npm test` MUST run lint + the full suite green.

## Verification

- `node -e "require('qcobjects')"` and `import 'qcobjects'` both resolve.
- `npx tsc --noEmit -p tsconfig.d.json` passes; jasmine suite passes.
