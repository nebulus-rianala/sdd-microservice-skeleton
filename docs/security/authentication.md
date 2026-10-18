# Authentication

Ref: OWASP ASVS, NIST SSDF.

**Applicability**: in the default skeleton architecture, Kong owns authentication and
this service implements none (see `api-security.md`'s "Authentication boundary"). This
file applies only when a service is itself the authentication authority — e.g. an
identity-provider-style service, or a service issuing its own bearer tokens to
non-Kong-authenticated external callers (see `token-standards.md` for that narrower
case). Do not add a login/session subsystem to a Kong-fronted service without an ADR
justifying why Kong's model doesn't cover the use case.

## Core mechanism

- Use a centralized, standard authentication mechanism (OAuth2/OIDC) — do not build a
  home-grown auth system.
- Hash passwords with Argon2id (explicit memory/time/parallelism parameters) or bcrypt
  (cost ≥ 12). Never unsalted MD5/SHA1/SHA256.
- Compare password hashes using a constant-time comparison from the library — never `==`
  or a hand-rolled comparison.
- Check new passwords against a breach list (e.g. Have I Been Pwned's range API) at
  sign-up, where the technical spec calls for it.

## Sessions and cookies (OWASP ASVS 5.0)

- Issue session cookies with `HttpOnly`, `Secure`, and `SameSite=Strict` or `Lax`.
- Regenerate the session identifier after authentication succeeds (anti session-fixation).
- Invalidate the session server-side at logout — never rely on the client discarding the
  cookie.
- Generate session identifiers with a CSPRNG, minimum 128 bits — never `Math.random()` or
  a language's non-cryptographic RNG.
- Enforce an inactivity timeout.
- Where the technical spec requires it, limit the number of concurrent sessions per user.

## Multi-factor authentication

- Require MFA for: admin accounts, privileged/back-office accounts, production access,
  and CI/CD accounts with deploy privileges.
- Prefer FIDO2/WebAuthn or TOTP; do not build a new SMS-OTP flow (SIM-swap risk).

## JWT verification (if this service is a resource server)

- Verify the JWT against an algorithm allowlist (RS256/ES256 or equivalent asymmetric
  algorithm). Reject `alg: none` and reject any token whose header claims an unexpected
  algorithm.
- Validate `iss`, `aud`, `exp`, `nbf`, and `jti` server-side on every request — not only
  at issuance.
- Implement revocation (a `jti` denylist, or a DB-backed check) and refresh-token
  rotation.
- Detect refresh-token reuse and revoke the entire token family, then alert.
- Set access-token lifetime to 15 minutes or less.
- Do not store a JWT in `localStorage`; prefer an `HttpOnly` cookie or in-memory storage.

## Anti-brute-force

- Apply combined rate limiting by account, IP, and device — not just one dimension.
- Apply progressive (exponential) backoff on repeated failures.
- Keep response content and timing constant regardless of whether the account exists
  (anti account-enumeration).
- Trigger a contextual CAPTCHA beyond a configurable failure threshold, where the
  technical spec calls for it.

## Privileged operations

- Every privileged operation (admin action, permission change, data export, security
  configuration change) requires MFA and reinforced access control beyond the baseline
  authorization check.
- Every privileged operation is recorded in the immutable audit log — see
  `logging-and-audit.md`.
- Support and admin roles are distinct in code, not the same role with a flag.
- For critical/irreversible operations, prefer a "four-eyes" (second-approver) workflow
  where the technical spec calls for it.
