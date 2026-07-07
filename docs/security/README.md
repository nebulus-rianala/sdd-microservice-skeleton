# Security Policies

Global security constraints that apply to **every story, every week, by default**. This
`README.md` plus every domain-specific policy file below are loaded at the start of every
Claude Code and Codex session alongside the technical spec. They are not deliverables —
they are guardrails. User stories reference these documents; they do not redefine them.

This file carries the rules that apply **regardless of domain** — always load it. Load
only the domain files relevant to the feature being implemented; that's the point of the
split below.

## Level convention

- **`MUST`** — Non-negotiable generation constraint. The code is compliant or the AI
  session stops and asks.
- **`SHOULD`** — Strongly recommended. Apply unless explicitly justified in a comment.
- **`ASK`** — Ask the question / request the missing context before generating, or make an
  explicit assumption and mark it `SECURITY-REVIEW-REQUIRED: <reason>`.

## AI behavior rules (apply on top of every domain policy below)

- **MUST** apply every applicable `MUST` directive in every loaded policy file, without
  exception.
- **MUST** never invent a function, library, API, or security primitive. Use only
  existing, verifiable components; when in doubt, say so rather than hallucinating a
  secure-looking call.
- **MUST** never write home-grown cryptography (see `cryptography.md`).
- **MUST** never produce a hardcoded secret, even as an example (see
  `secret-management.md`).
- **MUST** treat all external input as untrusted — user, third-party API, file,
  environment variable, database content, or the output of another AI (see
  `input-validation.md`).
- **MUST** favor safe defaults: deny by default, fail closed.
- **MUST** state the assumptions made (framework, version, assumed threat model) at the
  top of the response or in a comment.
- **MUST** mark any point requiring a human decision with `SECURITY-REVIEW-REQUIRED:
  <reason>` — a waiver, missing context, a security-architecture choice, or handling of
  sensitive data.
- **MUST** refuse, or propose the compliant alternative, rather than silently satisfying a
  request that would violate a `MUST` directive.
- **SHOULD** accompany the code with security tests: nominal case + abuse case (see
  `security-testing.md`).
- **SHOULD** tie the code back to the specific policy directive it satisfies, in a comment,
  where non-obvious.
- **SHOULD** prefer the simplest solution that satisfies the security constraints.

## Security by design — before writing a single line

Ref: NIST SSDF SP 800-218, OWASP SAMM, OWASP Top 10 A04. Work through this **before**
coding a new feature or an architecture change, not after:

- **ASK** the assets handled by the code to be generated (data, services, secrets) and
  their classification. If unknown, ask, or assume the worst case and flag it.
- **ASK** which trust boundaries are crossed (network, tenant, privilege).
- **MUST** enumerate the relevant abuse cases before coding — what would an attacker do?
- **MUST** derive the feature's security requirements from those abuse cases.
- **MUST** include security acceptance criteria in the generated tests.
- **SHOULD** produce a mini threat model in a header comment: assets, actors, trust
  boundaries, sensitive flows, residual risks.
- **ASK**, if personal data is processed: the GDPR legal basis and whether a DPIA is
  required (see `gdpr-compliance.md`). Mark `SECURITY-REVIEW-REQUIRED` if unanswered.

## Documents

Domain files, loaded selectively depending on what the feature touches:

