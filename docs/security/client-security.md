# Client (Browser/SPA) Security

Ref: OWASP Top 10, ASVS. Applies where this service's API is consumed by a browser-based
frontend, or where the service itself serves any HTML/JS.

## No secrets on the client

Never place a long-lived secret (API key, service-to-service credential) in client-side
code or a bundle shipped to the browser. A browser client authenticates through Kong /
the mechanisms in `authentication.md` and `token-standards.md`, never through an embedded
static credential.

## XSS prevention

Emit a Content-Security-Policy using a nonce or hash per response, without
`unsafe-inline` unless explicitly justified and marked `SECURITY-REVIEW-REQUIRED`. Encode
or sanitize output according to context (HTML/JS/CSS/URL). Never use
`dangerouslySetInnerHTML` (React) or `v-html` (Vue) on unsanitized input — run it through
a sanitizer (e.g. DOMPurify) first.

## Local storage discipline

Minimize what's kept in `localStorage`/`sessionStorage`; never put a bearer token or
personal data there when an `HttpOnly` cookie or in-memory storage is viable (see
`authentication.md`'s JWT-storage rule).

## Frontend dependencies

Scan frontend dependencies for known vulnerabilities the same way backend dependencies
are scanned (see `dependency-policy.md`) — a frontend supply-chain compromise is not a
lesser risk than a backend one.
