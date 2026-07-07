# Runbooks

Operational knowledge built up during implementation: how to safely run a background
command, handle a stuck record, debug a failed job, recover from an edge case. Written as
each feature that needs one is delivered — don't pre-write a runbook for an operation that
doesn't exist yet.

## Template

```markdown
# Runbook: [Operation name]

**Service:** · **Command / Endpoint:** ·
**Risk level:** Low | Medium | High · **Reversible:** Yes | No | Partial

## When to use this runbook
## Pre-conditions
## Steps
## Verification
## Rollback
## Escalation
```

## General rules

- Always dry-run first where the command supports it.
- Prefer the framework's own management-command runner
  (`docker compose exec app php artisan <command>` / `docker compose exec api python -m
  scripts.<script>`) over ad hoc SQL against production.
- Any GDPR-related operation (purge, erasure, export) must be logged in the incident
  tracker / audit trail **before** execution, per `docs/security/gdpr-compliance.md`.
