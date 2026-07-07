# Cloud and Infrastructure-as-Code Security

Ref: NIST SSDF.

## Infrastructure as code

Generated IaC must pass a static scanner (Checkov, tfsec, or equivalent) with no critical
findings before merge. Apply least-privilege IAM throughout — no wildcard (`*`) on
actions or resources in a policy statement generated for this service.

## Network defaults

Resources default to a private network with controlled, explicit egress rather than open
by default. Enable volume/disk encryption at rest by default (this is the "managed
storage encryption" option referenced in `cryptography.md`). Never generate a publicly
readable/writable storage bucket or resource without an explicit
`SECURITY-REVIEW-REQUIRED` marker calling out why it must be public.

## Audit visibility

Enable the cloud provider's own audit logging (e.g. CloudTrail, Activity Log) for
infrastructure this service owns, so infrastructure-level changes are traceable
independently of the application's own audit trail (`logging-and-audit.md`).
