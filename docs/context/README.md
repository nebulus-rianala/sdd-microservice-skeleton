# Context

Background/legacy reference material: notes on a system this service replaces, external
event-bus conventions, prior-version migration guides — anything that helps a developer or
an AI agent understand **why** something is done a certain way, without being prescriptive
about **what** to build.

Context documents are **read-only reference material**. They are explicitly **not
authoritative** — changes to business rules or domain models must first update
`docs/specs/`, never a context document. If a context document and the technical spec ever
disagree, the spec wins.

## Suggested starting set

- Legacy-system mapping (if this service replaces or coexists with an older system)
- External event-bus / pub-sub conventions this service participates in
- Any migration/coexistence call-guide for a cutover period
