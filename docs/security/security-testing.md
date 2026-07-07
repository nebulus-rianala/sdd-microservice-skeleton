# Security Testing

Ref: OWASP ASVS, OWASP SAMM.

## Baseline expectation

Every feature that touches authorization, input handling, or a sensitive data flow ships
with security tests alongside its functional tests: at least one nominal (expected/legal
use) case and one abuse case per identified threat, not just the happy path.

## Authorization tests

Any authz-relevant code path (see `api-security.md`'s authorization model) generates both
horizontal tests (can user/tenant A read or modify user/tenant B's resource? — must fail)
and vertical tests (can a lower-privileged role perform a higher-privileged action? —
must fail). Cross-tenant tests specifically required by `tenant-isolation.md` are part of
this same baseline, not an optional extra.

## API security tests

Where the technical spec's API reference defines an endpoint, cover: invalid/malformed
input (schema violations, boundary values), missing or wrong authorization context, and
behavior once a rate limit or business quota is exceeded (see `api-security.md`'s
business-abuse section).

## Static analysis and scanning

If the project does not already have a SAST/SCA/IaC-scan integration wired into CI,
propose adding one (see `dependency-policy.md` and `cicd-container-security.md`) rather
than treating its absence as acceptable for a new feature.

## Coverage bar

Security tests count toward, but do not replace, the stack's overall coverage gate
(≥ 80%, per the relevant `stack-profiles/*.md`) — a feature is not "tested" if its only
coverage is the happy path.
