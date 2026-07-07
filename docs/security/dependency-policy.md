# Dependency Policy

## Audit gate

A dependency-vulnerability audit runs in every CI pipeline and blocks merge on any
non-zero exit:

- Laravel: `composer audit`
- FastAPI: `uv` / `pip-audit` equivalent, run alongside `ruff`/`mypy` in the lint stage

No exception without an ADR-recorded waiver stating the CVE, why it's not exploitable in
this service's context, and the remediation SLA (see "Vulnerability management" below).

## Adding new dependencies

- Verify the exact package name before adding it — never hallucinate a package name or
  guess at a plausible-sounding one (anti-typosquatting; this is a `MUST`, not a
  style preference — see `README.md`'s AI behavior rules).
- Prefer a framework-native solution or a dependency already present in the project
  before introducing a new one.
- For any dependency that isn't trivially justified, flag it, name its license, and mark
  the addition `SECURITY-REVIEW-REQUIRED` for human sign-off.

## Vulnerability management

If a known vulnerability (CVE) is detected in the code or in a dependency in use, flag it
explicitly rather than propagating it silently. Any accepted waiver records a remediation
SLA by severity: **P0 < 24h, P1 < 7 days, P2 < 30 days, P3 next sprint** — state which
tier applies and why in the ADR-recorded waiver above.

## Pinning strategy

Pin exact versions and commit the lock file (`composer.lock` / `uv.lock`). Never widen a
version constraint just to make an installation succeed without understanding why it
failed.

## Approved dependency categories

Dev-only tooling (test frameworks, static analyzers, formatters) belongs in the
dev-dependency group, never the production dependency group.

## Removed or deprecated packages

When a package is dropped, remove its config, its `.env` keys, and any dead code that
only existed to call it — do not leave orphaned integration code "just in case."

## Language/runtime version policy

Track the language/runtime version (PHP / Python) as an explicit, tested minimum in the
stack profile; upgrading it is a deliberate decision recorded as an ADR update to the
relevant stack profile, not an incidental side effect of a dependency bump.
