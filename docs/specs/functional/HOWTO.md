# How to add the functional spec

Drop your team's functional/product spec for {{SERVICE_NAME}} in this folder as one or
more Markdown files (e.g. `{{SERVICE_NAME}}-functional-spec.md`). This is the only input
generator 1 (`docs/prompts/01-generate-technical-spec.md`) needs to start.

No fixed section list is imposed here — this document is product-owned, not
implementation-owned — but a functional spec that covers at least the following will make
generator 1's interview much shorter:

- **Product overview** — what this service does, one paragraph.
- **Actors & personas** — who calls this service, directly or via Kong.
- **Domain model** — the core entities and how they relate.
- **State machine** — if the domain has a root entity with a lifecycle, its states and
  valid transitions.
- **Functional flows** — the main user/system journeys, step by step.
- **Business rules** — the rules a developer must not get wrong, stated as rules, not
  buried in flow prose.
- **Error & edge cases** — what should happen when things go wrong, in product terms.
- **Non-functional constraints** — expected volume, retention, any hard latency/uptime
  requirement.
- **Known gaps / open questions** — anything the product side hasn't decided yet. Generator
  1 will surface these explicitly rather than silently resolving them.

Once this folder has content, run `docs/prompts/01-generate-technical-spec.md`.
