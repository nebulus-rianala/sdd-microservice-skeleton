# ADR-NNNN: Title

**Date:** YYYY-MM-DD
**Status:** Accepted | Superseded by ADR-NNNN | Deprecated
**Deciders:** {{WHO_WAS_INVOLVED}}

## Context

What forced this decision. What were the constraints or trade-offs? What would happen if
we did nothing?

## Decision

What was decided, stated as an instruction an AI agent can follow without interpretation.

## Consequences

What becomes easier. What becomes harder. What is now forbidden.

---

## Rules for this file type

- ADRs are **immutable once accepted**. Never edit the Decision or Context of a past ADR.
- To reverse a decision: write a **new** ADR, set its Status to `Accepted`, and set the
  old ADR's Status to `Superseded by ADR-NNNN`. Do not delete the old ADR.
- Number sequentially, zero-padded to 4 digits (`ADR-0001`, `ADR-0002`, …). No gaps.
- An ADR that only applies to one stack lives in `docs/stack-profiles/{{STACK}}.md`
  instead of this folder, and is still listed (with its scope marked) in
  `docs/adr/README.md`'s index.
