# ADR-0001: INT primary keys, not UUIDs

**Date:** {{SERVICE_INCEPTION_DATE}}
**Status:** Accepted
**Deciders:** Key Predict platform convention

## Context

Every Key Predict microservice sits in a closed ecosystem behind Kong: the Central Test
platform (V2 and V3 alike) uses `INT` auto-increment identifiers throughout, and
cross-service references already follow an `s_`-prefixed integer convention (e.g.
`s_odoo_partner_id`, `s_participant_id`). Introducing UUIDs in a new service would create
a second, incompatible ID convention at every integration boundary for no isolation
benefit — these are internal microservices behind a gateway, not public-facing systems
where ID unguessability matters.

## Decision

All primary keys are `INT` auto-increment. Cross-service references are plain `INT`
columns prefixed `s_` (e.g. `s_odoo_partner_id`), never a foreign key into another
service's database, and never a UUID.

- Laravel: Eloquent default (`$incrementing = true`, `$keyType = 'int'`) — do not override.
- FastAPI: SQLAlchemy `Integer` primary key with `autoincrement=True` — do not use `UUID`
  or `String` as a PK type.

## Consequences

- IDs are sequential and guessable — this is acceptable because every inbound request is
  already authenticated/authorized by Kong before it reaches this service; ID unguessability
  is not a security control here (see `docs/security/api-security.md` — authorization is
  enforced server-side, never via ID obscurity).
- Joins and indexes stay compact and fast.
- Any new cross-service reference field must follow the `s_` prefix convention and must
  not be declared as a foreign key (the referenced row lives in another service's database).
