# Tenant Isolation

Ref: OWASP API1/API3:2023.

## Every query is tenant-scoped

Every database query this service runs on behalf of a caller is filtered by that
caller's tenant/company scope (`X-Consumer-Id`, per `api-security.md`) — there is no such
thing as a global query without a tenant constraint in a multi-tenant table. This applies
equally to list/search endpoints, background jobs operating on a tenant's data, and
exports.

## Cross-tenant tests are mandatory

Any endpoint or job that reads or writes tenant-scoped data ships with a test asserting
that tenant A cannot read or affect tenant B's data through it, even by guessing or
enumerating an ID (see `security-testing.md`, and ADR-0002's rule against relying on ID
obscurity).

## Per-tenant key separation

Where the technical spec's threat model calls for it (e.g. a tenant explicitly requiring
data isolation at the cryptographic level, not just the query level), separate encryption
keys per tenant rather than a single shared application key — see
`secret-management.md`/`cryptography.md` for key handling once this is decided.
