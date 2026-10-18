# SDD Microservice Skeleton

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

A clonable **Specification-Driven Development (SDD)** starting point for building
microservices with AI coding agents (Claude Code, Codex, or any agent that reads Markdown).

This repository contains no application code. It holds the rules, templates, and generator
prompts that turn a functional spec into a technical spec, then a delivery plan, then user
stories, and finally reviewed, tested code. All of it is plain Markdown that a human and a
coding agent read and follow in the same way.

## Purpose

AI coding agents write code quickly, but without guardrails they drift. They invent
conventions, re-argue settled decisions, skip migrations' rollback paths, leak stack traces
in error responses, and forget tenant isolation. Each new service ends up with its own
slightly different set of rules.

This skeleton fixes that by making the methodology **part of the repository**:

- **One authoritative spec per service.** It is generated through an interview that brings
  unstated assumptions to the surface. When a story or plan contradicts the spec, the spec
  wins.
- **Decisions are written down once.** ADRs (`docs/adr/`) record choices such as integer
  primary keys, explicit ID lookups, and reversible migrations, so agents apply them
  instead of reopening them.
- **Security guardrails are loaded in every session.** `docs/security/` covers API
  security, tenant isolation, GDPR, secrets, logging, and more. Stories reference these
  policies and never redefine them.
- **A two-agent loop with review.** One agent validates and reviews, the other generates.
  Every deviation is flagged and given a disposition, and accepted work is recorded in an
  append-only audit trail.
- **Quality gates are non-negotiable.** Zero style violations, zero static-analysis
  errors at the strictest level, and at least 80% test coverage.

The conventions were extracted from real production microservices, not invented from
scratch. `docs/context/skeleton-bootstrap-assumptions.md` lists the few places where a
decision had to be made without a reference.

## Who it is for

Teams that build several small HTTP microservices and want every one of them to follow the
same architecture, security baseline, and delivery process, whoever (or whichever agent)
writes the code.

It assumes the following architecture. Adapt the ADRs and `CLAUDE.md` if yours differs:

- **Stacks:** PHP 8.3 / Laravel 11, or Python 3.12 / FastAPI.
- **Database:** PostgreSQL by default, MySQL as the alternative.
- **API gateway:** every service sits behind [Kong](https://konghq.com/), which owns
  authentication, CORS, rate limiting, and TLS. Services have no auth middleware, trust the
  `X-Consumer-*` headers that Kong forwards, and guard `/internal/*` routes with a static
  API-key header.
- **Errors:** RFC 9457 Problem Details.

## Quickstart

```bash
git clone https://github.com/nebulus-rianala/sdd-microservice-skeleton.git my-service
cd my-service
```

1. Fill in the placeholders you already know in `CLAUDE.md` (`{{SERVICE_NAME}}`,
   `{{STACK}}`, `{{DB}}`, ...).
2. Put your functional spec in `docs/specs/functional/`.
3. Run the three generator prompts in `docs/prompts/` in order.
4. Implement story by story with the validate, generate, review loop.

`USAGE.md` is the full step-by-step runbook. Once you have cloned the skeleton for a real
service, replace this README's introduction with a description of that service.

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

## Quality gates (must pass before merging any pull request)

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

## Contributing

Contributions are welcome. See [CONTRIBUTING.md](CONTRIBUTING.md).

## License

Released under the [MIT License](LICENSE).
