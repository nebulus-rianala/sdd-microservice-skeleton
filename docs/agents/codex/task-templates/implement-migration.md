# Codex task: implement a migration

## Context

- Spec: `docs/specs/technical/{{SERVICE_NAME}}-technical-spec.md` §Database
  [FILL: table name(s)]
- Story: `docs/user-stories/features/week-{{WEEK}}/{{ID_PREFIX}}-[FILL: story id].md`
- Skill: `docs/skills/write-migration.md` (read before starting — has copy-paste patterns
  for enum columns, JSONB, soft-delete pairs, partial unique indexes, self-referential FKs)

## Task

Create the migration for `[FILL: table name(s)]` exactly as the spec's data-model section
describes. This is a **focused, single migration** task — do not touch application code
in this task.

## Tables to create / alter

| Table | Columns |
|---|---|
| [FILL] | [FILL: name : type : nullable? : default, one row per column] |

## Constraints

- `INT` auto-increment primary key, never UUID (ADR-0001).
- Both directions implemented and must actually work — `up()`/`down()` (Laravel) or
  `upgrade()`/`downgrade()` (Alembic) (ADR-0004).
- `DATETIME`, not `TIMESTAMPTZ`, for timestamp columns.
- `JSON`/`JSONB` (per stack) for structured columns, never a serialized string column.
- Cross-service reference columns: `s_`-prefixed `INT`, never a foreign key (ADR-0001).
- If this table is marked append-only in the spec: no FK back to its subject that would
  cascade-delete (ADR-0003), no update/soft-delete path.
- `{{STACK: laravel}}`: generate with `php artisan make:migration`.
- `{{STACK: fastapi}}`: generate with `alembic revision --autogenerate`, then hand-review
  the diff.

## Verification steps (report back that you did these, don't just claim it)

- [ ] Migration applies cleanly against a fresh test database
- [ ] Schema matches the spec exactly (column names/types/nullability/defaults/indexes)
- [ ] Down/rollback operation applies cleanly and returns the schema to its prior state
- [ ] Re-applying after rollback succeeds (round-trip tested, not just eyeballed)
