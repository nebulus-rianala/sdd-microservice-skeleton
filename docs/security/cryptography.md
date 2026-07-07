# Cryptography

Ref: OWASP ASVS, NIST SSDF. Covers algorithm choice and use; key **storage, rotation, and
governance** live in `secret-management.md` — this file is about what you encrypt/sign
with, not where the key lives.

## Approved algorithms only

Never write home-grown cryptography — no custom cipher, no custom hash construction, no
"obfuscation" treated as encryption. Use the language/framework's standard cryptography
library or a proven, named library (e.g. `libsodium`, `sodium_crypto_*`, the
`cryptography` package in Python) — never a hand-rolled primitive.

## Application-level encryption

Where data must be encrypted at the application level (as opposed to relying on managed
storage encryption — see below), use an AEAD construction: AES-256-GCM or
ChaCha20-Poly1305, with a unique nonce/IV generated per encryption operation — never a
reused or predictable nonce.

## Prefer managed encryption where sufficient

Encrypt at the application level only when the threat model requires it (e.g. field-level
encryption for a specific sensitive column). Otherwise, delegate to managed storage
encryption (cloud provider disk/volume encryption + KMS) rather than re-implementing it —
see `cloud-iac-security.md` for the infrastructure-level default.

## Transport encryption

Enforce TLS 1.2 or higher for all communication; TLS 1.3 is recommended where the
runtime/infrastructure supports it. This service does not terminate TLS itself (Kong
does, per `api-security.md`), but any outbound call this service makes to another service
or third party must itself use TLS — never plaintext HTTP for anything carrying
authentication material or personal data.

## No hardcoded keys

Never hardcode a cryptographic key or secret in code, even as a placeholder default that
"falls back" to something real. See `secret-management.md` for where keys are read from
instead.
