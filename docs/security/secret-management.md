# Secret Management

## What is a secret

Any API key, password, private key, signing secret, or credential — including ones for
non-production environments. A non-prod secret is still a secret — see "Non-production
environments" below for the rule that non-prod uses its own distinct secrets, never a
copy of production's.

## Rules

### Never commit a secret

No secret, of any kind, in any versioned file — code, config, fixtures, test data,
migrations, or commit messages. `.env` (or its stack equivalent) is gitignored; only
`.env.example` (with placeholder values, never real ones) is committed.

### Never hardcode a secret

Not even as a "temporary" default or an example value in a comment. Read secrets from the
environment / a vault at runtime, always.

### Environment variable naming

Use a consistent, greppable naming convention (e.g. `{{SERVICE}}_API_KEY`,
`{{SERVICE}}_API_KEY_NAME` for the outbound header name when calling another service).

### Config files, not direct env-read calls, in business code

- Laravel: `env()` is only allowed inside `config/*.php`. Services and controllers read
  from `config()`, never call `env()` directly — this keeps config cacheable and testable.
- FastAPI: secrets are read via `pydantic-settings` (`app/core/config.py`), never via
  ad-hoc `os.environ` calls scattered through the codebase.

### Cloud storage / KMS — key governance

Where a secret is used to encrypt or sign data at rest, prefer managed KMS/Vault-backed
keys over an application-level secret. Read keys from a dedicated KMS/Vault, never from a
value baked into config at deploy time. Separate keys by usage (e.g. one key per purpose,
not one shared key for everything) and by environment (prod keys never usable against
staging/dev data or vice versa). Restrict KMS access to least privilege and audit key
usage. See `cryptography.md` for the algorithms these keys are used with.

### Rotation

Application secrets and encryption keys (e.g. a framework-level encryption key, a
signing key) must support scheduled rotation without downtime, and emergency revocation
if compromised. Document the rotation procedure in `docs/runbooks/` once the service has
one.

## Local development

Local `.env` values are never production secrets, even for read-only/staging-mirroring
convenience — use synthetic or scoped-down local credentials.

## Non-production environments

Never copy raw production data into a dev/test/staging environment — fixtures and test
data are synthetic or irreversibly anonymized. Non-production configs use secrets that are
distinct from production's, never a downgraded copy of the same credential.
