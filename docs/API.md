# TrenchNote — Public API Contract (v1)

This document is the **boundary** decided in
[ADR 0005](adr/0005-core-premium-extension-boundary.md): the interface that
premium features, integrations, and any third-party tool may build on —
all with exactly the same access. If it isn't described here, it isn't
contract, and building on it is at your own risk.

TrenchNote's API **is** [PocketBase's standard REST
API](https://pocketbase.io/docs/api-records/) — the core adds collections
and rules, not custom endpoints. Everything below is reachable at the same
origin that serves the pages (e.g. `http://192.168.1.50:8090`).

## Contract collections

These five collections — their fields, semantics, and the operations marked
allowed — are stable. A breaking change to any of them requires a new ADR
and a version bump of this contract, announced in the release notes.

| Collection | Read (list/view) | Create | Update | Delete |
|---|---|---|---|---|
| `items` | ✔ contract | ✔ | ✔ | admin-only |
| `locations` | ✔ contract | ✔ | ✔ | admin-only |
| `assets` | ✔ contract | ✔ | ✔ (see cache rule) | admin-only |
| `movements` | ✔ contract | ✔ (see XOR rule) | **never** | **never** |
| `reservations` | ✔ contract | ✔ | ✔ | admin-only |

Field-level shapes are defined by the migrations in `pb_migrations/` and
explained in [ARCHITECTURE.md](ARCHITECTURE.md#data-model). Highlights that
are load-bearing for API clients:

- **`movements` is an append-only ledger** (ADR 0002). No client — premium,
  third-party, or future core code — can ever update or delete a movement.
  Corrections are new records.
- **The XOR create rule** ("XOR" = one or the other, never both): a
  movement is either an asset move (`asset` set, `item` empty,
  `quantity = 0`) or a bulk move (`asset` empty, `item` set,
  `quantity > 0`). Enforced server-side; malformed records are rejected no
  matter who sends them.
- **`assets.current_location` is a cache, not truth.** Clients that log an
  asset move must write the movement record *first*, then PATCH the cache —
  in that order, always.
- **Bulk stock is derived, never stored.** Compute stock-on-hand per
  location by summing bulk movements (in minus out). There is no stock
  column, and there never will be (ADR 0002).
- **`tag_code` is permanent** once printed on a label (ADR 0004): unique,
  never recycled onto different gear.

## Other contract surface

- **Query features:** PocketBase's `filter`, `sort`, `expand` (including
  nested expands like `asset.item`), and pagination (`page`, `perPage`,
  max 500 per page) on the collections above.
- **File URLs:** `/api/files/{collection}/{recordId}/{filename}` — e.g. an
  item's photo at `/api/files/items/{id}/{photo}`.
- **Realtime:** PocketBase's SSE subscriptions ("server-sent events" — a
  one-way stream the server pushes changes over) at `/api/realtime`, for
  the contract collections. Prefer this over tight polling loops; the
  server may be a Raspberry Pi whose first job is serving field scans.

## Not contract

The internal SQLite file layout and `pb_data/` contents, the PocketBase
admin UI and admin-only endpoints, PocketBase system collections, and any
endpoint or behavior not listed here. These can change without notice.

## Authentication

Phase 1 rules are deliberately open for LAN use (see the security note in
the [README](../README.md)) — so today, anonymous requests work. **That is
an artifact, not the contract.** The contract is: when the Phase-2 auth
lockdown lands, API clients authenticate as an ordinary PocketBase auth
record (a "service account" — credentials of their own, permissions of any
authenticated client, never a superuser key). Build clients so a token can
be added to requests later without restructuring.

## Versioning

- This is **contract v1**. Additive changes (new optional fields, new
  collections) don't bump the version; breaking changes (renamed/removed
  fields, changed rules, changed URL patterns) require an ADR and bump this
  document's version, announced in release notes.
- Core is currently developed and tested against **PocketBase 0.28.x**
  (pin with `PB_VERSION=0.28.4 ./scripts/setup.sh`). A PocketBase upgrade
  that changes REST behavior is treated as a breaking change and handled
  the same way.
