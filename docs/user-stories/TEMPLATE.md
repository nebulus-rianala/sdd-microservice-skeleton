# User Story — [{{ID_PREFIX}}-XXX] Title

**Week:** {{WEEK}} · **Epic:** {{EPIC}} · **Status:** Draft | Ready | In Progress | Done

## Context

As a {{ACTOR}}, I want to {{CAPABILITY}}, so that {{BENEFIT}}.

## Spec references

- `docs/specs/technical/{{SERVICE_NAME}}-technical-spec.md` §{{SECTION}}
- `docs/specs/api/openapi.yaml` — operation {{METHOD}} {{PATH}} (if applicable)
- `docs/specs/data-contracts/{{CONTRACT}}.schema.json` (if applicable)

## Acceptance Criteria

- [ ] {{CRITERION_1}}
- [ ] {{CRITERION_2}}

## Business Rules

Rules pulled in intent (not paraphrased away) from the technical spec section above —
if a rule here and the spec ever disagree, the spec wins; fix this story, not the spec.

- {{RULE}}

## Error Cases

| Condition | HTTP Status | Detail |
|---|---|---|
| {{CONDITION}} | {{STATUS}} | {{DETAIL}} |

## Tests Required

- Unit: {{WHAT}}
- Integration: {{WHAT}}

## Skills to load

- `docs/skills/implement-feature.md`
- {{ADDITIONAL_SKILL_IF_ANY}} (e.g. `write-migration.md` if this story adds a table)

## ADRs in scope

- {{ADR_ID}} — {{WHY_IT_APPLIES_HERE}}

## Claude Code prompt

```
Read docs/specs/technical/{{SERVICE_NAME}}-technical-spec.md §{{SECTION}}, the skills
above, and docs/security/README.md. Validate this story: confirm the acceptance criteria
match the spec, flag any missing edge case, and produce a concrete implementation contract
(exact files, method signatures, test list) for Codex to generate from. Do not write
implementation code yourself — validate and produce the contract.
```

## Codex prompt

```
Using the implementation contract above and docs/agents/codex/system-prompt.md, implement
{{ID_PREFIX}}-XXX. Follow docs/stack-profiles/{{STACK}}.md conventions exactly. Write tests
alongside the implementation per docs/skills/write-tests.md.
```

## Out of scope

- {{EXPLICITLY_NOT_THIS_STORY}}

## Notes

- {{NOTE}}
