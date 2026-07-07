# AI/LLM Integration Security — if the generated feature integrates an LLM

Ref: OWASP LLM Top 10. This file governs a **feature this service implements** that
itself calls or embeds an LLM (e.g. a summarization endpoint, an agentic workflow). It is
distinct from `README.md`'s AI behavior rules, which govern the coding assistant that is
generating this codebase, not code the codebase runs at runtime.

## Prompt injection

Treat any externally-sourced prompt content (user input, a fetched document, output from
another model or tool) as untrusted. Strictly separate system instructions from user/data
content in the prompt construction — never concatenate untrusted content directly into
the instruction the model is told to obey.

## Tool/function exposure

Restrict the tools or functions exposed to the model to the strict minimum an allowlist
requires for the feature. Do not expose a general-purpose file/network/shell tool to a
model handling untrusted input unless the feature's threat model explicitly requires it
and it's marked `SECURITY-REVIEW-REQUIRED`.

## Input/output filtering

Filter sensitive data out of both what's sent to the model and what the model returns —
a model response is not automatically safe to display or log verbatim (see
`gdpr-compliance.md`, `logging-and-audit.md`).

## Output validation

Validate and constrain the model's output before it triggers any side-effecting action
(a DB write, an outbound call, a state transition). Treat model output as untrusted input
to the rest of the system, subject to the same rules as `input-validation.md`.

## Auditability

Log the actions a model triggers (which tool, which parameters, which result) so an
agentic flow's behavior is reconstructable after the fact — see `logging-and-audit.md`.
