> **Read this file first, every session.** It is auto-loaded by Claude Code. If you are
> Codex or another agent, read `AGENTS.md` and `docs/agents/codex/system-prompt.md`
> instead — same rules, different framing.

# {{SERVICE_NAME}} — Claude Code Project Instructions

## Overview

{{SERVICE_NAME}} is a Kong-fronted microservice built from the SDD Microservice Skeleton. {{ONE_PARAGRAPH_SERVICE_DESCRIPTION}}

Before implementing anything, read `docs/specs/technical/{{SERVICE_NAME}}-technical-spec.md`
in full — it is the authoritative specification. This file (`CLAUDE.md`) covers
conventions, structure, and absolute rules; it does not replace the spec.

## Technology Stack

<!-- DECISION: this table is filled once {{STACK}} is chosen during generator 1's interview
     (see docs/skills/generate-technical-spec.md Phase B). Delete the row that doesn't apply. -->

| Concern | `laravel` | `fastapi` |
|---|---|---|
| Language | PHP 8.3, `declare(strict_types=1)` everywhere | Python 3.12, full type annotations everywhere |
| Framework | Laravel 11 | FastAPI |
| ORM / DB layer | Eloquent | SQLAlchemy (async) |
| Migrations | Laravel migrations (up + down) | Alembic (`upgrade()` + `downgrade()`) |
| Database | {{DB}} ({{DB}} = `postgres` default, `mysql` alternative) | {{DB}} |
| Queue / async jobs | Horizon (Redis-backed) | ARQ (Async Redis Queue) |
| Test runner | Pest | pytest (unit / integration / contract / e2e) |
| Static analysis | PHPStan level 8 (zero errors) | mypy `strict = true` (zero errors) |
| Style | Pint (zero violations) | ruff (zero violations) |
| API docs | Scramble (generated from code) | OpenAPI 3.1 (`docs/specs/api/openapi.yaml`, hand-maintained contract) |
| Architecture | Controller → FormRequest → Service → Model, single-responsibility controllers | Hexagonal: domain → application → infrastructure → api, strict layer separation |

Full stack conventions for the chosen stack live in `docs/stack-profiles/{{STACK}}.md` —
read it before writing any code.

## Absolute Rules

These are non-negotiable. A generated change that violates one of these must not be
merged, regardless of what a story or plan says.

- **No UUIDs.** All primary keys are `INT` auto-increment (see ADR-0001). Cross-service
  references use the `s_` prefix convention (e.g. `s_crm_partner_id`) and are plain
  `INT`, never a foreign key to another service's database.
- **No auth middleware.** Kong API Gateway owns authentication, CORS, rate limiting, and
  TLS termination. This service trusts Kong-forwarded headers (`X-Consumer-Groups`,
  `X-Consumer-Id`) without re-verifying them inside the service boundary. `/internal/*`
  routes are the one exception: they are protected by a static `X-Internal-Api-Key`
  header, checked against an env-provided value, because they are called service-to-service
  and never cross Kong.
- **Explicit ID lookup, never framework-magic binding, never PII in a URL** (see
  ADR-0002). Look up the entity by its raw `{id}` path/route parameter explicitly and
  handle "not found" yourself; do not rely on implicit model-binding or ORM auto-resolution
  that hides the lookup and its failure mode.
- **Every migration has both directions.** No migration may ship without a working
  down/backward operation (see ADR-0004).
- **Audit logs have no FK to their subject and survive hard-delete** (see ADR-0003).
  GDPR erasure of a parent entity must never cascade-delete its audit trail.
- **DATETIME, not TIMESTAMPTZ.** Schema stays database-portable between `{{DB}}` choices.
- **JSON/JSONB for structured columns**, never a serialized string column.
- **Errors are RFC 9457 Problem Details.** Never leak a stack trace, a raw exception
  message, an internal model name, or an ORM/driver-level error to a client response.
- **Deny by default, fail closed.** Every new endpoint, flag, and integration defaults to
  the most restrictive behavior until explicitly widened.
- **`{{STACK}}`-specific hard rules** (chunk() not cursor(), no business logic in a
  router/controller, job constructors carry primitive IDs not model instances, etc.) live
  in `docs/stack-profiles/{{STACK}}.md` — they are absolute within that stack, but are not
  restated here because they don't apply to the other stack.

