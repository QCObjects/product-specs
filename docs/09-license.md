# 09 — License

## Purpose

Record the licensing position and the binding migration path to MIT.

Sources: `LICENSE.txt` headers in generated code + READMEs (FOSSA badges),
roadmap P1 mandate. Legal advice out of scope; this is the product position.

## Scope

All repos in the QCObjects GitHub org and every npm distribution.

## Current position (normative, v2.x)

- Core, SDK, and CLI distributions are **LGPL-3.0** (`LICENSE.txt` in-repo,
  `"license": "LGPL-3.0"` in `package.json`, FOSSA-tracked). The app template
  ships `"license": "LGPL-3.0-or-later"` — same family, different SPDX string.
- Every generated source file on the v2.x line MUST carry the LGPL header
  (quoted verbatim from source, with its stale link flagged inline):

```
QuickCorp/QCObjects is licensed under the GNU Lesser General Public License v3.0
[LICENSE] (https://github.com/QuickCorp/QCObjects/blob/master/LICENSE.txt)   <-- STALE: wrong org mirror + dead branch; canonical is https://github.com/QCObjects/QCObjects/blob/main/LICENSE.txt
Permissions of this copyleft license are conditioned on making available
complete source code of licensed works and modifications under the same
license or the GNU GPLv3. Copyright and license notices must be preserved.
Contributors provide an express grant of patent rights. However, a larger
work using the licensed work through interfaces provided by the licensed
work may be distributed under different terms and without source code for
the larger work.
Copyright (C) 2015 Jean Machuca,<correojean@gmail.com>
```

- Contributor Covenant `CODE_OF_CONDUCT.md` (carried by the core repo; SDK/CLI
  checkouts lack it — each migrating repo MUST add it); violations report to
  `info@quickcorp.cl`; contributions follow `CONTRIBUTING.md`.

## Migration to MIT (normative, v3.0 P1)

- Target: the entire ecosystem MUST migrate to **MIT** to remove linkage
  friction for enterprise/proprietary adoption.
- New repos on the v3.0 track (including this `product-specs` repo) MUST be
  MIT from day one.
- Each migrating repo MUST, in a single release: replace `LICENSE.txt` with the
  MIT text, set `"license": "MIT"` in `package.json`, swap the source-file
  header, and note the change in `CHANGELOG.md`. Partial states MUST NOT ship.
- v2.x EOL lines keep LGPL-3.0 headers untouched on archive tags; backports
  MUST NOT relicense old lines.
- CI SHOULD fail on stale headers post-migration.

## Verification

- `grep -ri "lesser general public" --include="*.ts" src/` returns empty post-migration.
- `npm view <pkg> license` reports `MIT` for the v3.0 line.
