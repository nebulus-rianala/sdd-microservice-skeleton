# Architecture Decision Records

Immutable records of significant technical decisions. Once accepted, an ADR is never
edited — a reversed decision gets a new ADR that marks the old one
`Superseded by ADR-NNNN`. See `TEMPLATE.md` for the format and rules.

Every Claude Code and Codex session loads this index before generating code, so past
decisions aren't silently re-litigated.

## Index

| ADR | Decision | Scope |
|---|---|---|
| [ADR-0001](ADR-0001-int-primary-keys-not-uuid.md) | INT primary keys, not UUIDs | universal |
| [ADR-0002](ADR-0002-explicit-id-lookup-no-pii-in-url.md) | Explicit `{id}` lookup, never framework-magic binding, never PII in a URL | universal |
| [ADR-0003](ADR-0003-audit-log-no-fk-survives-deletion.md) | Audit log tables have no FK to their subject and survive hard-delete | universal |
| [ADR-0004](ADR-0004-migrations-require-up-and-down.md) | Every migration implements both an up and a down operation | universal |
| [ADR-0005](ADR-0005-created-at-updated-at-on-every-entity.md) | Every mutable entity table carries automatic `created_at`/`updated_at` | universal |
| [laravel-ADR-001](../stack-profiles/laravel.md#adr-001-chunk-over-cursor-for-batch-write-operations) | `chunk()` over `cursor()` for batch write operations | laravel |
| [laravel-ADR-002](../stack-profiles/laravel.md#adr-002-job-constructors-carry-primitive-ids) | Job constructors carry primitive IDs, never Eloquent model instances | laravel |
| [laravel-ADR-003](../stack-profiles/laravel.md#adr-003-failed-hook-not-try-catch-for-job-status) | `failed()` hook, not try/catch, for terminal job-failure status | laravel |
| [laravel-ADR-004](../stack-profiles/laravel.md#adr-004-laravel-11-scheduler-via-withschedule) | Laravel 11 scheduler registered via `withSchedule()` in `bootstrap/app.php` | laravel |
| [fastapi-ADR-001](../stack-profiles/fastapi.md#adr-001-arq-over-celery) | ARQ over Celery for background tasks | fastapi |
| [fastapi-ADR-002](../stack-profiles/fastapi.md#adr-002-hexagonal-architecture) | Hexagonal (ports & adapters) architecture, strict `domain → application → infrastructure → api` layering | fastapi |

## Rules

- ADRs are **immutable once accepted**. Never edit the Decision or Context of a past ADR.
- To reverse a decision: write a new ADR with status `Accepted`, set the old one's status
  to `Superseded by ADR-NNNN`.
- Number sequentially in this folder (`ADR-0001`, `ADR-0002`, …), zero-padded to 4 digits,
  no gaps. Stack-only decisions are numbered separately within their stack profile and are
  still listed here for discoverability.
- A new decision forced by an interview answer during technical-spec generation (see
  `docs/skills/generate-technical-spec.md`) becomes a new ADR file here, never an inline
  edit to `CLAUDE.md` or the technical spec alone.

## Policy change process

1. Open the change as a PR.
2. Write the ADR (new file, next sequential number).
3. Update every story/spec section that the decision affects.
4. If any already-delivered week's audit record (`docs/audit/`) is affected by the change,
   note it in that audit record — audit records are append-only, so add a note, don't
   rewrite history.
