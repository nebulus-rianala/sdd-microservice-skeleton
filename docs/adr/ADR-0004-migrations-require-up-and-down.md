# ADR-0004: Every migration implements both an up and a down operation

**Date:** {{SERVICE_INCEPTION_DATE}}
**Status:** Accepted
**Deciders:** Skeleton platform convention

## Context

A migration that only knows how to move forward turns every deploy into a one-way door:
if a schema change is bad, the only recovery path is a new forward migration undoing it —
slower, riskier, and it litters the migration history with corrective noise. Both
frameworks in use support a reversible migration natively; skipping the reverse operation
is always a shortcut, never a technical necessity.

## Decision

Every migration must implement a working reverse operation, and it must actually be
tested (applied then reverted then re-applied) before merge — not just present as an
empty stub.

- Laravel: every migration class implements both `up()` and `down()`.
- FastAPI/Alembic: every revision implements both `upgrade()` and `downgrade()`. Always
  generate with `alembic revision --autogenerate`, never hand-author a revision from
  scratch, and never edit a revision that has already been applied to a shared environment
  — write a new one instead.

## Consequences

- Any deploy can be rolled back without hand-writing an emergency corrective migration
  under time pressure.
- A migration whose down operation is `pass`/a no-op comment is a defect, not an
  acceptable stub — code review must reject it.
- Destructive down operations (e.g. dropping a column that had data) must be flagged
  `SECURITY-REVIEW-REQUIRED` per `docs/security/backup-recovery.md` if
  the rollback itself would be irreversible.
