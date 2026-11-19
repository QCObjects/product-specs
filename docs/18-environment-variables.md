# Environment variables and `$ENV(…)` resolution

This page documents how environment variables reach `config.json`, `CONFIG`, and
runtime behavior in QCObjects apps. It is the operational counterpart to the
processor mechanics in [03-core-framework](./03-core-framework.md).

Short version:

> `$ENV(NAME)` / `$ENV(NAME,default)` in **any string** processed by a QCObjects
> processor is replaced with the value of `process.env.NAME`. Templates,
> component attributes, scripts, stylesheets, and `CONFIG` values all honor this.

## Resolution chain

```text
┌────────────────────────────┐
│ shell env (DEVMODE, DOMAIN)│   process.env.*
└─────────────┬──────────────┘
              │
┌─────────────▼──────────────┐
│ defaultsettings.ts          │   setProcessor(const ENV = …)
│ (node/CLI entry, always)    │   - registers ENV handler with Processor
└─────────────┬──────────────┘
              │  CONFIG.set("foo", "$ENV(DEVMODE)")
┌─────────────▼──────────────┐
│ ConfigSettings.instance     │   encrypted at rest: _CONFIG_ENC
│                             │   plain object: _CONFIG
└─────────────┬──────────────┘
              │  CONFIG.get("foo")
┌─────────────▼──────────────┐
│ ProcessObject(obj)          │   recurses strings × processors
│ └─ process(template)        │   substitutes $<NAME>(<args>) placeholders
└─────────────┬──────────────┘
              │  resolved values
┌─────────────▼──────────────┐
│ logger.debugEnabled,        │
│ CONFIG.get("domain") …      │
└─────────────────────────────┘
```

### The four places that call the processor

1. **CLI startup (`defaultsettings.ts`)** — registers the comma-default form
   `$ENV(VAR,default)` as a `Processor` handler and applies it to every string
   in `config.json` as values are read via `CONFIG.get`.
2. **App `config.json`** — every string in the file honors `$ENV(…)`, resolved
   at read time (not at load time). Example: `"devmode": "$ENV(DEVMODE,info)"`
   means Debug/Info/Warn depends on what the shell provided when starting the
   process, not on the value written at build time. A plain string in a
   template like `"port": 8080` is indistinguishable from a resolved one.
3. **Component templates** — templates (e.g., `.tpl.html`) are processed by the
   `Component` runtime via `Processor.process(str, component)` at parse time
   (`DefaultTemplateHandler.assign` runs `Processor.processObject` on the
   whole template string). Because substitution is done by global regex, any
   string containing `$ENV(NAME)` will be substituted — including inside
   `<style>` and `<script>` blocks. `SourceCSS`/`SourceJS` themselves
   do not run the substitution for external files; the template-level pass
   does it before the CSS/JS is attached.

## The `ENV` processor itself

Defined in `src/defaultsettings.ts` (CLI) with **comma-separated default**
syntax:

```ts
function ENV(_component: any, arg: string) {
  if (typeof process === "undefined") { return ""; }
  const commaIndex = arg.indexOf(",");
  if (commaIndex === -1) { return process.env[arg.trim()] ?? ""; }
  const varName = arg.substring(0, commaIndex).trim();
  const defaultValue = arg.substring(commaIndex + 1).trim();
  return process.env[varName] ?? defaultValue;
}
Processor.setProcessor(ENV);
```

Core's processor (`src/Processor.ts`) recognizes the name `ENV` without the
comma form and returns `process.env[name] ?? ""`.

### Supported syntaxes

| Syntax | Handler | Missing value → |
|---|---|---|
| `$ENV(NAME)` | core processor, CLI processor | empty string `""` |
| `$ENV(NAME,default)` | CLI processor (registered) | `default` literal |

Shell-side: always pass the wrapper the way you expect. Quote values holding
spaces or `#`; `$ENV(DOMAIN,localhost)` needs no quoting; `$ENV(API_KEY,\#tag)`
would need single-quotes around the placeholder or a literal value instead.

## Encrypted config values

