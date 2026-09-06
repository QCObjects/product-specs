# 06 — Resulting App Structure

## Purpose

Pin the layout every QCObjects app MUST follow so tooling, docs, and
deployments can assume it.

## Scope

As produced by `qcobjects create` / `src/templates/*` and exemplified by
`qcobjects-new-app` (v2.4.40-ts at time of writing).

## Normative

```
myapp/
  config.json          # runtime truth: domain, ports, routes, paths ($ENV/$config)
  package.json         # scripts: start/serve/collab/shell/createcert, test, lint
  src/
    index.html         # shell (or build/index.html → public/)
    js/
      config.ts        # CONFIG settings binding
      init.ts          # boot sequence (Init component)
      customWidgets.ts # app widgets registration
      packages/        # Package() namespaces: org.myapp.*
    css/  img/  templates/  favicon.ico  manifest.json  sw.js
    404.html  robots.txt  humans.txt
  public/              # build output only (parcel/esbuild distDir)
  spec/                # jasmine specs mirroring src/
```

- `config.json` MUST define: `domain`, ports, `documentRoot` (+ `documentRootFileIndex`),
  `relativeImportPath`, `backend.routes[]` (each with `name`, `path` regex,
  `microservice`, `redirect_to`/`responseHeaders`/`cors` as needed).
- `src/js/init.ts` MUST boot exactly one root component; feature code MUST live
  under `src/js/packages/<org>.<app>.*` namespaces.
- `public/` MUST be generated artifacts only — never hand-edited, never the
  place for source.
- PWA files (`manifest.json`, `sw.js`, `robots.txt`, `404.html`) MUST exist in
  every production app; `sw.js` MUST NOT cache authenticated API responses.

## Verification

- `npm run build` from clean checkout reproduces `public/` byte-equivalent config.
- `npx eslint "src/**/*.ts"` passes; jasmine `spec/` suite passes.