| File | Covers |
|---|---|
| [authentication.md](authentication.md) | Login mechanisms, password hashing, sessions/cookies, MFA, JWT verification, anti-brute-force, privileged-operation access control — applies only to a service that implements its own authentication rather than delegating entirely to Kong |
| [api-security.md](api-security.md) | Kong trust boundary and trusted header model, authorization model, business-abuse/quota protection, API inventory & versioning, error responses |
| [input-validation.md](input-validation.md) | Server-side validation of every external input, SQL-injection prevention, mass-assignment protection, SSRF, unsafe deserialization/parsing, file uploads |
| [cryptography.md](cryptography.md) | Approved algorithms, AEAD/encryption-at-rest, TLS enforcement — key storage and rotation live in `secret-management.md` |
| [token-standards.md](token-standards.md) | Entropy, generation, storage, transmission of any bearer/access token this service issues |
| [secret-management.md](secret-management.md) | What counts as a secret, where secrets may and may not live, KMS/Vault key governance, rotation, non-production environments |
| [logging-and-audit.md](logging-and-audit.md) | Structured security logging, PII-in-logs prohibition, the immutable audit trail (which actions must be recorded), detection/alerting |
| [immutability-rules.md](immutability-rules.md) | Which tables are append-only, how that's enforced in code, what "mutable" means for everything else |
| [gdpr-compliance.md](gdpr-compliance.md) | Retention window, purge procedure, what survives deletion, access requests, legal basis, data classification and lifecycle |
| [dependency-policy.md](dependency-policy.md) | Audit gates, adding new dependencies, pinning, approved categories, known-vulnerability handling and remediation SLAs |
| [cicd-container-security.md](cicd-container-security.md) | Non-root containers, security context, image provenance, CI security gates |
| [cloud-iac-security.md](cloud-iac-security.md) | IaC scanning, least-privilege IAM, network/egress defaults, encryption at rest |
| [security-testing.md](security-testing.md) | Required security-test coverage: nominal + abuse case, authz tests, API security tests |
| [tenant-isolation.md](tenant-isolation.md) | Per-tenant query filtering, cross-tenant test requirements, per-tenant key separation |
| [webhook-security.md](webhook-security.md) | Signing/verifying outbound and inbound webhooks, replay protection, treating third-party responses as untrusted |
| [backup-recovery.md](backup-recovery.md) | Safeguards on irreversible deletion, RPO/RTO documentation |
| [configuration-hardening.md](configuration-hardening.md) | Safe defaults, debug-mode discipline, technical-endpoint exposure, per-environment config separation |
| [client-security.md](client-security.md) | Browser/SPA-facing concerns: no client-side secrets, CSP, XSS prevention, local-storage discipline |
| [mobile-security.md](mobile-security.md) | Mobile-app concerns, where applicable: Keychain/Keystore storage, certificate pinning |
| [ai-llm-integration-security.md](ai-llm-integration-security.md) | Rules for a *feature* that itself integrates an LLM (prompt-injection resistance, tool allowlisting) — distinct from the AI-behavior rules above, which govern the coding assistant generating this codebase |

## How to use these documents

**For AI sessions (Claude Code / Codex):** load this `README.md` (always) plus every
domain file relevant to the task before generating code. Apply every applicable `MUST`
without exception; mark anything requiring a human decision
`SECURITY-REVIEW-REQUIRED: <reason>` rather than silently guessing.

**For user story authors:** reference the specific policy file and section a story must
satisfy (e.g. "per `token-standards.md` §Entropy") instead of re-describing the rule
inline. If a story seems to require violating a policy, that's a signal the policy needs
an ADR-backed exception, not that the story should quietly ignore it.

**For code reviewers:** a PR that implements a feature correctly but violates a security
policy in this folder must not be merged.

## End-of-generation self-check (AI self-review)

Before returning the code, the AI session runs this internal review and summarizes it
briefly:

- [ ] No applicable `MUST` directive, in any loaded domain file, is violated.
- [ ] No hardcoded secret; no home-grown crypto; no security control disabled.
- [ ] All external inputs are validated/typed server-side.
- [ ] DB access is parameterized; no query concatenation.
- [ ] Authorization verified server-side (and authz tests provided where relevant).
- [ ] No sensitive data logged.
- [ ] Dependencies: exact names, pinned versions, no invented package.
- [ ] Assumptions stated; points to validate marked `SECURITY-REVIEW-REQUIRED`.
- [ ] Security tests (nominal + abuse case) provided where relevant.

## Practical integration

**Human review**: this folder **does not replace** the Merge Request checklist. Generated
code remains subject to human review and CI/CD gates; the `SECURITY-REVIEW-REQUIRED`
comments serve there as priority points of attention.

## Policy change process

1. Open the change as a PR.
2. Write an ADR in `docs/adr/` recording why the policy changed.
3. Update every story/spec section the change affects.
4. Note the change in any already-delivered week's audit record that it retroactively
   affects (append a note; audit records are append-only).
