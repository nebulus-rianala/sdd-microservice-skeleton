# Skill: Generate user stories

Scope: **generator 3** of the SDD chain. Reads the plan, the technical spec, and the
skills, and emits implementation-ready user stories into
`docs/user-stories/features/week-{{WEEK}}/`, one file per story, following
`docs/user-stories/TEMPLATE.md`. Run once per week, targeting the next unstarted week by
default.

## Procedure

1. **Load:** `CLAUDE.md`, `docs/planning/{{SERVICE_NAME}}-plan.md`,
   `docs/specs/technical/{{SERVICE_NAME}}-technical-spec.md`, `docs/skills/README.md`,
   `docs/adr/README.md`, `docs/security/README.md`.
2. **Pick the target week** — a parameter to this skill; default to the next week in the
   plan that has no story files yet under `docs/user-stories/features/week-{{WEEK}}/`.
3. **Drift detection (mandatory, runs before writing anything).** Diff the target week's
   planned deliverables against the *current* technical spec — the plan may have been
   written before a subsequent spec update. If the plan (or a prior story) diverges from
   the current spec:
   - **The spec wins.** Write the stories to match the spec, not the stale plan.
   - List every divergence found in a `## Drifts detected` block at the very top of the
     week's folder (a `README.md` in `docs/user-stories/features/week-{{WEEK}}/`),
     mirroring the technical spec's own "Implementation Drifts" section format:
     what was planned, what the spec actually says now, and which one this batch of
     stories follows (always the spec).
   - Append the same divergence to the technical spec's "Implementation Drifts" section,
     so the spec's own drift log stays the single place this history is recorded.
4. **Expand each deliverable into one or more stories**, each named `{{ID_PREFIX}}-###`
   (sequential across the whole project, not reset per week), following
   `docs/user-stories/TEMPLATE.md` exactly:
   Context · Spec references · Acceptance Criteria · Business Rules (pulled in intent,
   not paraphrased away, from the spec section referenced) · Error Cases with HTTP codes ·
   Tests Required · Skills to load · ADRs in scope · Claude Code prompt · Codex prompt ·
   Out of scope · Notes.
5. **One independently-implementable unit per story.** Never expand a story to cover more
   than one vertical slice — if a deliverable is too big for one story, split it into
   multiple stories with an explicit dependency order noted in each story's `## Notes`,
   so Codex/subagents can take one story each with its own targeted context.
6. **Fix stories.** When a story discovered *during implementation* (not planning) turns
   out to be a correction to already-delivered work rather than new work, generate it from
   `docs/user-stories/FIX-TEMPLATE.md` instead, and place it in
   `docs/user-stories/fixes/` — never inside a `week-NN/` folder, and never using the
   plain `TEMPLATE.md`.

## Self-check before finishing

- [ ] Drift detection ran first; any divergence found is logged in both the week's
      `## Drifts detected` block and the spec's "Implementation Drifts" section
- [ ] Every story file follows `TEMPLATE.md`'s exact section list
- [ ] Every story references concrete spec sections, not "see the spec"
- [ ] No story exceeds one independently-implementable unit
- [ ] Fix stories (if any this run) use `FIX-TEMPLATE.md` and live in `fixes/`, not a week folder
- [ ] Story IDs are sequential and don't collide with an existing story anywhere in the project