Encrypted `config.json`: strings beginning `"__encoded__":` hold a base64 +
encrypted payload produced by the Config Encryptor tool. The wire form looks
like a string, so it is run through the same `$ENV(…)` substitution — which
is the exact reason a misencrypted literal `"ENC($ENV(FOO))"` does not leak
into runtime; the processor expands only values that actually parse as
`$NAME(args)` placeholders inside plain strings. Do NOT prefix encrypted
values with `$ENV` for "automatic" resolution — the ciphertext itself has no
structure and will never match the processor's regex; you intend to hit a
different processor. Encrypted config values resolve to whatever `CONFIG.get`
is asked for and never participate in the `$ENV(…)` expansion.

## Practical checklist for app authors

- `.env` files are NOT auto-loaded. The QCObjects CLI reads `process.env`
  directly. Put `export DEVMODE=debug` in your shell profile, a CI secret, or
  pass it inline: `DEVMODE=debug NODE_ENV=production npm start`.
- For Netlify-style deployments, set the same variable names in the host's
  environment settings UI (matching the names referenced by `$ENV(…)` in your
  `config.json`).
- Ship a `.env.example` listing every `$ENV(VAR,default)` referenced. A
  reviewer can then confirm the defaults are safe (e.g., `info` for
  `devmode`, `localhost` for `domain`).
- App templates have no `.env`, no `.envrc`, no shell wrapper — environment is
  supplied by the operator or CI.

## What `$ENV(…)` does NOT cover

- **`config.json` file discovery.** The CLI looks for `config.json` at
  `CONFIG.get("projectPath")/config.json`. `projectPath` is set from
  `process.cwd()` at startup, not via `$ENV(…)`.
- **Ports / listen addresses.** `server.listen(process.env.PORT || CONFIG.get("serverPortHTTP"))`:
  if `PORT` is set in the process env, it wins regardless of `config.json`.
  Putting `"serverPortHTTPS": "$ENV(PORT, 8080)"` in `config.json` also works —
  `CONFIG.get("serverPortHTTPS")` expands through the processor chain at read
  time. The config.json default is the numeric string `"8080"` / `"8443"` — no
  `$ENV(…)` shape, passes through unchanged.
- **Shell expansion.** `DEVMODE=debug` in your shell sets `process.env.DEVMODE`
  in the Node process it launches; it does not globally export into the HTTP
  stack's `serverPort*` sockets beyond the startup moment.
- **Compile-time bundling.** Placeholders survive the build; they fire at
  runtime. Distributing a minified or prebundled `config.json` is therefore
  safe only if the host resolves those variables at runtime, not baked in at
  build time.
- **`\` escapes inside the placeholder itself.** `$ENV(\\nNAME)` is a literal
  backslash-n-NAME. The placeholder parser reads `\$ENV` as-is and looks up
  `process.env["\\nNAME"]`, so \_with\_ no escape-aware lookup. Use a real name.
- **Lookup of nested `.` names.** `$ENV(foo.bar)` looks up
  `process.env["foo.bar"]`. No nesting in the lookup — no `process.env.foo.bar`.
- **Binary values.** `process.env` values are strings; a base64-blob in
  `$ENV(CERT)` yields a raw base64 string you must decode yourself before use.

## Deferred reads vs. startup reads

`CONFIG.set(x, "$ENV(DEVMODE)")` stores the literal string `$ENV(DEVMODE)`.
`CONFIG.get(x)` expands it to `"debug"` on every read. Two consequences:

1. A value read at startup (e.g., for `setDevMode` in `defaultsettings.ts`)
   is expanded once, at startup, with whatever env was present then.
2. A value read after the process received a new env var (e.g., via a test
   hook that mutates `process.env`) re-expands to the new value — the stored
   literal is deterministic per `$ENV(NAME)` expression, not per call-site
   history.

## Related docs

- [19-secrets-and-api-settings](./19-secrets-and-api-settings.md) — the secrets chains that sit on top of this resolver.


- [03-core-framework](./03-core-framework.md) — the processor mechanics.
- [05-cli](./05-cli.md) — CLI runtime context and startup.
- [06-app-structure](./06-app-structure.md) — app file layout and env contract.
- [12-schemas](./12-schemas.md) — `config.json` shape and validation.
