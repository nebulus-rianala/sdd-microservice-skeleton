# AGENTS.md — Agent-Agnostic Entry Point

This file is for any coding agent that does not auto-load `CLAUDE.md` (Codex, Cursor,
Cline, or the raw Anthropic/OpenAI API). Claude Code auto-loads `CLAUDE.md` from the repo
root; if you are driving a Claude Code session, read that file instead — it carries the
same rules with a Claude-Code-specific framing (skills table, git workflow, etc.).

## Mandatory load order (every session, every agent)

Do not generate or modify code before reading, in this order:

1. `CLAUDE.md` — stack, absolute rules, project structure, domain model, API surface
2. `docs/specs/technical/{{SERVICE_NAME}}-technical-spec.md` — the authoritative spec
3. `docs/security/README.md` (always-loaded meta rules and level convention) + any
   `docs/security/*.md` domain policy relevant to the task (e.g. touching input →
   `input-validation.md`; touching a token → `token-standards.md`)
4. `docs/adr/README.md` — decisions already made; do not re-litigate, do not violate
5. The specific skill file(s) for the task at hand, under `docs/skills/`
6. The story or fix file being implemented, under `docs/user-stories/`

## The one rule that overrides all others

**The spec wins over the story.** If a user story, a plan, or a prior implementation
contradicts the current technical spec, the spec is correct and the other artifact is
stale. Flag the contradiction (in the story's `## Notes` or a new fix story), do not
silently resolve it by changing the spec to match code, and do not implement the
contradicting instruction.

## Claude Code vs. Codex — who does what

| Task | Agent |
|---|---|
| Full feature implementation (domain → API) | Claude Code |
| Migration authoring | Claude Code |
| Multi-file refactoring | Claude Code |
| Spec/story validation, review of generated output against gates | Claude Code |
| Focused endpoint or schema generation | Codex |
| Single-function fix | Codex |

**Claude Code validates and reviews. Codex generates.** Both read the same Markdown —
there is no Codex-only or Claude-Code-only source of truth for what to build, only for how
each is briefed. Codex has no auto-load mechanism: every Codex session must be given
`docs/agents/codex/system-prompt.md` as its system prompt and the relevant
`docs/agents/codex/task-templates/*.md` (filled in) as its task body.

## Implementation loop

1. Claude Code reads the story, the spec sections it references, the relevant skill(s),
   and the security policies in scope. It challenges assumptions, flags missing edge
   cases, and produces a concrete implementation contract (exact files, signatures,
   test list).
2. Codex generates production code and tests from that contract.
3. Claude Code reviews the generated output against the spec, the ADRs, the security
   policies, and the stack's quality gates (see `docs/stack-profiles/{{STACK}}.md`).
   Deviations are marked `[DEVIATION]` with a disposition (acceptable / fix required) —
   never silently merged.
4. Acceptance is recorded in `docs/audit/` once all gates pass.
