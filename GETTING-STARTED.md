# Getting started — setup status

This folder started from a generic spec template, was first adapted to another product (RMS), and was re-targeted to **WhizEats POS Lite** on 2026-11-05. This file tracks what is done and what is still open.

## Done

- Names, folder paths and the WhizEats stack (Laravel 12 API, MySQL, React Native app, Blade super-admin panel) are filled in across the folder.
- Platform notes use the fixed sub-sections: POS API, Data — MySQL, POS app, Super-admin panel, Integrations (see [`conventions.md`](./conventions.md#platform-notes)).
- Developer test categories are set for this API: authentication, account state, **restaurant isolation**, role, super-admin boundary, input validation, business-rule bypass, over-posting, concurrency, rate limiting, injection, error hygiene.
- The real response shapes, rate limits and known security gaps of the API are recorded in [`project-overview.md`](./project-overview.md#api-conventions).
- [`features/template/`](./features/template/) holds the skeleton for a new feature, including tester and developer test-case files.

## Still open

1. ~~**Fill in `project-overview.md`**~~. Done 2026-11-13: the product description, role matrix, POS app languages and out-of-scope list are written. The glossary's ordering and billing rules (channel codes, variants, tax, discount, bill arithmetic, numbering, payment types) were added the same day from the code and the feature specs.
2. ~~**Feature order**~~. Done: AUTH first, then all 18 features in `project-overview.md` → Core features (2026-11-05 to 2026-11-12).
3. ~~**Developer test runner**~~. Done 2026-11-13: PHPUnit 11 feature tests in `laravel-pos-api/api/tests/Feature/<FeatureFolder>/`, one class per use case, one method per case ID.
4. ~~**Super-admin panel scope**~~. Done 2026-11-12: in scope, split across ONB, RST, STAFF, TBL, MENU and ADM.
5. ~~**Implementation folder**~~. Done 2026-11-13:
   - `laravel-pos-api/api/specs/README.md` and `POS_APP-kot/specs/README.md` are created. The feature sub-folders come with each first plan.
   - The API repo has a new `CLAUDE.md` (project facts + spec rules), and `POS_APP-kot/CLAUDE.md` has the spec rules appended.
   - **Open follow-up:** the PHPUnit runner needs a MySQL test database built from `db_script`, because `phpunit.xml` uses in-memory SQLite. Make that the first step of the first plan.

   The original item read: create `laravel-pos-api/api/specs/` with the first plan. Add a `CLAUDE.md` to the API repo and merge [`_for-implementation-repos/CLAUDE.md`](./_for-implementation-repos/CLAUDE.md) into both repos' `CLAUDE.md`.
6. ~~**Decide what `api/openapi.yaml` is**~~. Decided 2026-11-13: it will be **generated from the Laravel code** (e.g. Scribe) when a developer adds a generator to the API repo in a later plan. Until then it is a placeholder. Each feature's `api-contract.md` is the contract, and it wins over any generated file.
7. ~~**Git.**~~ Checked 2026-11-13, re-checked 2026-11-24: this folder **is** a git repository.
   - Branch `main`, remote `origin` on GitHub (`WIzEats_Lite_Spec`). All 18 features and the setup are committed; the 2026-11-24 re-sync is uncommitted until someone commits it.
   - A `.gitignore` was added (personal `.claude/settings.local.json`, OS and editor files).
   - The API repo (`C:\xampp82\htdocs\laravel-pos-api\api`) **is** a git repository (branch `dev`, commit `d54d6c8` at the re-sync). The app (`D:\whizeats_Lite_Spec\frontend\POS_APP-kot`) is **not**, so app changes can't be diffed — re-check screens against `ui-ux.md` directly.
8. ~~**Re-sync with the code (2026-11-24).**~~ Done 2026-11-24: every feature and `project-overview.md` re-checked against API commit `d54d6c8`, `POS_APP-kot` and `db_script/pos_stage.sql`. Stance: **today's code plus a target for every known bug** ([conventions.md → Decision status](./conventions.md#decision-status)); the work list is the [bug-fix index](./project-overview.md#bug-fix-index-all-features-2026-11-24).
9. **Implementation-repo set-up — open again (found 2026-11-24).** At the current locations, the API repo has no `CLAUDE.md` and no `specs/` folder, and `POS_APP-kot/CLAUDE.md` has no "Specs" section and there is no `POS_APP-kot/specs/`. The 2026-11-13 merge was done in the repos' earlier locations. To do (in the code repos, with the developer): merge [`_for-implementation-repos/CLAUDE.md`](./_for-implementation-repos/CLAUDE.md) into both, and create `specs/README.md` in each. The PHPUnit MySQL test database (item 5) is still open.

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
