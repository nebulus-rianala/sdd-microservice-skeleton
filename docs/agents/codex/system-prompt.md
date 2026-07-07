# Codex system prompt — {{SERVICE_NAME}}

Codex does not auto-load repository files. Paste this file's contents as the **system
prompt** for every Codex session on this project, and pass the relevant
`docs/agents/codex/task-templates/*.md` (filled in) as the **task body**. Never assume a
prior Codex session's context carries forward.

## Non-negotiable rules

- The spec wins over the story. `docs/specs/technical/{{SERVICE_NAME}}-technical-spec.md`
  is authoritative; if the task body seems to contradict it, stop and say so instead of
  silently picking one.
- No UUIDs — `INT` auto-increment primary keys only (ADR-0001).
- Explicit `{id}` lookup, never framework-magic model binding; never PII in a URL (ADR-0002).
- Audit/history tables have no FK to their subject and are never updated or deleted (ADR-0003).
- Every migration has both an up/forward and a down/backward operation (ADR-0004).
- No auth middleware — Kong owns authentication; trust `X-Consumer-Groups`/`X-Consumer-Id`
  without re-verifying them; `/internal/*` routes check `X-Internal-Api-Key` instead.
- Every external input is validated server-side by a strict schema
  (FormRequest/Pydantic model) with unknown fields rejected.
- Never invent a function, library, or package. Never write a hardcoded secret, even as an
  example. Never write home-grown cryptography.
- Mark anything you can't verify or that needs a human decision with
  `SECURITY-REVIEW-REQUIRED: <reason>` rather than guessing.
- Errors are RFC 9457 Problem Details; never leak a stack trace or internal exception text.
- `{{STACK}}`-specific rules in `docs/stack-profiles/{{STACK}}.md` are equally
  non-negotiable within that stack (e.g. `chunk()` not `cursor()` for Laravel; no business
  logic in a FastAPI router).

## Implementation order

domain/model layer → persistence layer (migration/repository) → business logic
(service/use case) → API surface (controller+FormRequest+Resource, or
router+Pydantic schema) → tests (unit, then integration). Never skip a layer; never
reorder it.

## Tech stack

`{{STACK}}` — see `docs/stack-profiles/{{STACK}}.md` for the full profile (versions,
quality-gate commands, ADRs).

## Full spec

`docs/specs/technical/{{SERVICE_NAME}}-technical-spec.md` — read the sections the task
body references before writing any code.
