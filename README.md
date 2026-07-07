# {{SERVICE_NAME}} — SDD Microservice Skeleton

This repository is a **clonable Specification-Driven Development (SDD) skeleton** for Key
Predict (Central Test) microservices. It is not a methodology description — it is the
methodology, encoded as Markdown that both a human and a coding agent (Claude Code, Codex)
read and obey.

Two stacks are supported: **PHP/Laravel** and **Python/FastAPI**. Database is **PostgreSQL
by default, MySQL as the alternative**. Every service sits behind **Kong API Gateway**: no
auth middleware inside the service, trusted `X-Consumer-*` headers, `/internal/*` routes
guarded by a static API-key header.

See `USAGE.md` for the full clone → spec → generate → implement runbook. This file covers
the document map and the SDD workflow diagram.

## SDD workflow

```
FUNCTIONAL SPEC (docs/specs/functional/, team-authored)
        │
        ▼  generator 1 — docs/prompts/01-generate-technical-spec.md
TECHNICAL SPEC (docs/specs/technical/, authoritative — spec wins over story)
        │
        ▼  generator 2 — docs/prompts/02-generate-planning.md
DELIVERY PLAN (docs/planning/, weekly batches)
        │
        ▼  generator 3 — docs/prompts/03-generate-user-stories.md
USER STORIES (docs/user-stories/features/week-NN/, one file per unit of work)
        │
        ▼  implementation loop
Claude Code validates story against spec+ADRs+security
        │
        ▼
Codex generates code + tests from the validated contract
        │
        ▼
Claude Code reviews generated output against spec + quality gates
        │
        ▼
AUDIT (docs/audit/, append-only acceptance record)
```

Every AI session in this loop obeys the same guardrails: **`CLAUDE.md`** (root), the
security policies in `docs/security/`, and the ADRs in `docs/adr/`. Claude Code validates
and reviews; Codex generates; both read the same Markdown. **The spec always wins over the
story** — if a plan or a story ever contradicts the current technical spec, fix the story,
not the spec.

## Document map

| Folder | Answers |
|---|---|
| `docs/specs/functional/` | What the product needs (team-authored input; generator 1 reads this) |
| `docs/specs/technical/` | What to build — authoritative. *Do not edit to match code; fix the code to match this spec.* |
| `docs/specs/api/` | The OpenAPI contract every endpoint must match |
| `docs/specs/data-contracts/` | Cross-service JSON Schemas for integration payloads |
| `docs/adr/` | What technical decision, why, immutable once accepted — superseded by a new ADR, never edited |
| `docs/security/` | Guardrails loaded in every AI session. Not deliverables — stories reference these, never redefine them |
| `docs/skills/` | Domain/task knowledge injected into an AI session for a specific kind of work (writing a migration, generating the technical spec, etc.) |
| `docs/planning/` | In what order, in what weekly batches — a sequenced delivery backlog. Not a specification: if it contradicts the spec, the spec wins |
| `docs/user-stories/` | What unit of work, in what week — one file per independently-implementable story |
| `docs/agents/` | Agent-specific entry points: Claude Code overrides, Codex system prompt + task templates |
| `docs/prompts/` | Thin runnable wrappers that invoke the three generator skills |
| `docs/stack-profiles/` | Everything true for exactly one stack (Laravel or FastAPI), never the shared core |
| `docs/context/` | Background/legacy reference. Not authoritative — read-only |
| `docs/architecture/` | Living system-design documents (domain model, state machine, queue topology) — unlike ADRs, these evolve |
| `docs/audit/` | Append-only acceptance records, one per delivered weekly batch |
| `docs/runbooks/` | Operational procedures for production |

## Quality gates (must pass before merging any MR)

See `docs/stack-profiles/{{STACK}}.md` for the exact commands. Both stacks share the same
bar: **zero style violations, zero static-analysis errors, ≥80% test coverage.**

## Key conventions for AI sessions

Every Claude Code and Codex session must load, in order:
1. `CLAUDE.md` (project root) — conventions, stack, absolute rules
2. `docs/specs/technical/{{SERVICE_NAME}}-technical-spec.md` — authoritative specification
3. `docs/security/README.md` (always-loaded meta rules) + the relevant `docs/security/*` domain policies for the task
4. `docs/adr/README.md` — decisions already made, not to be re-litigated
5. The relevant skill file(s) for the current task under `docs/skills/`

`AGENTS.md` restates this load order for agent-agnostic / Codex sessions.
