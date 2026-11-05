# Getting started — setup status

This folder started from a generic spec template, was first adapted to another product (RMS), and was re-targeted to **WhizEats POS Lite** on 2026-11-05. This file tracks what is done and what is still open.

## Done

- Names, folder paths and the WhizEats stack (Laravel 12 API, MySQL, React Native app, Blade super-admin panel) are filled in across the folder.
- Platform notes use the fixed sub-sections: POS API, Data — MySQL, POS app, Super-admin panel, Integrations (see [`conventions.md`](./conventions.md#platform-notes)).
- Developer test categories are set for this API: authentication, account state, **restaurant isolation**, role, super-admin boundary, input validation, business-rule bypass, over-posting, concurrency, rate limiting, injection, error hygiene.
- The real response shapes, rate limits and known security gaps of the API are recorded in [`project-overview.md`](./project-overview.md#api-conventions).
- [`features/template/`](./features/template/) holds the skeleton for a new feature, including tester and developer test-case files.

## Still open

1. **Fill in `project-overview.md`** through a `spec-intake` interview: product description, what each role (owner / waiter / chef) may do, POS app languages, the rest of the glossary, out of scope. The questions are listed in [`intake/project-overview.md`](./intake/project-overview.md) → Open.
2. **Feature order** — which module gets the first spec. It becomes the worked example that the others copy.
3. **Developer test runner** — Postman collection (+ Postman CLI), or PHPUnit / Pest feature tests in `laravel-pos-api-dev_guid/tests/`. The test-case files are written runner-neutral until this is decided.
4. **Super-admin panel scope** — covered by specs and tests now, or POS app + API only.
5. **Implementation folder** — create `laravel-pos-api-dev_guid/specs/` with the first plan. Add a `CLAUDE.md` to the API repo and merge [`_for-implementation-repos/CLAUDE.md`](./_for-implementation-repos/CLAUDE.md) into both repos' `CLAUDE.md`.
6. **Decide what `api/openapi.yaml` is** — hand-maintained, generated from the Laravel routes, or dropped.
7. **Git.** This folder is not a git repository. The "change with the spec" rules work without git, but history and review are easier with one.

Find the remaining TODO callouts with:

```sh
grep -rn "TODO" . --include="*.md" --include="*.html" --exclude-dir=template
```

## Writing a feature

```sh
cp -R features/template features/<feature-name>
```

Then in the new folder:

- Rename `01-use-case.md` to `01-<actual-slug>.md`, and the test-case files to match
- Replace `{{FEATURE_NAME}}` and `{{FEATURE_CODE}}` wherever they appear
- Fill in the TODO callouts
- Read `conventions.md` once before drafting

## Skills

Run Claude Code inside this folder so the skills under `.claude/skills/` auto-discover. Start with `spec-intake`, then `feature-spec-from-figma`. Use `grill-me` to pressure-test a plan.
