# Contributing

Thanks for your interest in improving the SDD Microservice Skeleton. This repository is
documentation only: every file is Markdown (plus one OpenAPI template) that humans and
coding agents read and obey. A contribution is a change to those rules, templates, or
prompts.

## What makes a good contribution

- **A fix to an inconsistency.** Two files that disagree (for example a stack profile that
  contradicts an ADR) are bugs. Point out both locations and say which one should win.
- **A convention backed by real use.** The skeleton was extracted from real services, not
  invented. When you propose a new rule, say where it was applied and what problem it
  solved. Untested ideas go in an issue for discussion first.
- **A new stack profile.** Add `docs/stack-profiles/<stack>.md` that follows the shape of
  the existing `laravel.md` / `fastapi.md`, and update every place that lists the supported
  stacks (`CLAUDE.md`, `README.md`, the generator skills).
- **Clearer wording.** Agents follow instructions literally, so an ambiguous sentence
  causes real damage. Making a rule more precise is a welcome change.

## Ground rules

- **Keep the shared core stack-agnostic.** Anything true for only one stack belongs in
  that stack's profile, not in `CLAUDE.md`, the ADRs, or `docs/security/`.
- **Don't edit an accepted ADR.** Supersede it with a new one (see `docs/adr/README.md`).
- **Keep placeholders intact.** `{{PLACEHOLDER}}` tokens are filled in per service by
  generator 1. Don't replace them with concrete values in the skeleton itself.
- **No organization-specific content.** No company names, internal hostnames, private
  product names, or real credentials. Use `example.com` for domains.
- **Run the self-check** at the end of `USAGE.md` before opening a pull request.

## Workflow

1. Fork the repository and create a branch (`feature/<slug>` or `fix/<slug>`).
2. Make the change and keep cross-references consistent. If you rename or move a file,
   search the repository for its old path.
3. Open a pull request that explains the problem, the change, and any evidence that it
   works in practice.

By contributing, you agree that your contributions are licensed under the
[MIT License](LICENSE).
