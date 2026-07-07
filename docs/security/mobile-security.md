# Mobile Security — if applicable

Ref: OWASP MASVS. Applies only where this service has a dedicated mobile client
consuming its API; skip this file entirely for a service with no mobile consumer.

## Token storage

Store tokens in the platform's secure storage — Keychain (iOS) / Keystore (Android) —
never in cleartext in app-private storage, shared preferences, or a local file.

## Local logging

Never write sensitive data (tokens, personal data) to local device logs; the same rule
as `logging-and-audit.md` applies on-device, not just server-side.

## Platform security features

Delegate biometric authentication to the OS-provided APIs rather than implementing a
custom biometric flow. Apply certificate pinning where the threat model justifies the
added operational cost of pin rotation.
