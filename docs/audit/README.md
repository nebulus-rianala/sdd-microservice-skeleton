# Audit

One acceptance record per delivered weekly batch, confirming (or noting deviation from)
the stories' acceptance criteria and the stack's quality gates.

Audit records are **append-only**. Once written, they are not edited — a deviation
discovered later generates a new audit record or a fix story (`docs/user-stories/fixes/`,
using `FIX-TEMPLATE.md`), never a rewrite of a past record.

## Record format

```markdown
# Audit — Week NN ({{ID_PREFIX}}-XXX through {{ID_PREFIX}}-XXX)

**Date:** · **Reviewer:** · **Branch:** ·
**Test results:** N/N pass · Coverage: N% · **Static analysis:** 0 errors

## Stories reviewed

### {{ID_PREFIX}}-XXX — Title

- [PASS] AC1: ...
- [DEVIATION] AC3: ... — actual behavior: ... — disposition: acceptable | fix required

## Quality gate results

| Gate | Result |
|---|---|
| Style | |
| Static analysis | |
| Coverage | |

## Open items

- [FIX-NNN] Description of the deviation that requires a fix story, if any
```

A `[DEVIATION]` marked "fix required" must spawn a `FIX-NNN` story before the week is
considered closed. A `[DEVIATION]` marked "acceptable" is a deliberate, reviewed decision
— not a silently-ignored gap — and stays recorded as such permanently.
