# API Security

## Authentication boundary

**Kong API Gateway owns all authentication.** This service implements zero authentication
middleware. Every request that reaches this service has already been authenticated by
Kong. Do not add authentication middleware without an ADR and explicit approval.

- Laravel: no `auth:api` middleware, no JWT parsing, no session cookies.
- FastAPI: no OAuth2/JWT dependency on public routers.

## Trusted header model

Kong forwards caller identity via headers. This service reads and trusts them without
re-verification — Kong ensures they cannot be forged by an external caller.

| Header | Meaning |
|---|---|
| `X-Consumer-Groups` | Caller's role (e.g. `evaluator`, `admin`) |
| `X-Consumer-Id` | Caller's tenant/company scope (typically `s_odoo_partner_id` in this ecosystem) — **not** a personal user ID |

Do not implement header-signature verification — Kong handles this upstream. Do not
assume a header's absence means "no caller"; treat a missing expected header as a
configuration error (fail closed, return 403), not as anonymous/public access.

## Authorization model

Server-side access control on **every** endpoint and action, verified by
`(role × resource × action × tenant × business state)` — never by ID obscurity alone
(see ADR-0001, ADR-0002). Generate horizontal (cross-tenant) and vertical (role
escalation) authorization tests for every authz-relevant code path.

`/internal/*` routes are the one exception to the Kong-header model: they are called
service-to-service, never through a caller's Kong consumer context, and are instead
protected by a static `X-Internal-Api-Key` header checked against an env-provided value.
A mismatch returns 401.

## No direct database access from external callers

External callers (candidates, participants, or any non-authenticated-by-Kong actor)
never receive a raw database ID as their sole credential. Where a service needs to let an
external party access one specific record without a Kong session (e.g. a candidate
completing an assessment via an emailed link), issue a dedicated access token — see
`token-standards.md` — and require it in the request body, never the URL.

## CORS, rate limiting, HTTPS

All three are Kong's responsibility. Do not implement CORS handling, rate limiting, or
TLS termination in this service.

## Response security headers

Kong applies perimeter security headers (CSP, `Permissions-Policy`, `Referrer-Policy`,
`frame-ancestors`). This service does not need to duplicate them for its own JSON API
responses, but must never disable or override headers Kong has set.

## Error responses

Every error response is RFC 9457 Problem Details (`application/problem+json`) and must
never leak: a stack trace, a raw driver/ORM exception message, an internal file path, a
framework/library version, or the existence/non-existence of a resource the caller isn't
authorized to know about (return 404, not 403, when disclosing "it exists but you can't
see it" would itself be a leak).

## Rate limiting and public endpoints

Perimeter rate limiting is Kong's responsibility (see "CORS, rate limiting, HTTPS"
above). Every route this service exposes requires Kong-authenticated context by default;
a route that is intentionally public (no Kong consumer context expected) must say so
explicitly in the technical spec's API reference — "public" is never the silent default
for an undocumented route.

## Business-abuse protection (OWASP API6:2023)

Beyond authn/authz, protect sensitive business flows against automation: enforce
application-level business quotas (account creation, critical/irreversible operations,
export generation) in addition to Kong's perimeter rate limit, since a single caller
authenticated by Kong can still abuse a business flow within its own rate-limit budget.
Where the technical spec identifies a scraping/fraud/mass-creation/enumeration risk for an
endpoint, add detection (unusual request patterns, repeated near-miss lookups) rather than
relying on rate limiting alone.

## API inventory and governance (OWASP API9:2023)

Every endpoint this service exposes is documented in `docs/specs/api/openapi.yaml` —
owner, data classification, and exposure (Kong-authenticated / `/internal/*` /
intentionally public) — before it ships. Keep the OpenAPI spec in sync with the code; a
merged endpoint with no corresponding OpenAPI entry is a spec-drift bug, not a paperwork
gap. Define an explicit versioning and deprecation policy for any breaking change to a
published endpoint.
