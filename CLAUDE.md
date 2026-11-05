# CLAUDE.md — App-Specs (WhizEats POS Lite)

This is the `App-Specs` folder at `C:\whizeats-lite\App-Specs`. It is the single source of truth for **what** WhizEats POS Lite does — covering the Laravel POS API, the MySQL database, the React Native POS app and the super-admin web panel listed in [`project-overview.md`](./project-overview.md#platforms).

**This folder contains no code.** It contains specifications, API contracts, data models, UI/UX notes, test cases and architecture decisions. Implementation happens in the sibling repos `../laravel-pos-api-dev_guid` (API, super-admin panel, SQL in `db_script/`) and `../POS_APP-master` (React Native app). Implementation plans, task status and test results live in `../laravel-pos-api-dev_guid/specs/<feature>/`.

> **Setup status.** Open setup items are listed in [`GETTING-STARTED.md`](./GETTING-STARTED.md).

---

## Orientation — read these first

1. `project-overview.md` — product, roles, platform decisions, **API conventions (real response shapes)**, known gaps and the glossary
2. `conventions.md` — the spec-writing and test-case style guide (about authoring specs, not code)

For a specific feature, also read:

3. `features/<feature-name>/README.md` — feature summary and index of use cases
4. The relevant use-case file(s), plus `data-model.md`, `api-contract.md`, `ui-ux.md` and `test-cases/`

For anything on a POS app screen, also read `../POS_APP-master/CLAUDE.md` (theme, shared components, navigation by role).

---

## Repo structure

```
App-Specs/
├── CLAUDE.md                        ← you are here
├── README.md                        ← human-facing entry point
├── GETTING-STARTED.md               ← setup status
├── tutorial.html / developer-guide.html
├── project-overview.md              ← product, roles, platform decisions, API conventions, glossary
├── conventions.md                   ← how to write a spec and its test cases
├── intake/                          ← pre-spec Q&A and client-provided info
├── features/
│   ├── template/                    ← skeleton files for a new feature
│   └── <feature-name>/
│       ├── README.md                ← feature summary + use-case index
│       ├── developer-guide.html     ← plain-language overview (HTML)
│       ├── NN-<use-case>.md
│       ├── data-model.md            ← MySQL tables, columns, rules
│       ├── api-contract.md          ← endpoints, messages, auth
│       ├── ui-ux.md                 ← POS app screens (and admin panel pages, if in scope)
│       ├── test-cases/
│       │   ├── README.md            ← fixtures, coverage matrix, endpoint checklist
│       │   ├── NN-<use-case>.tester.md
│       │   └── NN-<use-case>.developer.md
│       └── assets/
├── adr/
├── api/openapi.yaml                 ← placeholder; see GETTING-STARTED
├── _for-implementation-repos/CLAUDE.md
└── .claude/skills/
```

---

## Platform decisions (settled — do not reopen without an ADR)

The same table appears in `project-overview.md`; keep them in sync.

| Concern | Decision |
|---|---|
| Backend stack | Laravel 12 / PHP 8.2 (`laravel-pos-api-dev_guid`). Route → controller → FormRequest → service (`app/Services/<Module>/`) → Eloquent model |
| Database | One MySQL / MariaDB schema. Every restaurant row carries `restaurantId`, and every query filters on the tenant bound by `RestaurantTenantMiddleware` |
| Data access | Eloquent and the query builder inside services |
| Authentication | Sanctum tokens for the POS app (phone + PIN; legacy OTP). Session guard `admin` for the super-admin panel. `restaurantId` and `userId` come from the token, never from the request body |
| Authorization | Roles `owner` / `waiter` / `chef` in `users.role`. The app shows screens by role; the API does not check roles yet (see Known gaps) |
| API shape | `POST /api/<module>/<action>`; body `{ status, message, data }`; real HTTP codes (422 validation, 401, 403, 404, 500) |
| Client surfaces | React Native POS app (`POS_APP-master`); Blade super-admin panel in the API repo |
| Supported languages | Super-admin panel: en, es, de, bn. POS app: <!-- TODO: confirm from intake --> |

---

## Before a spec exists — use `intake/`

Capture what you know in `intake/<topic-slug>.md`. The [`spec-intake`](./.claude/skills/spec-intake/SKILL.md) skill runs the interview and writes Q&A as it goes. See [`intake/README.md`](./intake/README.md).

## How to write a new feature spec

1. Copy `features/template/` to `features/<name>/`
2. Start with `README.md` — summary, use-case table, feature code for test IDs
3. One file per use case, numbered (`01-`, `02-`, …), following `conventions.md`
4. Add `data-model.md` and `api-contract.md` once the data and API shape is clear
5. Add `ui-ux.md` for any feature with a POS app screen (or an admin panel page, if in scope)
6. Add `developer-guide.html` (HTML, not markdown; copy the template)
7. Add `test-cases/`: one tester and one developer file per use case, plus `README.md`
8. Get user approval, then plan in `../laravel-pos-api-dev_guid/specs/<name>/plan.md` and `tasks.md`

## How to edit an existing spec

State: what is changing and why; which layers are affected; **breaking** or **additive**; which test cases were added, updated or marked `Removed`. A behaviour change always updates its related test cases in the same change.

---

## Rules for writing specs

**One use-case per file.** Do not combine multiple user flows in one file.

**Never duplicate behaviour — link instead.**

**Open questions stay in the document.** When resolved, mark them resolved with the answer and the date — never delete them.

**Platform-specific behaviour goes in Platform notes** — the fixed sub-sections from `conventions.md`: POS API, Data — MySQL, POS app, Super-admin panel, Integrations.

**Acceptance criteria are mandatory**, and specific enough that a tester can derive pass/fail cases without asking. For app screens they include loading, empty, error and offline states and the role that sees the screen.

**Test cases change with the spec.** Tester (UI) and developer (API) cases per use case, under `features/<name>/test-cases/`. Test **definitions** live here; **actual results** are recorded in `../laravel-pos-api-dev_guid/specs/<name>/test-results.md`.

**Prerequisites are mandatory.** State `none` explicitly when there are none.

**Every feature ships with a `developer-guide.html`.**

---

## For Claude Code

- Do not generate code in this folder. This is a documentation folder.
- When asked to write a spec, read `conventions.md` in full first.
- When asked to update a spec, read the existing file in full first.
- When something is ambiguous, check `project-overview.md` (glossary) first. If that doesn't resolve it, stop and ask. Do not guess.
- For data-model facts, read `../laravel-pos-api-dev_guid/db_script/tables/tables.sql`, `database/migrations/` and `app/Models/`. If a table isn't there, ask the user for it.
- For real API behaviour (status codes, messages), read `routes/api.php`, the controller, its FormRequest in `app/Http/Requests/`, the service in `app/Services/`, `app/Http/Middleware/RestaurantTenantMiddleware.php` and `bootstrap/app.php`. Never invent a response a test expects; raise an open question instead.
- For app behaviour (screens, messages, role routing), read `../POS_APP-master/src/app/` and its `CLAUDE.md`.
- Preserve the history of open questions.
- This folder is not a git repository yet. Do not create branches or commits here unless the user asks.