## References

- Authoritative spec: `docs/specs/technical/{{SERVICE_NAME}}-technical-spec.md`
- Architecture decisions: `docs/adr/README.md`
- Domain/task skills: `docs/skills/README.md`
- Security guardrails (loaded every session): `docs/security/README.md`
- Stack conventions: `docs/stack-profiles/{{STACK}}.md`

## Project Structure

<!-- DECISION: filled by generator 1 once the stack and service identity are known.
     Keep this tree in sync with the real repo — a stale tree here is worse than no tree. -->

```
{{PROJECT_STRUCTURE_TREE}}
```

## Domain Model

<!-- DECISION: filled by generator 1 from the functional spec's domain model section.
     ASCII entity tree + a mapping table if this service migrates data from a legacy system. -->

{{DOMAIN_MODEL_TREE}}

## State Machine

<!-- DECISION: only present if {{DOMAIN_ROOT_ENTITY}} has a lifecycle. Delete this section
     if the service has no stateful root entity. -->

{{STATE_MACHINE_DIAGRAM}}

Every transition is enforced server-side; an invalid transition raises a domain exception
mapped to HTTP 422, and every transition writes an immutable history row.

## API Base URL & Endpoints

**Base URL:** `{{API_BASE}}` (e.g. `https://api.example.com/{{SERVICE_NAME}}/v1`)

| Method | Path | Purpose | Caller |
|---|---|---|---|
| {{METHOD}} | {{PATH}} | {{PURPOSE}} | {{CALLER}} |

Full contract: `docs/specs/api/openapi.yaml`. The spec's `§API Reference` /
`§API Specification` section is authoritative if the two ever diverge — fix the YAML.

## Auth model (Kong)

| Caller type | Header | Access |
|---|---|---|
| {{ROLE}} | `X-Consumer-Groups: {{ROLE_VALUE}}` | {{SCOPE_DESCRIPTION}} |
| Internal service | `X-Internal-Api-Key` (env `INTERNAL_API_KEY`) | `/internal/*` routes only |

`X-Consumer-Id` carries the caller's company/tenant scope (typically `s_crm_partner_id`
in this ecosystem) — every query must be filtered by it unless the caller's group grants
cross-tenant access. These headers are trusted without verification: Kong guarantees they
cannot be forged by an external caller. Do not implement header signature verification,
CORS, rate limiting, or TLS termination in this service — all of that is Kong's job.

## Error Response Format

All error responses are RFC 9457 Problem Details (`application/problem+json`):

```json
{
  "type": "https://api.example.com/problems/{{PROBLEM_SLUG}}",
  "title": "{{HUMAN_READABLE_TITLE}}",
  "status": 422,
  "detail": "{{SPECIFIC_DETAIL_SAFE_FOR_CLIENT}}",
  "instance": "{{API_BASE}}/{{RESOURCE}}/{{ID}}"
}
```

| Status | Meaning |
|---|---|
| 400 | Malformed request |
| 401 | Missing/invalid `X-Internal-Api-Key` on an `/internal/*` route |
| 403 | Authenticated (by Kong) but not authorized for this resource/tenant |
| 404 | Resource not found (after an explicit lookup, never implicit) |
| 409 | Conflict with current state (e.g. invalid state-machine transition attempted through a non-transition endpoint) |
| 422 | Validation failure or invalid state-machine transition |
| 429 | Rate limited (enforced by Kong, not this service) |

## Quality gates

See `docs/stack-profiles/{{STACK}}.md` for exact commands. Both stacks share the same bar:

- **Style**: zero violations
- **Static analysis**: zero errors at the stack's strict setting (PHPStan level 8 / mypy strict)
- **Tests**: ≥80% coverage, all green

## Git workflow

- Gitflow-style branching: `feature/{{ID_PREFIX}}-{ticket}-{slug}`, `fix/{{ID_PREFIX}}-{ticket}-{slug}`
- CI must pass style + static analysis + tests before merge; no `--no-verify`, no skipped gates
- Every migration in a PR must be reversible (`down`/`downgrade()` present and tested)

## Items Pending Clarification

<!-- DECISION: living ledger. Generator 1 appends here whenever the functional spec leaves
     something ambiguous; never silently resolve an ambiguity by picking an answer and
     deleting the question. -->

- {{OPEN_QUESTION}}
