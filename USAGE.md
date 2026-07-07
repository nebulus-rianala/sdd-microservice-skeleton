# USAGE — Clone → Spec → Generate → Implement

The end-to-end operator guide for this skeleton. Read this once before starting a new
service; the three generator prompts each carry their own shorter reminder inline.

## 1. Clone and rename

```bash
git clone <this-skeleton-repo> {{SERVICE_NAME}}
cd {{SERVICE_NAME}}
```

Set what you already know in `CLAUDE.md`: `{{SERVICE_NAME}}`, `{{STACK}}` (`laravel` or
`fastapi`), `{{DB}}` (`postgres` default or `mysql`), `{{ID_PREFIX}}`, `{{API_BASE}}`.
Leave anything you don't know yet as a placeholder — generator 1's interview will ask for
it.

## 2. Drop the functional spec

Put your team's functional/product spec in `docs/specs/functional/`. See
`docs/specs/functional/HOWTO.md` for the shape that makes generator 1 fastest, though no
fixed format is enforced — this is a product-owned document.

## 3. Run generator 1 — technical spec

```
docs/prompts/01-generate-technical-spec.md
```

Interactive. It will ask for the stack, the database, service identity, the SDD-00
security framing (assets, trust boundaries, abuse cases, GDPR legal basis if personal
data is involved), integrations, and non-functionals — batched as one set of questions.
It will not generate the spec until every question is answered or explicitly deferred
with `SECURITY-REVIEW-REQUIRED`.

Output: `docs/specs/technical/{{SERVICE_NAME}}-technical-spec.md` (authoritative from this
point on), plus filled-in placeholders in `CLAUDE.md` and any new ADR the interview forced.

## 4. Run generator 2 — delivery plan

```
docs/prompts/02-generate-planning.md
```

Reads the technical spec, produces `docs/planning/{{SERVICE_NAME}}-plan.md` — weekly
batches, each with Deliverables / Claude Code prompt / Codex prompt / Acceptance criteria.

## 5. Run generator 3 — user stories

```
docs/prompts/03-generate-user-stories.md
```

Run once per week (default: the next unstarted week in the plan). Runs drift detection
first — if the plan has drifted from the spec since it was written, the stories follow
the spec, and the divergence is logged in both the week's own drift note and the spec's
"Implementation Drifts" section. Output: one file per story under
`docs/user-stories/features/week-NN/`.

## 6. Implement

For each story:

1. **Claude Code validates** — reads the story, its spec references, the relevant skills
   under `docs/skills/`, and the security policies in scope; challenges assumptions, flags
   missing edge cases, produces a concrete implementation contract.
2. **Codex generates** — production code and tests from that contract, briefed via
   `docs/agents/codex/system-prompt.md` + the relevant `docs/agents/codex/task-templates/*.md`.
3. **Claude Code reviews** — the generated output against the spec, the ADRs in
   `docs/adr/`, the security policies, and the stack's quality gates
   (`docs/stack-profiles/{{STACK}}.md`). Deviations are marked `[DEVIATION]` with a
   disposition (acceptable / fix required) — never silently merged.
4. **Record acceptance** in `docs/audit/` once all gates pass, following
   `docs/audit/README.md`'s format. A `[DEVIATION]` marked "fix required" spawns a
   `FIX-NNN` story (`docs/user-stories/fixes/`, using `FIX-TEMPLATE.md`) before the week
   closes.

## 7. The feedback loop

Human review → spec update (if the review reveals the spec itself was wrong, not just the
implementation) → next cycle. **The spec always wins over the story**: if an
implementation, a story, or a plan is ever found to contradict the current technical spec,
the spec is correct and the other artifact is stale — fix the other artifact, or update the
spec deliberately (via generator 1, with the change recorded), never by quietly editing the
spec to match whatever the code happened to do.

---

## Self-check (run after cloning, before starting real work)

- [ ] Every folder in the tree below is present
- [ ] Every `.md` file is non-empty
- [ ] No `{{PLACEHOLDER}}` is left unresolved in `CLAUDE.md` once generator 1 has run
- [ ] Every ADR in `docs/adr/README.md`'s index is classified (universal / laravel / fastapi)
- [ ] All three generators (`docs/skills/generate-*.md`) cross-reference
      `docs/security/`, `docs/adr/`, and `docs/stack-profiles/`
- [ ] "The spec wins over the story" (or an equivalent explicit statement) appears in all
      three generator skill files
- [ ] No invented library, framework, or hardcoded secret appears anywhere in the skeleton

## What to fill in next (a fresh clone, before generator 1 has run)

- [ ] `CLAUDE.md` — `{{SERVICE_NAME}}`, `{{STACK}}`, `{{DB}}`, `{{ID_PREFIX}}`, `{{API_BASE}}`
- [ ] `docs/specs/functional/` — at least one functional spec file
- [ ] `README.md` — `{{SERVICE_NAME}}` (top-level title placeholder)
- [ ] Run generator 1, 2, 3 in order — do not skip generator 1 by hand-writing the
      technical spec; the interview is what catches unstated assumptions
