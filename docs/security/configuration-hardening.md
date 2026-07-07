# Configuration and Hardening

Ref: OWASP A05.

## Safe defaults

Generated configuration defaults to secure behavior — deny by default, fail closed — the
same principle `README.md`'s AI behavior rules state for code, applied to configuration
values specifically.

## Debug mode

Debug mode is disabled in production by construction (environment-driven, not a
manually-flipped flag someone can forget). Error pages/responses in production are
neutral — see `api-security.md`'s error-response rule; a stack trace is a debug-mode leak
specifically.

## Technical endpoints

Health checks, metrics, framework admin panels (Laravel Telescope/Horizon dashboard,
FastAPI's auto-generated docs UI), and any actuator-style endpoint are protected or
disabled in production — never exposed unauthenticated on the public route surface.

## Per-environment configuration

Configuration is separated by environment (local/staging/production) through the
framework's standard config mechanism (Laravel `config/` + `.env` per environment,
FastAPI `pydantic-settings` profiles) — never a single shared config file with
environment checks sprinkled through business logic.
