# Backup and Recovery

Ref: NIST SSDF.

## Irreversible operations

Never generate code that performs an irreversible deletion without a safeguard: a
soft-delete/confirmation step, or — where the technical spec explicitly calls for a true
hard delete (e.g. a GDPR erasure request, see `gdpr-compliance.md`) — an explicit
`SECURITY-REVIEW-REQUIRED` marker on the code path, since hard-delete code is exactly
where a bug has no undo. See ADR-0004 for why every migration in particular must ship a
working `down`/`downgrade` — a migration is itself a potentially irreversible operation
if it can't be rolled back.

## Recovery objectives

Where a change affects the data schema or a backup-relevant table, document the
recovery-point and recovery-time objective (RPO/RTO) impact in the technical spec, and
flag anything that would make a documented RPO/RTO no longer achievable (e.g. a table
that grows fast enough to blow a backup window).
