# Immutability Rules

## Append-only tables

Any table that exists to record a fact about the past — audit logs, state-transition
history, reconnection/access logs — is append-only. Once a row is written, it is never
updated and never deleted, including by the entity it describes being hard-deleted (see
ADR-0003). See `logging-and-audit.md` for which specific actions must produce an audit-log
row in the first place; this file covers the storage-level enforcement once that decision
is made.

The technical spec's data-model section must explicitly list which tables in this service
are append-only.

## Enforcement pattern

- Laravel: override `save()` (or the relevant persistence method) on the Eloquent model to
  throw a `LogicException` if `$this->exists` — i.e. any attempt to write to an
  already-persisted row of an immutable model fails loudly, in code, not just in
  documentation.
- FastAPI: the repository implementation for an append-only entity exposes only an
  `insert`/`create` method on its port — no `update`/`delete` method exists on the
  interface at all, so calling code cannot even attempt it; this is enforced by the type
  system, not a runtime check.

This is fundamentally a **code review gate** — static analysis cannot fully prove
immutability on its own, so any PR touching an append-only table's model/repository must
be checked by hand against this rule.

## Do not add soft-delete to an immutable model

Soft-delete implies the row can later be un-deleted or its `deleted_at` toggled — that's a
mutation. Immutable/append-only models do not get a soft-delete trait/mixin.

## Do not add an update path to an immutable table

If a future feature seems to need "correcting" a historical row, the correct pattern is
writing a new row that supersedes it (an explicit correction record), never editing the
original. This preserves the audit trail's own integrity.

## Mutable tables

Every other table is mutable by default. Soft-delete on a mutable entity (where the
technical spec calls for it) is implemented explicitly via an `active`/`archived_at`-style
field pair combined with the entity's own state machine, not a framework soft-delete
trait, so the "deleted" state participates in the same validated transition logic as every
other state.
