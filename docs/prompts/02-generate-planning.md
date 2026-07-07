# Prompt: Generate the delivery plan

Load and follow `docs/skills/generate-planning.md` against the technical spec in
`docs/specs/technical/`. Produce `docs/planning/{{SERVICE_NAME}}-plan.md`.

Prerequisite: `docs/specs/technical/{{SERVICE_NAME}}-technical-spec.md` exists (run
`docs/prompts/01-generate-technical-spec.md` first if it doesn't).

Reminder: the plan is not a specification. If anything you're about to write into the
plan would contradict the technical spec, stop and flag it — don't let the plan quietly
redefine what's being built.
