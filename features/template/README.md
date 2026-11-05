# {{FEATURE_NAME}}

<!-- TODO: one paragraph — what this feature lets which role do, and why it matters to the restaurant. -->

**Feature code:** `{{FEATURE_CODE}}` (used in test IDs, for example `{{FEATURE_CODE}}-01-T01`, `{{FEATURE_CODE}}-01-D01`)

## Use cases

| # | Use case | Actor | Tester file | Developer file |
|---|---|---|---|---|
| 01 | [<!-- TODO: verb phrase -->](./01-use-case.md) | <!-- owner / waiter / chef / public / super admin --> | [tester](./test-cases/01-use-case.tester.md) | [developer](./test-cases/01-use-case.developer.md) |

## Key concepts

See the [glossary](../../project-overview.md#glossary). Terms specific to this feature:

- <!-- TODO, or "None beyond the glossary." -->

## Documents

- [data-model.md](./data-model.md) — MySQL tables and rules
- [api-contract.md](./api-contract.md) — endpoints and messages
- [ui-ux.md](./ui-ux.md) — POS app screens
- [test-cases/README.md](./test-cases/README.md) — fixtures, coverage matrix, endpoint checklist
- [developer-guide.html](./developer-guide.html) — plain-language overview
- Implementation: `laravel-pos-api-dev_guid/specs/<feature>/` (plan, tasks, test results)

## Depends on

<!-- TODO: cross-feature dependencies, or "None — this feature has no cross-feature dependencies." -->

## Decisions

<!-- Decisions taken for this feature, with date. Example: "{{FEATURE_CODE}}-D1 (YYYY-MM-DD): only owners may … — the API enforces it." -->

## Cross-cutting open questions

<!-- Numbered {{FEATURE_CODE}}-Q1, Q2… Resolved: append "_Resolved YYYY-MM-DD: answer._" — never delete. -->

## Feature-wide out of scope

- <!-- TODO -->
