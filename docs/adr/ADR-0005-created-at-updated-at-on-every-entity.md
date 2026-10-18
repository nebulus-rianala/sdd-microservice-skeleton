# ADR-0005: Every entity table carries automatic `created_at`/`updated_at`

**Date:** {{SERVICE_INCEPTION_DATE}}
**Status:** Accepted
**Deciders:** Skeleton platform convention

## Context

Debugging, support investigations, sorting/pagination ("most recently updated first"),
and a good number of business rules ("only within 24h of creation") all depend on knowing
when a row was created and when it last changed. Left as a per-table decision, this drifts:
some tables get both columns, some get only one, some get them but populated by
hand-written application code that's one missed code path away from going stale (e.g. a
bulk `UPDATE` that never touches `updated_at`). Both frameworks in use have a native,
battle-tested mechanism for this that costs nothing to adopt uniformly, so there's no
reason to leave it to per-table judgment.

## Decision

Every table backing a mutable entity has `created_at` and `updated_at` columns, type
`DATETIME` (per the existing DATETIME-not-`TIMESTAMPTZ` portability convention), stored
in UTC, managed automatically — application code never sets or edits these columns by
hand.

- Laravel: rely on Eloquent's default (`$table->timestamps()` in the migration; do not
  set `public $timestamps = false`). Any bulk update that bypasses Eloquent's query
  builder events (`DB::table(...)->update(...)`, `Model::query()->update([...])` without
  loading models) must explicitly set `updated_at` in that same call — Eloquent does not
  patch this in for you outside of `save()`/model-event-driven updates.
- FastAPI/SQLAlchemy: declare `created_at` with `server_default=func.now()` and
  `updated_at` with `server_default=func.now()` **and** `onupdate=func.now()`, so the
  timestamp is enforced at the column-default level — correct even for a raw `UPDATE`
  issued outside the ORM session, not only ORM-mediated writes.
- **Append-only / immutable tables** (see `docs/security/immutability-rules.md`) get
  `created_at` only. Do not add `updated_at` to an append-only table — a column that
  implies "this can change" is actively misleading on a table that, by design, never
  receives an update.
- Both columns are excluded from every inbound DTO/`FormRequest`/Pydantic request schema
  (see `docs/security/input-validation.md`) — a caller can never set or override either
  value through the API; they are server-managed only.

## Consequences

- Every mutable table is sortable/debuggable by recency without a per-table decision or a
  spec section deciding whether "this one needs it too."
- A table that genuinely has no independent lifecycle worth timestamping (e.g. a pure
  many-to-many pivot table with no other attributes) is still expected to state that
  explicitly in the technical spec's data-model section — silently omitting the columns
  is a spec gap, not an accepted exception.
- Bulk/raw updates that bypass the ORM's normal save path are a known footgun for
  `updated_at` specifically; code review checks for this the same way it checks
  immutability-rule compliance (see `docs/security/immutability-rules.md`'s note that this
  class of rule isn't fully provable by static analysis alone).
- This is unrelated to, and does not replace, the audit trail in ADR-0003 —
  `created_at`/`updated_at` describe an entity's own row lifecycle; the audit log
  describes actions taken against it (by whom, what changed) and lives in a separate,
  append-only table.
