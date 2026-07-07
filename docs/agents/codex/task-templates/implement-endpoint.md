# Codex task: implement an endpoint

## Context

- Spec: `docs/specs/technical/{{SERVICE_NAME}}-technical-spec.md` §[FILL: section]
- OpenAPI contract: `docs/specs/api/openapi.yaml` — operation `[FILL: method + path]`
- Story: `docs/user-stories/features/week-{{WEEK}}/{{ID_PREFIX}}-[FILL: story id].md`
- Skill: `docs/skills/implement-feature.md` (read before starting)

## Task

Implement `[FILL: METHOD /path]` exactly as specified above. This is a **focused, single
endpoint** task — do not touch unrelated files, do not refactor adjacent code, do not
change the OpenAPI contract unless the story explicitly says the contract itself is wrong.

- [FILL: 1-3 bullets naming the exact request/response shape and the use case/service it calls]

## Constraints

- Follow `docs/stack-profiles/{{STACK}}.md` conventions exactly (layer order, naming,
  error handling).
- Validate all input server-side; reject unknown fields.
- Match `docs/specs/api/openapi.yaml` exactly — request/response shape, status codes.
- Errors are RFC 9457 Problem Details.
- No business logic in the router/controller — delegate to the use case/service named
  above.

## Business rules to enforce

- [FILL: from the story's `## Business Rules` section]

## Error cases to handle

| Condition | HTTP Status | Detail |
|---|---|---|
| [FILL] | [FILL] | [FILL] |

## Tests required

- Unit: [FILL]
- Integration: [FILL]
