# Token Standards

Applies to any bearer/access token this service issues to an external, non-Kong-
authenticated caller (e.g. a candidate/participant access link, a one-time confirmation
token). Does not apply to Kong's own session/auth mechanism, which this service never
implements (see `api-security.md`).

## Format

Define one canonical external representation per token type in the technical spec (e.g.
Base58 — excludes ambiguous characters `0 O I l` and non-alphanumeric `+ / =` — for
tokens a human might need to read or type). Do not invent a new encoding per feature.

## Entropy

Minimum 256 bits (32 random bytes) of entropy per token. Never derive a token from
predictable input (sequential ID, timestamp, email hash).

## Generation

Generate with a cryptographically secure random source only:

- Laravel: `random_bytes()`. Never `rand()`, `mt_rand()`, or `uniqid()`.
- FastAPI/Python: `secrets.token_bytes()` / `secrets.token_urlsafe()`. Never `random`
  module functions.

## Storage

Store the token (or a salted hash of it, if the threat model in the technical spec calls
for it) in a dedicated column, never reused as or derived from the primary key.

## Lookup

Look the token up via an explicit, indexed query (`findActiveToken`-style service method) —
never accept it as a route/path parameter that gets logged in access logs; see
"Transmission" below.

## Validity

Every token has an explicit validity window and/or single-use/consumed state, enforced
server-side at lookup time — an expired or already-consumed token must fail lookup the
same way a nonexistent one does (constant response shape, to avoid leaking which case
occurred).

## Transmission

Tokens are transmitted in the **request body**, never in the URL (path or query string) —
URLs land in access logs, browser history, and referrer headers, all of which are outside
this service's control once the token is exposed there.
