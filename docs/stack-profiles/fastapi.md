# Stack Profile: FastAPI

Applies when `{{STACK}} = fastapi`. Everything in this file is absolute within a FastAPI
service; it does not apply to a Laravel service (see `laravel.md` for that profile's
equivalents), and the shared core (`CLAUDE.md`, `docs/adr/`, `docs/security/`) still
applies underneath it.

## Technology stack

| Concern | Choice |
|---|---|
| Language | Python 3.12, full type annotations everywhere |
| Framework | FastAPI |
| ORM | SQLAlchemy (async) |
| Database | PostgreSQL 16 (default) or MySQL (alternative) — `{{DB}}` |
| Migrations | Alembic — always `upgrade()` + `downgrade()` |
| Async jobs | ARQ (Async Redis Queue) |
| Config | `pydantic-settings` (`app/core/config.py`) |
| Tests | pytest, tiered: unit / integration / contract / e2e |
| Static analysis | mypy, `strict = true`, zero errors |
| Style | ruff (lint + format), zero violations |
| API docs | OpenAPI 3.1 contract, hand-maintained at `docs/specs/api/openapi.yaml` |

## Architecture — hexagonal (ports & adapters)

Strict layer separation, dependency direction `api → application → domain`, with
infrastructure implementing ports defined by the application layer. **No layer may import
from a layer above it.**

- **Domain layer** (`app/domain/`) — pure Python. No framework imports, no I/O.
- **Application layer** (`app/application/`) — use cases + pipeline orchestration.
  Depends on domain + repository ports only, never a concrete ORM/infra type.
- **Infrastructure layer** (`app/infrastructure/`) — implements the ports: SQLAlchemy
  repositories, Redis/ARQ, the LLM client, external HTTP clients.
- **API layer** (`app/api/`) — FastAPI routers (+ MCP tools, if the service exposes any).
  Depends on the application layer only; contains no business logic.

Build/implementation order for any new feature (see `docs/skills/implement-feature.md`):
domain entity/enum → repository port → use case → SQLAlchemy model → repository
implementation → Alembic migration → Pydantic schema → FastAPI router → unit tests →
integration tests. Never skip a step; never reorder it.

## Hard constraints (FastAPI-specific, in addition to CLAUDE.md's Absolute Rules)

- No business logic in a router — a router calls a use case and translates its
  result/exception to an HTTP response, nothing else.
- No direct database access from a use case — only through a repository port.
- Everything is `async def`; no blocking I/O (use `asyncio.to_thread` when a blocking
  call is unavoidable); no `time.sleep` (`asyncio.sleep`); no `requests` (use
  `httpx.AsyncClient`).
- Full type annotations on every function; no bare `except:`; no `print()` (use the
  configured logger).
- No hardcoded config — everything comes from `pydantic-settings`, never a scattered
  `os.environ` call.
- `JSONB`, not `JSON`, for structured columns.
- No manual timestamp management — `server_default=func.now()` /
  `onupdate=func.now()`.
- Every migration has both `upgrade()` and `downgrade()` (ADR-0004).
- `/internal/*` routes are protected by an `X-Internal-Api-Key` header dependency, checked
  against the `INTERNAL_API_KEY` setting — a mismatch raises `HTTPException(401)`.

## FastAPI-only ADRs

### ADR-001: ARQ over Celery

**Status:** Accepted

**Context:** FastAPI/SQLAlchemy in this stack are async-native. Celery's default worker
model is synchronous and needs extra configuration (and a separate result backend) to work
well with an async application; the operational surface area of running Celery well is
larger than this service needs.

**Decision:** Use ARQ (Async Redis Queue) for all background/async task processing.

**Consequences:** Redis is a required dependency for the queue itself; job state in Redis
is treated as ephemeral — durable job status is tracked in Postgres (e.g. a `status`
column on the owning entity), not relied upon from Redis alone. The worker runs as its own
process/Docker image and Kubernetes deployment, separate from the API.

### ADR-002: Hexagonal architecture

**Status:** Accepted

**Context:** Domain and application logic need to be unit-testable without a database or
an LLM client running; infrastructure choices (which LLM provider, which DB) need to be
swappable behind a stable interface without touching business logic.

**Decision:** Use hexagonal (ports & adapters) architecture with strict layer separation:
`domain → application → infrastructure → api`. See "Architecture" above for the layer
definitions. `docs/skills/implement-feature.md` enforces this build order for every
coding agent.

**Consequences:** More files/boilerplate per feature (an entity, a port, a use case, a
model, a repository implementation, a schema, a router — at minimum) in exchange for
domain/application code that's testable with zero infrastructure dependencies, and
infrastructure (DB, LLM, PDF generation, etc.) that's swappable via the adapter alone.

## Quality gates

```bash
uv run ruff check app/ tests/
uv run mypy app/
uv run pytest --cov=app --cov-report=term-missing --cov-fail-under=80
```

## Test tiers

```bash
uv run pytest tests/unit/          # domain + application logic, no I/O
uv run pytest tests/integration/   # real test Postgres + Redis
uv run pytest tests/contract/      # response shape vs. docs/specs/api/openapi.yaml
uv run pytest                      # everything, incl. e2e if present
```

## Alembic

```bash
uv run alembic revision --autogenerate -m "<description>"
uv run alembic upgrade head
uv run alembic downgrade -1
```

Never hand-author a revision from scratch; never edit a revision already applied to a
shared environment — write a new one instead.

## .env.example keys (starting set — extend per service)

```
APP_NAME={{SERVICE_NAME}}
ENVIRONMENT=local

DATABASE_URL=postgresql+asyncpg://user:pass@localhost:5432/{{SERVICE_NAME}}
REDIS_URL=redis://localhost:6379/0

INTERNAL_API_KEY=

GDPR_RETENTION_DAYS=730

# Outbound integration credentials (per external service this project calls)
{{INTEGRATION}}_API_KEY=
{{INTEGRATION}}_BASE_URL=
```
