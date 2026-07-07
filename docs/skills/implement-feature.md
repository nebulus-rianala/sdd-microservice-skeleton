# Skill: Implement a feature

Scope: build one vertical slice of functionality (one endpoint or one small group of
tightly related endpoints) end to end, from domain layer to tests, for a single user
story. This is the master playbook both Claude Code (validating) and Codex (generating)
follow for ordinary feature work — one-off migrations, test-writing, and technical-spec
generation each have their own narrower skill file.

Reference spec: `docs/specs/technical/{{SERVICE_NAME}}-technical-spec.md` — read the
relevant `§Domain Model`/`§API Reference` sections for the entity/endpoint in question
before starting.

## Responsibilities in mandatory order

1. **Read first.** The story file, its `## Spec references`, `CLAUDE.md`, and the
   security policies the story names. Do not write code before this step is complete.
2. **Domain layer.**
   - `{{STACK: laravel}}` — the Eloquent model (or a plain value object for
     non-persisted domain concepts) plus any backed enum for state/type/status fields.
   - `{{STACK: fastapi}}` — a framework-free domain entity (dataclass) and any domain
     exceptions, with zero I/O and zero framework imports.
3. **Persistence contract.**
   - `{{STACK: laravel}}` — the migration (see `write-migration.md`), factory, and any
     model relationships/casts/scopes needed.
   - `{{STACK: fastapi}}` — a repository port (an abstract interface in the application
     layer) before any concrete implementation exists.
4. **Business logic.**
   - `{{STACK: laravel}}` — a service class implementing the use case; the controller
     only validates (via a FormRequest) and delegates to it. No business logic in the
     controller.
   - `{{STACK: fastapi}}` — a use-case class in the application layer that depends only
     on domain types and repository ports — never a concrete ORM/infrastructure type.
5. **Infrastructure implementation** (FastAPI only — Laravel's Eloquent model already is
   the persistence implementation). Implement the repository port against SQLAlchemy,
   with an explicit `_to_domain` mapper — never return a raw ORM model out of the
   repository.
6. **API surface.**
   - `{{STACK: laravel}}` — FormRequest (input validation), API Resource (output shape,
     using `whenLoaded()` to avoid N+1), controller method, route registration.
   - `{{STACK: fastapi}}` — Pydantic request/response schemas, a thin router function
     that only calls the use case and translates its result/exceptions to an HTTP
     response — no business logic in the router.
7. **Match the OpenAPI contract.** The request/response shape must match
   `docs/specs/api/openapi.yaml` exactly. If the story requires a contract change, update
   the YAML in the same change — do not let code and contract drift.
8. **Errors.** Every failure mode listed in the story's `## Error Cases` maps to the RFC
   9457 Problem Details format (see `CLAUDE.md §Error Response Format`) with the correct
   HTTP status.
9. **Tests.** See `write-tests.md` — write them alongside the implementation, not after,
   and include the security-relevant cases (authz, invalid input) SDD-12 requires.
10. **Quality gates.** Run the full gate set for the chosen stack (see
    `docs/stack-profiles/{{STACK}}.md`) before considering the story done. Never skip a
    gate to save time.

## Side effects and edge cases

- Side effects (queue jobs, pub/sub events, notifications) are dispatched from the
  service/use-case layer, never the controller/router, and carry primitive IDs only —
  never a model/entity instance — into async work (see `laravel-ADR-002` /
  the general "don't serialize stale domain state into async work" principle it encodes).
- If the entity has a state machine, every transition goes through it — never mutate a
  status field directly, even for what looks like a trivial internal update.
- If the endpoint returns a collection, it is paginated — never an unbounded list.
