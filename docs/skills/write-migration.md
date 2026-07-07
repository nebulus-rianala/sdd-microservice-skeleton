# Skill: Write a migration

Scope: create or modify a database table in a way that's reversible, portable between
`{{DB}}` choices (postgres default / mysql alternative), and consistent with the schema
conventions in the technical spec's `§Database Schema`/`§Database Design` section.

Reference spec: `docs/specs/technical/{{SERVICE_NAME}}-technical-spec.md §Database`,
and `docs/adr/README.md` (ADR-0001, ADR-0003, ADR-0004, ADR-0005 all constrain schema
shape).

## Responsibilities in mandatory order

1. **Check the spec first.** The table, its columns, and their types must already be
   described in the technical spec's data-model section. A migration that isn't backed
   by a spec section is a sign the spec is out of date — update the spec in the same
   change, don't silently diverge (see ADR-0003's principle applied to specs: the
   authoritative document must reflect reality).
2. **Generate, don't hand-author from scratch.**
   - `{{STACK: laravel}}` — `php artisan make:migration <description>`.
   - `{{STACK: fastapi}}` — `uv run alembic revision --autogenerate -m "<description>"`,
     then review the generated diff by hand before trusting it.
3. **Primary key.** `INT` auto-increment, never UUID (ADR-0001).
4. **Both directions.** Implement `up()`/`down()` (Laravel) or
   `upgrade()`/`downgrade()`(Alembic) — see ADR-0004. A no-op or `pass` down operation is
   a defect, not an acceptable stub.
5. **Timestamps.** `DATETIME`, not `TIMESTAMPTZ` — schema stays portable across `{{DB}}`
   choices. Every mutable entity gets automatic `created_at`/`updated_at`
   (`$table->timestamps()` / `server_default=func.now()` + `onupdate=func.now()`) — see
   ADR-0005. Append-only tables (step 8 below) get `created_at` only, never `updated_at`.
6. **Structured columns.** `JSON`/`JSONB`, never a serialized string column, for any
   field the spec describes as structured/nested data.
7. **Cross-service references.** An `s_`-prefixed `INT` column, never a foreign key
   (ADR-0001) — the referenced row lives in another service's database.
8. **Append-only tables.** If the table is one the technical spec marks append-only (see
   `docs/security/immutability-rules.md`), do not add an FK from it back to its subject
   entity if that FK would cascade-delete (ADR-0003); do not give the model a
   soft-delete/update path.
9. **Soft-delete columns.** Where the spec calls for soft-delete on a *mutable* entity,
   use an explicit `active` (boolean) + `archived_at` (nullable datetime) column pair
   driven by the entity's own state machine — not a framework soft-delete trait.
10. **Verify before merging.** Apply the migration, confirm the schema matches the spec
    exactly (column names/types/nullability/defaults/indexes), then revert it and
    re-apply it once more to prove the down operation actually works — don't just eyeball
    the down operation's code.

## Recurring patterns worth naming

- **Enum-backed columns**: create the enum type (or check constraint, for `{{DB: mysql}}`
  or SQLite-compatible test environments) before the table that uses it, and drop it
  after the table in the down operation.
- **Unique / partial-unique constraints**: e.g. "only one active row per group" needs a
  partial unique index (`WHERE is_active = true`), not an apparently-simpler flat unique
  constraint that would block having any historical inactive rows.
- **Self-referential FK**: nullable, with an explicit `ON DELETE` behavior chosen
  deliberately (not left at the driver default) and stated in the spec.
