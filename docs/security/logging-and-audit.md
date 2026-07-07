# Logging, Audit Trail, and Detection

Ref: OWASP A09, ASVS.

## Structured application logging

Emit structured (JSON) logs with an ISO 8601 UTC timestamp and a correlation
`request_id` that can be traced across this service and the services it calls or is
called by. Never log: a password, token, secret, or sensitive health/payment/PII data —
see `gdpr-compliance.md`'s "Logging and auditability" for the PII-specific rule.
Structured logs may reference an entity by its `INT` ID, never by name/email/other
personal data. Where a feature's tests can regress this, add a test asserting the
absence of sensitive fields in the emitted log line. Send logs to the centralized
destination (SIEM/log aggregator) already configured for this service; do not introduce
a second, unaggregated log sink.

## Immutable audit trail

Separate from application logs, this service maintains an immutable audit log (see
`immutability-rules.md` for the storage-level append-only enforcement) recording, at
minimum:

- admin actions,
- permission/role changes,
- data exports,
- access to data classified as sensitive (see `gdpr-compliance.md`'s data classification),
- security-configuration changes.

Every privileged operation defined in `authentication.md` writes an audit-log row as part
of the same transaction/operation, not as a best-effort side channel that can silently
fail.

## Detection and incident response

Emit security-relevant events and metrics that the alerting system can actually act on:
authentication failures, authorization denials (`ACCESS_DENIED`, `IDOR_ATTEMPT` —
logged without exploitable detail about *why* access was denied), and business-level
anomalies the technical spec identifies as abuse indicators (see `api-security.md`'s
business-abuse protection). Where a feature introduces a new class of security event,
document in a comment or the technical spec what alert threshold is expected to fire on
it — an event nobody is alerting on is not meaningfully "monitored."
