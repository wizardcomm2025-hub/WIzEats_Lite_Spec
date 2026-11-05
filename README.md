# WhizEats POS Lite — Specs

Living source of truth for the behaviour of WhizEats POS Lite: use cases, API contracts, data model, test cases and architecture decisions, across the Laravel POS API, the MySQL database, the React Native POS app and the super-admin web panel.

> **Setup status.** See [`GETTING-STARTED.md`](./GETTING-STARTED.md) for what is filled in and what is still open.

**New here?** Open [`tutorial.html`](./tutorial.html) for a 5-minute walkthrough of how the three stages (intake → write spec → live feature) work and which file to open when.

## Who this is for

- **Product & PMs** — write and review feature specs here
- **Designers (UX/UI)** — link Figma frames into specs; app screens follow the theme and shared components described in `POS_APP-master/CLAUDE.md`
- **Engineers** — read before implementing; propose changes in the same change as the code plan
- **Testers** — use the tester test cases in `features/<feature>/test-cases/`. They are UI-driven: run in the POS app as owner / waiter / chef, no endpoints, no request bodies
- **Developers** — use the developer test cases there too: direct API calls with garbage, unauthenticated or manipulated input (no or revoked token, another restaurant's IDs, a waiter token on an owner action, forged prices, extra fields)
- **Claude Code** — read `README.md` and the relevant `features/<feature>/` directory before any task

## Repository structure

```
App-Specs/
├── README.md                  ← you are here
├── CLAUDE.md                  ← orientation for Claude Code working in this folder
├── GETTING-STARTED.md         ← setup status: what's filled in and what's still open
├── tutorial.html              ← 5-min walkthrough of the 3-stage workflow (open in a browser)
├── developer-guide.html       ← repo-wide engineer onboarding (open in a browser)
├── project-overview.md        ← product summary, roles, platform decisions, API conventions, glossary
├── conventions.md             ← how to write a spec and its test cases (spec authoring, not code)
├── intake/                    ← pre-spec working folder: interview Q&A + client-provided info
├── features/
│   ├── template/              ← skeleton files for a new feature
│   └── <feature-name>/
│       ├── README.md          ← feature summary + index of use-cases
│       ├── developer-guide.html ← plain-language overview + per-surface key points (open in a browser)
│       ├── 01-<use-case>.md
│       ├── 02-<use-case>.md
│       ├── data-model.md      ← MySQL tables, columns and rules
│       ├── api-contract.md    ← endpoints, request/response, error messages
│       ├── ui-ux.md           ← POS app screens (and super-admin pages if in scope)
│       ├── test-cases/        ← tester (UI) and developer (API) test cases
│       └── assets/            ← screenshots, diagrams, exported Figma frames
├── adr/                       ← architecture decision records (numbered, immutable once accepted)
├── api/
│   └── openapi.yaml           ← HTTP API contract (status: placeholder, see GETTING-STARTED)
├── _for-implementation-repos/
│   └── CLAUDE.md              ← spec-reading rules to merge into the API and app CLAUDE.md
└── .claude/
    └── skills/                ← spec-authoring skills (auto-discovered by Claude Code)
```

## How the implementation repos use these specs

The specs are read in place. There is no copy inside the implementation repos.

```
C:\whizeats-lite\
├── App-Specs\                  ← this folder: WHAT each feature does + test cases
├── laravel-pos-api-dev_guid\   ← Laravel API, super-admin panel, SQL; reads specs at ../App-Specs/
│   └── specs\                  ← HOW: plan.md, tasks.md, test-results.md per feature
└── POS_APP-master\             ← React Native POS app; reads specs at ../App-Specs/
```

| Lives here (App-Specs) | Lives in `laravel-pos-api-dev_guid/specs/<feature>/` | Lives in code |
|---|---|---|
| Use cases, acceptance criteria | `plan.md` — files to change, steps, risks | Controllers, FormRequests, services, models |
| Data model, API contract, UI/UX | `tasks.md` — status per task | Migrations and `db_script/` SQL |
| Tester + developer test cases | `test-results.md` — actual result per test ID | App screens, slices, thunks, API clients |
| ADRs, glossary | | Postman collections / automated tests |

If you're unsure where something belongs: if it says **what** the system does or how to test it, it lives here. If it says **how** this change is built or how far it got, it lives in `laravel-pos-api-dev_guid/specs/`.

## Writing a new feature spec

1. Create `features/<name>/` by copying `features/template/`
2. Fill in `README.md`, the use-case files, `data-model.md`, `api-contract.md`, `ui-ux.md`, `test-cases/`, and `developer-guide.html` per [`conventions.md`](./conventions.md)
3. Get approval, then write `plan.md` / `tasks.md` in `laravel-pos-api-dev_guid/specs/<name>/`

## Editing an existing spec

The change description must state:

- What is changing and why
- Which layers are affected (POS API, MySQL, POS app, super-admin panel)
- Whether the change is breaking (migration needed) or additive
- Which test cases were added, updated or marked removed. Test cases always change with the spec.

## Spec conventions in brief

[`conventions.md`](./conventions.md) is the full spec-writing style guide. The short version:

- One use-case per file, numbered for stable ordering
- Each use case has: Summary, Prerequisites, Actors & preconditions, Main flow, Alternative flows, Acceptance criteria, Platform notes (POS API / Data — MySQL / POS app / Super-admin panel / Integrations), Configuration, Out of scope, Open questions
- Never duplicate behaviour across documents — link instead
- Open questions are tracked in-document. When resolved, mark them resolved with the answer; do not delete them
- Every use case has tester and developer test cases in the feature's `test-cases/` folder. They change together with the spec

## Skills bundled with the specs

Spec-authoring skills live under [`.claude/skills/`](./.claude/skills/): `spec-intake`, `feature-spec-from-figma`, `grill-me`. See [`.claude/skills/README.md`](./.claude/skills/README.md).

## For Claude Code

When working on a feature task in `laravel-pos-api-dev_guid` or `POS_APP-master`:

1. Read this `README.md`
2. Read `project-overview.md` (platform decisions, API conventions, known gaps, glossary)
3. Read `features/<feature>/README.md` and the relevant use-case file(s)
4. Read `api-contract.md`, `data-model.md` and `ui-ux.md` for the layers you touch
5. If a spec is ambiguous, missing, or contradicts another spec, stop and ask — do not guess
"# WIzEats_Lite_Spec" 
