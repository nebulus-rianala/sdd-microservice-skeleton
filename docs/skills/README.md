# Skills

Domain/task knowledge injected into a Claude Code or Codex session for a specific kind of
work. Load the relevant skill file **before** starting that type of task — don't rely on
re-deriving the convention from the spec each time.

| Skill | Load it when |
|---|---|
| [generate-technical-spec.md](generate-technical-spec.md) | Turning the functional spec into the authoritative technical spec (generator 1, run once at project start or when the functional spec materially changes) |
| [generate-planning.md](generate-planning.md) | Turning the technical spec into a sequenced weekly delivery plan (generator 2) |
| [generate-user-stories.md](generate-user-stories.md) | Expanding a plan week into implementation-ready story files (generator 3, run once per week) |
| [implement-feature.md](implement-feature.md) | Implementing any user story — the master vertical-slice playbook (domain → API → tests) |
| [write-migration.md](write-migration.md) | Any task that adds or changes a database table |
| [write-tests.md](write-tests.md) | Any task that needs unit/integration/contract/e2e test coverage (i.e. every story) |

The first three are **generators** — they read specs one level up and write the next
artifact down the chain (functional spec → technical spec → plan → stories). They are
normally invoked via their thin wrappers in `docs/prompts/`, not loaded directly.

The last three are **implementation skills** — they're loaded during the actual build of
a story, alongside whichever feature-area knowledge the story itself names.

All six skills defer to `CLAUDE.md`'s Absolute Rules and `docs/security/` on any conflict,
and all six carry forward the rule that **the spec wins over the story**.
