# Skill: Write tests

Scope: the test coverage required for any story before it can be marked done. Applies on
top of `implement-feature.md` (tests are step 9 of that skill) and to fix stories.

Reference: `docs/stack-profiles/{{STACK}}.md` for exact commands/coverage flags, and
`docs/security/security-testing.md` for the security-test requirement.

## Responsibilities in mandatory order

1. **Identify the test tier(s) the change needs.**

   | Tier | Scope |
   |---|---|
   | Unit | Domain logic / a single class or function in isolation, no DB, no network |
   | Integration | A real (test) database, real ORM, one component boundary |
   | Contract | The API response actually matches `docs/specs/api/openapi.yaml` |
   | E2E | A full request through the real stack, closest to production behavior |

2. **Write the nominal case.** The nominal, expected-input, expected-output path.
3. **Write the abuse/edge cases the story's `## Error Cases` and the security policies
   require** — invalid input, unauthorized caller (both horizontal cross-tenant and
   vertical role-escalation attempts, per `docs/security/api-security.md`), boundary
   values, and any state-machine invalid-transition attempt if the entity has one.
4. **Write a regression test for anything the security policies specifically call out**
   — e.g. a test asserting a sensitive value never appears in a log line
   (`docs/security/logging-and-audit.md`).
5. **Name tests so intent is legible without reading the body**:
   `{{STACK: fastapi}}` — `test_<scenario>_<expected_result>` (e.g.
   `test_sync_dictionary_deactivates_existing_templates_when_new_active_received`).
   `{{STACK: laravel}}` — Pest's `it('does X when Y')` description strings, equally
   literal.
6. **Use the project's shared fixtures/factories**, don't hand-roll ad hoc test data per
   test file — `{{STACK: laravel}}` model factories, `{{STACK: fastapi}}`
   `tests/conftest.py` fixtures.
7. **Run the coverage gate.** ≥80% on the code the story touches, using the stack's exact
   command (see `docs/stack-profiles/{{STACK}}.md`) — never hand-wave "should be enough
   coverage," always run the flag that fails the build under the threshold.
8. **Never mock away the thing under test.** An integration test that mocks the database
   it's meant to be testing against isn't an integration test — use a real test database
   (SQLite/test-Postgres per stack convention), and reserve mocking for genuine external
   boundaries (a third-party HTTP API, an LLM call).

## What to test per layer (FastAPI hexagonal split — Laravel's service-layer equivalent
follows the same shape, one layer down)

- **Domain entities**: pure logic, no I/O — invariants, value objects, state-transition
  rules in isolation.
- **Use cases / services**: business logic with repository ports mocked/faked — this is
  where "what happens on the abuse case" gets exercised cheaply.
- **Repositories**: real database, real queries — this is where `s_` scoping and
  tenant-filtering (`docs/security/tenant-isolation.md`) get verified for real.
- **Routers/controllers**: thin — mostly covered by integration/contract tests exercising
  the full request/response cycle, since there's deliberately little logic here to unit
  test in isolation.
