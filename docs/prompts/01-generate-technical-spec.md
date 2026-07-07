# Prompt: Generate the technical spec

Load and follow `docs/skills/generate-technical-spec.md`. Begin with the interview
(Phase B). Do not generate the spec until Phase B is complete — every question either has
an answer or an explicit `SECURITY-REVIEW-REQUIRED` assumption on record.

Prerequisite: at least one file exists in `docs/specs/functional/` (see
`docs/specs/functional/HOWTO.md` if it's empty).

Output: `docs/specs/technical/{{SERVICE_NAME}}-technical-spec.md`, plus the `CLAUDE.md`
placeholder updates and any new ADR described in the skill's Phase D.
