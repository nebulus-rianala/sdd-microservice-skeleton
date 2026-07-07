# ADR-0003: Audit log tables have no FK to their subject and survive hard-delete

**Date:** {{SERVICE_INCEPTION_DATE}}
**Status:** Accepted
**Deciders:** Key Predict platform convention

## Context

GDPR erasure requests require the ability to permanently delete a subject's data.
Standard practice would give an audit-log table a foreign key to the entity it audits, so
the database enforces referential integrity. But if that FK cascades (or even just blocks)
on delete, either the audit evidence disappears along with the data it was recording
("who deleted what, when"), or the erasure itself becomes impossible without first
destroying the audit trail — both are unacceptable for a compliance record.

## Decision

Audit/log tables that record actions taken on an entity (creation, state changes, admin
actions, exports, access to sensitive data — see `docs/security/gdpr-compliance.md` and
`docs/security/logging-and-audit.md`) have **no foreign-key constraint** on the subject
entity's ID. The subject ID is stored as a plain `INT` column. The audit row is written
**before** any delete/erasure operation executes, and is never deleted when its subject is
hard-deleted.

## Consequences

- Audit evidence outlives the data it describes — a hard-deleted entity's audit trail
  remains queryable after erasure, which is the whole point of an audit log for a
  compliance-relevant action.
- The database cannot enforce "this audit row always points to a real entity" — that
  invariant is not needed (audit rows are append-only historical facts, not live
  relationships) and its absence is intentional, not an oversight. Do not add the FK back
  to "fix" an orphaned-row warning.
- Immutable-table rules apply on top of this (see `docs/security/immutability-rules.md`):
  these tables are append-only; no `UPDATE`, no `SoftDeletes`, no ORM-level "delete".
