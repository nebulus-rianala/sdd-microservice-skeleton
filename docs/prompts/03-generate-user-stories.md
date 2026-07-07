# Prompt: Generate user stories for a week

Load and follow `docs/skills/generate-user-stories.md` for week {{WEEK}}. Run drift
detection first — the spec wins over the plan. Write stories under
`docs/user-stories/features/week-{{WEEK}}/`.

Prerequisite: `docs/planning/{{SERVICE_NAME}}-plan.md` exists (run
`docs/prompts/02-generate-planning.md` first if it doesn't).

If {{WEEK}} is not specified, default to the next week in the plan with no story files yet.
