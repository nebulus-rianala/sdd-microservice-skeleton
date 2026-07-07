# GDPR Compliance

## Retention window

Every table holding personal data has an explicit retention period, configured via an
env-driven setting (e.g. `GDPR_RETENTION_DAYS`), stated in the technical spec's data-model
section. Do not hardcode a retention period as a literal in business logic.

## Purge procedure

A scheduled job/command enforces the retention window: it identifies expired rows and
either hard-deletes them or anonymizes them (per the legal basis established for that
data — see "Data minimization" below), then writes an audit-log row for the purge action
**before** it executes (see ADR-0003 — audit rows never depend on the row they describe
still existing).

## What survives deletion

Append-only audit/history tables survive a subject's hard-delete by design (no FK, see
`immutability-rules.md`). Every other child table of a deleted entity is expected to
cascade-delete unless the technical spec explicitly says otherwise for a named table.

## Access requests

The service technically supports the standard GDPR rights: access, erasure, portability,
objection. For each, the technical spec's "Authentication & Authorization" or a dedicated
GDPR section must state the exact endpoint or command that fulfills it — a right that
exists only in policy prose and not as a working code path is not implemented.

## Data minimization

Collect and store only the personal data the current feature set actually needs. Before
adding a new personal-data field, the interview in `generate-technical-spec.md` asks for
its legal basis and whether a DPIA is required (per `README.md`'s "Security by design"
section) — mark `SECURITY-REVIEW-REQUIRED` if unanswered rather than silently proceeding.

## GDPR request lifecycle

Track every rights-exercise request (who requested what, when, and its resolution) in its
own table or ticketing integration — this tracking record is itself an audit artifact and
follows the same append-only rules.

## Logging and auditability

Never log personal data at the DEBUG/INFO level. Structured logs may reference an entity
by its `INT` ID, never by name/email/other PII. See `logging-and-audit.md` for the full
logging rule set.

## Data classification and lifecycle

Every field holding personal or otherwise sensitive data is classified in the technical
spec's data-model section (public / internal / confidential / restricted), with the
handling that classification implies — encryption at rest, masking in non-production
exports, restricted read access. Track who accessed or exported classified data and why;
support-team access to personal data is scoped and purpose-logged, not a standing
unrestricted grant. Data minimization and the retention/purge procedure above are this
policy's concrete implementation of "collect and keep only what's classified as
necessary."
