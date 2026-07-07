# User Story — [FIX-XXX] Title

**Week discovered:** {{WEEK}} · **Originating story:** {{ID_PREFIX}}-XXX ·
**Epic:** {{EPIC}} · **Status:** Draft | Ready | In Progress | Done

## Root cause

What was actually wrong, and why — name the specific spec section, story, or code that was
incorrect. This section exists so a reviewer never has to guess "why does this fix story
exist" from the diff alone.

{{ROOT_CAUSE_DESCRIPTION}}

## Context

As a {{ACTOR}}, I want {{CORRECTED_BEHAVIOR}}, so that {{WHY_IT_MATTERS}}.

## Spec references

- `docs/specs/technical/{{SERVICE_NAME}}-technical-spec.md` §{{SECTION}}
  (the section that was wrong, or that this fix clarifies)

## What NOT to change

The negative-scope guardrail — list everything adjacent to the bug that must stay exactly
as it is, so a fix doesn't quietly grow into a refactor:

- Do not change {{METHOD_SIGNATURE_OR_BEHAVIOR}} beyond the specific defect described above.
- Do not touch {{ADJACENT_COMPONENT}} — it is unaffected by this fix.

## Acceptance Criteria

- [ ] {{CRITERION_CONFIRMING_THE_FIX}}
- [ ] {{CRITERION_CONFIRMING_NOTHING_ELSE_CHANGED}}

## Claude Code prompt

```
Read docs/specs/technical/{{SERVICE_NAME}}-technical-spec.md §{{SECTION}} and the
originating story {{ID_PREFIX}}-XXX. Confirm the root cause above is accurate against the
current code and spec. Validate the fix's scope against "What NOT to change." Produce an
implementation contract for Codex.
```

## Codex prompt

```
Using the implementation contract above, apply the fix. Do not modify anything listed
under "What NOT to change." Follow docs/stack-profiles/{{STACK}}.md conventions. Add a
regression test that would have caught this defect.
```

## Reference document updates required

Runs **after** the code fix is validated (this is the step that keeps the spec/story
corpus from re-encoding the same bug into future work):

- `docs/specs/technical/{{SERVICE_NAME}}-technical-spec.md` §{{SECTION}} — {{WHAT_TO_CORRECT}}
- `CLAUDE.md` §{{SECTION}} (if the wrong assumption was stated there too)
- Produce a diff of the exact lines changed in each document, for review — do not
  silently rewrite a whole section.

## Notes

- {{NOTE}}
