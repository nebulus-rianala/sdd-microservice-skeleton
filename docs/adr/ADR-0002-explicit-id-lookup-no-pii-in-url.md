# ADR-0002: Explicit `{id}` lookup, never framework-magic binding, never PII in a URL

**Date:** {{SERVICE_INCEPTION_DATE}}
**Status:** Accepted
**Deciders:** Key Predict platform convention

## Context

Frameworks in both supported stacks offer a convenience feature that resolves a route
parameter straight into a model/entity instance before the handler runs (Laravel's
implicit route-model binding; an equivalent FastAPI dependency pattern that resolves-and-
injects). This convenience hides two things an AI-generated handler must not get wrong:
what happens when the ID doesn't resolve, and what the caller is allowed to put in a URL.
A resolved-instance parameter also makes it easy to accidentally accept a personally
identifiable value (email, name, token) as the lookup key in the URL, which then lands in
access logs and browser history in cleartext.

## Decision

Every endpoint that operates on a specific entity takes a raw integer `{id}` path
parameter and performs the lookup explicitly inside the handler/use case, handling the
"not found" case itself (mapped to HTTP 404 via the RFC 9457 error format). No URL path or
query string may ever contain an email, name, token, or other personal data — if a caller
needs to authenticate via a secret (e.g. a candidate access token), it is transmitted in
the request body, never the URL.

- Laravel: use `Model::findOrFail($id)` (or the service-layer equivalent) explicitly in the
  controller/service; do not use implicit route-model binding (see ADR-0006 in
  `docs/stack-profiles/laravel.md` for the Eloquent-specific version of this rule).
- FastAPI: the router depends on a use case that takes a plain `int` path parameter and
  raises a domain "not found" exception, translated to 404 by a FastAPI exception handler;
  do not resolve-and-inject the ORM model as a path-operation dependency.

## Consequences

- Every "not found" path is visible in the handler/use-case code, not hidden inside
  framework middleware — this makes it directly reviewable against the spec's error catalog.
- No PII ever appears in an access log or browser history via the URL.
- Slightly more boilerplate per endpoint (one explicit lookup call) in exchange for an
  explicit, spec-checkable failure mode.
