# Stack Profile: Laravel

Applies when `{{STACK}} = laravel`. Everything in this file is absolute within a Laravel
service; it does not apply to a FastAPI service (see `fastapi.md` for that profile's
equivalents), and the shared core (`CLAUDE.md`, `docs/adr/`, `docs/security/`) still
applies underneath it.

## Technology stack

| Concern | Choice |
|---|---|
| Language | PHP 8.3, `declare(strict_types=1);` at the top of every file |
| Framework | Laravel 11 |
| ORM | Eloquent |
| Database | PostgreSQL 16 (default) or MySQL (alternative) — `{{DB}}` |
| Queue | Horizon, Redis-backed |
| Tests | Pest |
| Static analysis | PHPStan, `level: 8`, zero errors |
| Style | Pint (`preset: laravel`, `declare_strict_types: true`) |
| API docs | Scramble (generated from route/FormRequest/Resource annotations) |
| File storage | Flysystem (GCS in production) |

## Hard constraints (Laravel-specific, in addition to CLAUDE.md's Absolute Rules)

- `declare(strict_types=1);` in every PHP file.
- Enums over constants: all state/type/channel/status values are PHP 8.1+ backed enums.
- No raw SQL outside an aggregate/statistics service, and even there only with bound
  parameters. Migrations may use `DB::statement()` for `CHECK` constraints, wrapped in
  `if (DB::connection()->getDriverName() !== 'sqlite')` to keep the SQLite test suite
  working.
- Single responsibility: controllers only validate (via a FormRequest) and delegate;
  business logic lives in a service class.
- No `SoftDeletes` trait on any model — soft-delete is explicit `active`/`archived_at` +
  state machine (see `docs/security/immutability-rules.md`).
- Side effects (lifecycle events) are published via a dedicated job + pub/sub service call
  — not Laravel's Event/Listener system.
- Use `chunk()`, never `cursor()`, for any command that writes inside its own iteration
  loop (ADR-001 below).
- Job constructors accept only primitive IDs/enums/scalars, never an Eloquent model
  instance (ADR-002 below).

## Laravel-only ADRs

### ADR-001: chunk() over cursor() for batch write operations

**Status:** Accepted

**Context:** `cursor()` keeps a streaming database cursor open across the iteration; a
write inside that same loop can conflict with the open read cursor, especially for
larger batches or bulk write commands.

**Decision:** Use `chunk(100)` (or a chunk size stated in the technical spec) instead of
`cursor()` for any Artisan command or job that writes to the database inside its
iteration loop.

**Consequences:** Slightly higher memory/query overhead per chunk boundary, in exchange
for eliminating a class of intermittent, hard-to-reproduce write-during-read-cursor bugs.

### ADR-002: Job constructors carry primitive IDs

**Status:** Accepted

**Context:** Passing an Eloquent model instance into a queued job's constructor
serializes it (`SerializesModels`) at dispatch time. By the time the job actually runs
(seconds to hours later), that serialized snapshot can be stale relative to the database.

**Decision:** Queued job constructors accept only `int`/enum/scalar arguments — never an
Eloquent model instance. The job refetches the current row from the database inside
`handle()`.

**Consequences:** The job always operates on current data, never a stale snapshot; the
job's serialized payload (visible in the queue driver) never carries a full model's
column data.

### ADR-003: failed() hook, not try/catch, for job status

**Status:** Accepted

**Context:** A job that fails is usually retried by Horizon per its configured retry
policy. Marking a status field "failed" from inside a `try/catch` in `handle()` would mark
it failed after the *first* attempt, even though retries are about to happen.

**Decision:** Use Laravel's `failed(Throwable $exception)` hook — invoked only after all
retries are exhausted — to write a terminal failure status. Do not wrap `handle()` in a
try/catch for this purpose.

**Consequences:** A status field only reads "failed" once the retry budget is truly
exhausted, not on a transient first-attempt error the retry mechanism was about to recover
from.

### ADR-004: Laravel 11 scheduler via withSchedule()

**Status:** Accepted

**Context:** Laravel 11 moved scheduled-command registration out of `app/Console/Kernel.php`
(which no longer exists in a fresh Laravel 11 app) into the application bootstrap file.

**Decision:** Register every scheduled command via `->withSchedule()` in
`bootstrap/app.php`. Do not create a `Kernel.php` to hold scheduling.

**Consequences:** Scheduling configuration lives in one place consistent with Laravel 11's
own bootstrap convention; there is no `Kernel.php` to keep in sync with it.

## Quality gates

```bash
./vendor/bin/pint --test              # zero style violations
./vendor/bin/phpstan analyse          # zero PHPStan level 8 errors
./vendor/bin/pest --coverage --min=80 # ≥80% coverage, all green
```

## Common Artisan commands

```bash
php artisan make:migration <description>
php artisan make:model <Name> -mf          # model + migration + factory
php artisan migrate
php artisan migrate:rollback
php artisan test                            # or ./vendor/bin/pest
php artisan schedule:list
```

## .env.example keys (starting set — extend per service)

```
APP_NAME={{SERVICE_NAME}}
APP_ENV=local
APP_KEY=
APP_URL=http://localhost

DB_CONNECTION={{DB}}
DB_HOST=127.0.0.1
DB_PORT={{DB_PORT}}
DB_DATABASE={{SERVICE_NAME}}
DB_USERNAME=
DB_PASSWORD=

REDIS_HOST=127.0.0.1
REDIS_PORT=6379
QUEUE_CONNECTION=redis

GDPR_RETENTION_DAYS=730

# Per-integration outbound API key pattern (see docs/security/api-security.md)
{{INTEGRATION}}_API_KEY_NAME=
{{INTEGRATION}}_API_KEY=

GATEWAY_API_KEY_NAME=X-Api-Key
GATEWAY_API_KEY=
```
