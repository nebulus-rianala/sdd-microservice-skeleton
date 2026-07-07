# Architecture

System-level design documents: domain model, state-machine diagram, queue topology,
deployment topology, notification/session flows. They are the reference point for
onboarding new contributors, validating that a proposed change fits the overall design,
and Claude Code sessions that need end-to-end context before generating cross-cutting
code.

Unlike ADRs (`docs/adr/`), which record point-in-time decisions and are immutable,
architecture documents are **living documents** — they evolve as the system grows. Update
them in place; don't version them like an ADR.

## Suggested starting set

| File | Status |
|---|---|
| `domain-model.md` | Planned |
| `state-machine.md` | Planned |
| `queue-topology.md` | Planned |
| `deployment-topology.md` | Planned |

## Key design principles

Cross-reference the ADR that established each principle so a reader can see why, not just
what:

- INT primary keys, no UUIDs (ADR-0001)
- Explicit `{id}` lookup, no PII in URLs (ADR-0002)
- Audit logs survive hard-delete (ADR-0003)
- Every migration is reversible (ADR-0004)
