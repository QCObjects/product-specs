# 19 — Secrets and API settings management

How QCObjects moves credentials from "only ops knows them" to "the framework
can use them" — and the limits of that promise.

Sources (read live at time of writing): `QCObjects` core `CONFIG`,
`ConfigSettings`, `Crypt`, `secretKey`, `serviceLoader`, SDK session tokens,
CLI `defaultsettings`, template `config.json`, `qcobjects-openai-api`.

## The three rules everything else hangs off

1. **Browser must never own a secret.** All credentialed third-party API
   calls go through a server-side proxy. The browser posts to a same-origin
   route; the route holds the key, calls the vendor, returns the response.
2. **Secrets enter the runtime only as `$ENV(NAME)` / `$ENV(NAME,fallback)`** —
   never as literals in `config.json` or in source. Resolved at read time by
   the processor chain ([18-environment-variables](./18-environment-variables.md)).
3. **You cannot "register a secret" at runtime and have it persist.** The
   framework's own crypto is XOR+Base64 obfuscation for in-memory caching,
   not real secret storage (§ below). If you need a secret stored encrypted
   at rest, encrypt it yourself with a real cipher and store the key in the
   environment.

## What `config.json` may contain

**Template-shipped `config.json` is plain-text and committed.** Look at any
tracked `config.json` (the canonical `qcobjects-new-app` one): every
credential field is a `$ENV(VAR)` string, not a literal. The binding wins or
loses at deploy/OS-level env var time — never at build time, never at
scaffolding time, and *especially never* committed with real keys.

Example from the template (verified against `v2.5.x` at time of writing):

```json
{
  "backend": {
    "auth": {
      "enabled": true,
      "defaultUser": "$ENV(DEFAULT_USER)",
      "defaultPasswd": "$ENV(DEFAULT_PASSWORD)",
      "microsoftapikey": "$ENV(MICROSOFT_API_KEY)",
      "googleapikey": "$ENV(GOOGLE_API_KEY)"
    }
  }
}
```

There is nothing to "encrypt at rest" here because none of these values are
secrets at rest — they are placeholders resolved at read time by the
processor chain. The pipeline is:

```text
shell env (export OPENAI_API_KEY=sk-...)
    ↓
OS-level env var (only this machine / this container / CI secret block)
    ↓
QCObjects Processor.ENV handler (registered at CLI startup in defaultsettings.ts)
    ↓
CONFIG.get("OPENAI_API_KEY") → process.env["OPENAI_API_KEY"] at read time
```

## Server-side API proxy (normative pattern)

**Reference: `qcobjects-openai-api`** — and by extension any credentialed
third-party API you adopt as an add-on. The canonical shape:

- **Client (browser)**: a `Service` subclass (`ProxyOpenAIService`) POSTs to a
  same-origin route (`/api/openai`, `external:false`, `cached:false`), carrying
  only the request body — NO key, NO `Authorization` header.
- **Server**: a `BackendMicroservice` reads the key from
  `CONFIG.get("OPENAI_API_KEY")` (operating on the Node leg where `process.env`
  is real), builds an upstream request with the key, executes via
  `serviceLoaderNode`, and writes the response into `this.body`. Errors surface
  as `this.body + done()` — fail closed with a body, not a stream write.
- **Rules**: the browser MUST NOT import, read, or hold any value of the key;
  the proxy MUST be same-origin; the key MUST come from `$ENV(...)`.

If you find yourself writing `headers.Authorization = "Bearer sk-..."` in a
front-end service class, stop — your route is server-side on Node; move the
call there.

## What `_secretKey` is — and what it is not

