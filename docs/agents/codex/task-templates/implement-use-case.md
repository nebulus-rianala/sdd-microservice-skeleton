# Codex task: implement a use case / service

## Context

- Spec: `docs/specs/technical/{{SERVICE_NAME}}-technical-spec.md` §[FILL: Domain Model &
  Business Rules section]
- Story: `docs/user-stories/features/week-{{WEEK}}/{{ID_PREFIX}}-[FILL: story id].md`
- Skill: `docs/skills/implement-feature.md` (read before starting)

## Task

Implement `[FILL: use case / service class name]`. This is a **focused, single business
operation** task — the class this task creates is called by a router/controller that
already exists or is being implemented in a separate task; do not add API-layer code here.

- [FILL: 1-3 bullets naming the exact method signature(s) and what each does]
- `{{STACK: fastapi}}`: depends only on domain types and repository ports passed in —
  never a concrete SQLAlchemy/infrastructure type.
- `{{STACK: laravel}}`: the service class the controller delegates to; no validation
  logic here (that's the FormRequest's job) — only business logic.

## Constraints

- Follow `docs/stack-profiles/{{STACK}}.md` conventions exactly.
- Any side effect (queue job, pub/sub event, notification) is dispatched from here with
  primitive IDs only — never pass a model/entity instance into async work.
- If the entity has a state machine, transitions go through it — never set a status field
  directly.
- Raise domain-level exceptions here; translating them to HTTP status codes is the API
  layer's job, not this one's.

## Business rules to enforce

- [FILL: from the story's `## Business Rules` section]

## Error cases to handle

| Condition | Domain exception raised |
|---|---|
| [FILL] | [FILL] |

## Tests required

- Unit: [FILL] (repository ports faked/mocked — this is the cheap layer to exercise abuse
  cases in)
