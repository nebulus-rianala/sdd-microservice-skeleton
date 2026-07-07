# Skill: Generate the technical spec

Scope: **generator 1** of the SDD chain. Turns the functional spec in
`docs/specs/functional/` into the authoritative technical spec at
`docs/specs/technical/{{SERVICE_NAME}}-technical-spec.md`. Run once at project start, and
again (as an update, never a silent rewrite) whenever the functional spec materially
changes.

This is an **interactive, spec-producing procedure** — the "Intention → Specification"
step of SDD. Do not skip the interview to get to the spec faster; a spec built on
unstated assumptions is exactly what this skill exists to prevent.

This skill produces the one artifact every other artifact in this repository defers to:
**the spec wins over the story** (and over the plan) downstream — so get it right here,
via the interview, rather than letting an ambiguity ride into every later story.

## Phase A — Load context (read, do not generate yet)

1. `CLAUDE.md`
2. Every file in `docs/specs/functional/`
3. `docs/security/README.md` (always-loaded meta rules) + the relevant `docs/security/*.md` domain policies
4. `docs/adr/README.md` and every universal ADR it lists
5. `docs/stack-profiles/{{STACK}}.md` (once the stack is known — see Phase B if it isn't yet)

## Phase B — Interview (ASK before generating)

Do not emit the technical spec until every item below is resolved — either answered by
the user, or recorded as an explicit assumption marked `SECURITY-REVIEW-REQUIRED`. Batch
these into one set of questions rather than asking one at a time.

1. **Stack** — `laravel` or `fastapi`? Sets `{{STACK}}`, the canonical section order (see
   Appendix below), and the quality gates.
2. **Database** — `postgres` (default) or `mysql` (alternative)? Confirm INT PKs,
   `DATETIME` (not `TIMESTAMPTZ`), `JSON`/`JSONB` column types against that choice.
3. **Service identity** — `{{SERVICE_NAME}}`, `{{ID_PREFIX}}` (e.g. `INV`), API base
   path/version (`{{API_BASE}}`).
4. **Security-by-design framing** (from `docs/security/README.md`'s "Security by design" section) — the assets handled and
   their classification; the trust boundaries crossed (network, tenant, privilege); a
   short list of abuse cases; whether personal data is processed and, if so, its GDPR
   legal basis and whether a DPIA is required.
5. **Integrations** — which external services this service calls or is called by; sync vs
   async (pub/sub); Kong consumer groups relevant to this service's endpoints; any
   `/internal/*` routes and what calls them.
6. **Non-functionals** — expected volume (drives pagination/chunking decisions), data
   retention windows, observability expectations.
7. **Open decisions the functional spec leaves ambiguous** — list every one explicitly.
   Do not silently pick an answer and move on; either get a decision from the user now, or
   carry it into the spec's "Open Decisions" section (Phase C) unresolved.

Every unanswered item becomes an explicit assumption, marked inline:
`<!-- SECURITY-REVIEW-REQUIRED: source unavailable — assumption made: ... -->`.

## Phase C — Generate the spec

Write `docs/specs/technical/{{SERVICE_NAME}}-technical-spec.md` using the canonical
section order for the chosen stack (see Appendix below). Rules the output must satisfy:

- **Absolute Rules from `CLAUDE.md` and every applicable `MUST` from the loaded
  `docs/security/*.md` policies are non-negotiable.** Where the functional spec conflicts
  with one of them, the rule wins — state the conflict explicitly in the relevant section
  rather than quietly resolving it in the functional spec's favor.
- Include a **Security** subsection sized to the domain policy files that actually apply
  to this service (not all of them apply to every service — e.g. `mobile-security.md`
  only applies if there's a mobile client) and **security acceptance criteria** inside the
  Testing section.
- Include an explicit **"Open Decisions / Items Pending Clarification"** section — every
  assumption from Phase B lands here, never silently buried in prose elsewhere.
- Include a final **"Implementation Drifts"** section, empty at first — this is where
  generator 3's drift-detection step (see `generate-user-stories.md`) logs future
  divergence between what was planned/storied and what the spec actually says.
- End with a **self-check**, echoing `docs/security/README.md`'s end-of-generation
  checklist, confirmed against this spec specifically.
- State at the very top: *"This spec is authoritative. Do not edit it to match code; fix
  the code to match this spec."*

### Canonical section order

**`laravel`:**
1. Overview · 2. Technology Stack · 3. Project Structure · 4. Docker & Infrastructure ·
5. Database Schema · 6. API Reference · 7. Domain Model & Business Rules · 8. Queue &
Background Jobs · 9. External Service Integrations · 10. Authentication & Authorization
(Kong) · 11. Error Handling · 12. Testing Strategy · 13. Observability · 14. Migration /
Coexistence (if this service replaces or coexists with a legacy system; omit otherwise) ·
15. Environment Variables · 16. Implementation Drifts

**`fastapi`:**
1. Overview · 2. Architecture (hexagonal) · 3. Technology Stack · 4. Repository Structure
· 5. Database Design · 6. Application Layers · 7. Execution/Pipeline (if this service runs
an async processing pipeline; omit otherwise) · 8. Export/Contracts (if this service
serializes a data contract for another system/an LLM pipeline; omit otherwise) · 9. MCP
(if applicable) · 10. API Specification · 11. Sync/Integrations · 12. Docker Support · 13.
Coding Best Practices · 14. SDD Workflow · 15. Scope vs Roadmap · 16. Implementation Drifts

Both stacks additionally carry, inside the numbered sections above (not as extra
top-level numbers): a **Security** treatment aligned to the applicable `docs/security/*.md`
policies, the **Open Decisions / Items Pending Clarification** section, and the closing
**self-check**.

## Phase D — Propagate

After generating the spec:

1. Fill the placeholders it now resolves into root `CLAUDE.md` — `{{SERVICE_NAME}}`, the
   Technology Stack table, the Project Structure tree, the Domain Model, the API
   endpoints table, the Auth model table.
2. If the interview forced a new decision not already covered by an existing ADR, create a
   **new** ADR from `docs/adr/TEMPLATE.md` (next sequential number) — never edit an
   existing ADR to fit a new project's needs.
3. If a functional-spec ambiguity was resolved by the user during the interview rather
   than deferred, record the resolution in the spec's relevant section, not just in this
   skill's own memory of the conversation.

## Self-check before finishing

- [ ] Every `MUST` from the loaded `docs/security/*.md` policies applicable to this
      service is satisfied or explicitly deferred with `SECURITY-REVIEW-REQUIRED`
- [ ] No section is missing from the canonical order for the chosen stack
- [ ] "Open Decisions" section exists and lists every unresolved item from the interview
- [ ] "Implementation Drifts" section exists and is empty
- [ ] `CLAUDE.md` placeholders were updated, not left stale
- [ ] Any new decision became a new ADR file, not an edit to an existing one