`_secretKey` is the obfuscation key used by `Crypt.encrypt/decrypt`: in
browsers, `location.host`; in Node, the literal string `"secret"`. It is
stored in the framework (public), not in the app. The `Crypt` implementation
is byte-wise XOR with a rotating key offset plus Base64 encoding — this is
**obfuscation**, not encryption. `_secretKey` is public knowledge by
construction (it's `location.host` in every deployed app).

**What it does:**

- Encrypts the in-memory `_CONFIG_ENC` payload inside the `CONFIG` singleton
  between `CONFIG.set(...)` and `CONFIG.get(...)` calls. It's symmetric
  obfuscation of a runtime cache, never a wire value.
- Encrypts SDK session tokens (`SessionUserToken`):
  `_Crypt.encrypt("userAgent|username|timestamp", origin)` — again,
  obfuscation with a ubiquitous key, not cryptographic secrecy.
- Derives the `__encoded__` value in encrypted `config.json` files at app
  startup (`defaultsettings.ts` decrypts the whole `__encoded__` blob with
  the `_secret_` fallback or `domain`).

**What it does not do:**

- It does not make the browser safe to hold secrets. Browsers hold `"secret"`
  or, worse, `location.host`; any code in the bundle can call
  `_Crypt.decrypt(anything,anything)`.
- It does not protect config values at rest beyond making them non-trivial
  to grep. Do not treat `__encoded__`-wrapped config files as confidential
  against a determined reader of the source repo.
- It is not a session-auth mechanism — `SessionUserToken` is a fingerprinting
  gift, not an authentication check.

## `__encoded__` (pre-encrypted config blob)

Some deployments ship a `config.json` whose single `__encoded__` field
contains the entire encrypted config object. At startup, the CLI does:

```ts
const _secretKey = (Object.hasOwn(_config, "domain"))
  ? (_config["domain"])
  : ("_secret_");
if (Object.hasOwn(_config, "__encoded__")) {
  _config = JSON.parse(_Crypt.decrypt(_config.__encoded__, _secretKey));
}
```

Notable: the decrypt key is either the `domain` field from the same plain
file, or `"_secret_"` — both are public in the repo. This is **not a real
secret-encryption boundary**; it is obfuscation for the transport of a
config blob, not for defense-in-depth. If you ship a config with `__encoded__`,
treat it as `PLAINTEXT` for secret-enagement purposes unless the domain value
is itself secret (and then re-encrypted into the blob so the outer `domain`
field is decoy — the only way this scheme confounds an attacker).

## `ComplexStorageCache` — session memory only

The `__cache__` object in `SessionUserToken` writes into
`ComplexStorageCache`, which caches per-instance and per-username in the
application's JS heap. It is **never** a persistence mechanism: refresh the
page, everything vanishes; same user in two tabs shares no cache (each has
its own runtime); no signing, no expiry beyond the timestamp argument inside
the token itself (which is a timestamp literally printed into the plain text
of the token — observable by anyone).

## Hard rules (normative)

- **No secrets in repositories, ever.** Not in `config.json`, not in
  `src/`, not in comments. If you find one, rotate it — it's already a
  leakage incident, not a pending nit.
- **Frontend service classes MUST NOT receive API keys.** Pass them to
  `Service` only from a server-side `BackendMicroservice` post-dispatch,
  never from the boot path.
- **`window.QCO = {}`, `global.අ`** — the QCObjects runtime
  **publicly exports** `_Crypt`, `shortCode`, `ComplexStorageCache`. Do not
  assume any of these are hidden; build your secret flows as if they were
  known utilities.
- **`process.env` is not the model.** Env vars are a transport and
  operator-convenient default; they are not a solution on their own. For
  third-party APIs call proxies or vault services (AWS Secrets Manager, 1Password,
  etc.) with a short-lived token instead of baking an API key anywhere in
  the bundle.
- **Rotate before you sleep.** If a config with `__encoded__` leaks, rotate
  whatever was decrypted from it. The encryption buys you nothing relative
  to publication of the repo.

## When you need real secrecy

- **Browser ↔ vendor secret:** never. Use a same-origin proxy.
- **Server ↔ vendor secret:** `process.env` in the run environment, or a
  vault-provisioned short-lived token — never in `config.json` literals,
  even encrypted.
- **Client ↔ server authentication for user sessions:** the built-in
  `SessionUserToken` is fingerprintable obfuscation; pair with a real
  backend session (JWT in `HttpOnly` cookie) in the route, or adopt an
  add-on like `qcobjects-admin` for OAuth flows.
- **Session secrets at rest (Node):** read from `process.env` via
  `$ENV(SESSION_SECRET)`; never let a `Buffer`/`_Crypt` of it survive the
  process.
- **App secrets inside source (e.g., a hardcoded basic-auth password):**
  don't. There is nothing in the framework that can protect you from the fact
  that anyone reading the repo has the password.

## Quick reference

| Need | Where it lives | How it reaches runtime |
|---|---|---|
| API key for a third-party vendor (proxy call) | `process.env["OPENAI_API_KEY"]` (CI secret / container env) | `$ENV(OPENAI_API_KEY)` resolved at `CONFIG.get("…")` |
| Database password | `$ENV(DEFAULT_PASSWORD)` in `config.json` |Same |
| User password (client login) | Browser obfuscation via `SessionUserToken` | In-memory only, per-tab, self-hosted |
| Encrypting app config for deploy | `__encoded__`-wrapped file | CLI decrypts at startup with `_secret_` / `domain` |
| Server ↔ client auth token bits | `__secretKey` = `location.host` / `"secret"` | `_Crypt.encrypt/decrypt` |

Secrets are a transport problem everywhere in QCObjects — the framework
receives them, uses them, and lets them go. The responsibility for
their effectiveness at rest and in transit lives entirely in the operator
environment, the host platform, and you.
