# 09 — License

## Purpose

Record the licensing position and the binding migration path to MIT.

## Scope

All repos in the QCObjects GitHub org and every npm distribution.
Legal advice is out of scope; this is the product position.

## Normative

- Current position (v2.x): core and SDK distributions are licensed
  **LGPL-3.0** (`LICENSE.txt` in-repo, `"license": "LGPL-3.0"` in package.json).
- Target position (v3.0): the entire ecosystem MUST migrate to **MIT** to
  remove linkage friction for enterprise and proprietary adoption.
- New repos created for the v3.0 track (including this `product-specs` repo)
  MUST be MIT from day one.
- Each migrating repo MUST, in a single release: replace `LICENSE.txt` with
  the MIT text, set `"license": "MIT"` in `package.json`, and note the change
  in `CHANGELOG.md`. Partial states (MIT text + LGPL field) MUST NOT ship.
- v2.x lines entering EOL MUST keep their LGPL-3.0 headers untouched on
  archive tags; backports MUST NOT relicense old lines.
- Every source file carrying a license header MUST be updated to the new
  header in the migration release; CI SHOULD fail on stale headers.

## Verification

- `grep -ri "lesser general public" --include="*.ts" src/` returns empty post-migration.
- `npm view <pkg> license` reports `MIT` for the v3.0 line.
