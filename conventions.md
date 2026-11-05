# Conventions for writing specs

> **What this is:** the style guide and template for **specifications** in this folder. It governs how use-case files, feature READMEs, data models, API contracts, UI/UX files, test cases and ADRs are written.
>
> **What this is not:** a code style guide. Code conventions live in `laravel-pos-api-dev_guid/CLAUDE.md` (to be added) and `POS_APP-master/CLAUDE.md`.

Read this before writing or significantly changing a spec.

For directory structure and workflow, see [`README.md`](./README.md). This document is about the **content** of specs.

> **Before a spec exists.** Discovery Q&A and client-provided facts are captured under [`intake/`](./intake/). That material is not a spec — it's the raw input that becomes one.

## The use-case template

Every use-case file (`features/<feature>/NN-<use-case>.md`) contains these sections in this order:

```
# NN — <Verb phrase>

## Summary
## Prerequisites
## Actors & preconditions
## Main flow
## Alternative flows
## Acceptance criteria
## Platform notes
### POS API
### Data — MySQL
### POS app
### Super-admin panel
### Integrations
## Configuration
## Out of scope
## Open questions
```

The skeleton is [`features/template/`](./features/template/). The first feature written becomes the worked example; link it here when it exists.

### Title

Format: `NN — <verb phrase>` where `NN` is a stable two-digit number.

- Numbers are stable. Don't renumber when reordering — the feature `README.md` index controls display order.
- The verb phrase describes the user or system action ("Add a location", "Switch a location on or off").

### Summary

One paragraph (3–5 sentences): the actor, the action, the outcome, and why it matters. No implementation detail.

### Prerequisites

What must be in place before this use case can be implemented end-to-end. A *gate*: if a prerequisite is unmet, work pauses or proceeds against a stub — not blindly.

- Bullet list; each bullet names the prerequisite and links to where it is defined.
- Group by type when it helps (`Backend`, `Data`, `Depends on`).
- State `none` explicitly when there are none.
- **Link, don't restate.** Endpoints → `api-contract.md`; tables and migrations → `data-model.md`; sibling use cases → their file.
- Cross-feature dependencies belong in the feature `README.md`'s `## Depends on` section.

### Actors & preconditions

Who triggers the use case (owner, waiter, chef, super admin, public customer, system), the trigger, and the state that must hold (restaurant approved, onboarding done, channel enabled in restaurant config…). Errors go in Alternative flows.

### Main flow

The happy path as a numbered list of system behaviour, 5–10 top-level steps. **What** happens, not how. Present tense, active voice. Reference data and APIs by their names in `data-model.md` / `api-contract.md`.

### Alternative flows

Edge cases, errors and branches. Each bullet starts with a bold short name: `- **Duplicate name.** …`. Cover failures, validation, double submit, concurrency, inactive / deleted state, role, no network, restaurant not approved.

### Acceptance criteria

Numbered, observable, independently testable conditions; "Given X, when Y, then Z" style. 5–15 per use case. Prefer observable outcomes ("the table appears in the zone as Inactive") over implementation language. For app screens, include the loading, empty, error and offline states and which role sees the screen, or link to the feature's `ui-ux.md` criteria.

### Platform notes

Every use case has the same five sub-sections, in this order:

| Sub-section | Covers |
|---|---|
| **POS API** | Endpoints (`POST /api/<module>/<action>`), middleware (`auth:sanctum` + `restaurant.tenant`, a throttle, or public), the FormRequest rules, server-side role checks, where `restaurantId` / `userId` come from (always the token), every response message and HTTP status. |
| **Data — MySQL** | Tables read or written, the `restaurantId` filter, `isActive` / `isDelete` handling, status values, columns that must stay consistent (for example `tables.status` with the running order). |
| **POS app** | Screen behaviour in `POS_APP-master`: screen name, which roles reach it, states, toasts, offline / retry, printing. Detailed layout lives in the feature's `ui-ux.md`; link to it. |
| **Super-admin panel** | Pages in the Blade panel that show or change the same data. Write "Not touched." when the panel isn't involved. |
| **Integrations** | E-mail, Google Sheets error log, image storage, Excel import / export, printer, or other external systems. |

If a part has nothing distinctive to say, write "No platform-specific behaviour beyond the main flow." or "Not touched." rather than omitting it.

### Configuration

What is data-driven vs hard-coded, and where the data lives (`restaurant_configs` switches, master tables, `.env` / `config/*.php`). Describe **what is configurable**, not the values.

### Out of scope

Bullet list. Each line links to where the topic is covered or marks it TBD/future.

### Open questions

- When resolved, append the answer in italics with a date; never delete the question: `- Question? _Resolved YYYY-MM-DD: answer._`
- Cross-cutting questions go in the feature `README.md`.
- Claude Code treats open questions as blockers — do not infer answers; ask.

## File scope and context-window economy

Each use-case file is one **vertical slice**: one user-observable behaviour through the UI, the API call, the service and the data write. An agent should be able to implement and verify that slice from the use-case file plus the feature `README.md`, `data-model.md`, `api-contract.md`, `ui-ux.md` and its test-case files. Shared content lives in those shared files and is linked, not duplicated.

## Writing style

- **Active voice, present tense.**
- **Behaviour, not implementation.** Specs say what; code says how.
- **No values that should be configuration.**
- **Product and table names are allowed** (MySQL, Sanctum, `tables`, `restaurant_configs`). Specific libraries are implementation detail.
- **One idea per sentence.**

## Cross-references and linking

- Link other specs by relative path.
- **Never duplicate behaviour.** "Out of scope" may briefly mention a topic before linking.
- Reference Figma frames by link AND export a screenshot into `assets/`.

## Other document types

### Feature `README.md`

- One-paragraph summary
- Index table of use cases
- Key concepts (or a link to the glossary in `project-overview.md`)
- Links to `data-model.md`, `api-contract.md`, `ui-ux.md`, `test-cases/README.md`, and the implementation folder `laravel-pos-api-dev_guid/specs/<feature>/`
- The feature code used in test IDs (for example `TBL` for zones & tables)
- **Depends on** — cross-feature dependencies, or `None — this feature has no cross-feature dependencies.`
- **Decisions** and **cross-cutting open questions**
- **Feature-wide out of scope**

### Feature `ui-ux.md`

Required for every feature with a POS app screen (and for super-admin panel pages, if the panel is in scope). It covers: screen and navigation entry (which roles reach it, from where), layout and fields, bottom sheets / dialogs, feedback states (loading, empty, error, offline, success toast), confirmations, role-based visibility, phone and tablet sizes, printing / sharing output if any, and the shared components reused from `POS_APP-master/src/shared/components/`. Copy text the user sees appears here verbatim, because tester cases quote it.

### Feature `developer-guide.html`

Required, one per feature, **HTML not markdown**. Copy [`features/template/developer-guide.html`](./features/template/developer-guide.html), keep its styling. Contents: what the feature is; v1 scope (link to the README); how it works end-to-end; a **What matters where you sit** section with one subsection per Platform-notes part. Summarise and link — the formal specs win if they disagree.

### `data-model.md`

One per feature. The data the feature owns in the MySQL schema:

- Tables created, altered or relied on, with primary key and purpose
- Columns of new or altered tables, constraints, status values, and the rules the service checks (always including the `restaurantId` filter)
- Migrations added or changed, and the matching change to `db_script/` (scripts are re-runnable: `IF NOT EXISTS`, guarded `ALTER`)
- Seed data, indexes that matter, row lifecycle (`isActive`, `isDelete`)
- **Naming follows the existing schema** (camelCase columns, `binary(16)` UUID keys, `isActive` / `isDelete`). Read the tables you touch and match their style.
- Physical SQL lives in `laravel-pos-api-dev_guid/database/migrations/` and `laravel-pos-api-dev_guid/db_script/`, and must match this file.

### `api-contract.md`

One per feature: endpoints, request and response fields, auth attribute, every message the endpoint can return, and which responses come from shared layers (see [API conventions](./project-overview.md#api-conventions)). `api/openapi.yaml` is a placeholder until GETTING-STARTED item 2 is decided; until then `api-contract.md` is the contract.

### Test cases (`test-cases/`)

Every feature has a `test-cases/` subfolder derived from its use cases, data model, API contract and UI/UX. Start from [`features/template/test-cases/`](./features/template/test-cases/).

The goal is a feature that is **fail-safe inside out**: testers cover everything a user can do through the screen; developers cover everything a direct or manipulated request can do.

**Two kinds, one pair of files per use case:**

| File | For | Covers | Contains |
|---|---|---|---|
| `NN-<use-case>.tester.md` | Testers: manual on a phone / tablet, or UI automation | Normal flows and every edge case a user can reach through the POS app (and the super-admin panel, if in scope): signing in as owner / waiter / chef, empty and long inputs, double taps, no network, app killed mid-flow, restaurant pending approval, phone and tablet sizes, feedback states | Precondition, plain steps, the **exact text** on screen and the visible outcome. No endpoints, request bodies, HTTP codes or SQL. |
| `NN-<use-case>.developer.md` | Developers: Postman / curl, SQL (runner to be decided — see GETTING-STARTED) | Input the app can't produce: no, malformed or revoked token, another restaurant's IDs, a waiter / chef token on an owner action, extra or forbidden fields (`restaurantId`, `role`, `price`…), garbage and out-of-range values, double submits, injection text, rate-limit abuse | Caller, the request and what is manipulated, then the expected HTTP status, `status` / `message`, and the **database state** after the call |

**System-only use cases** (no screen) have only a developer file; the coverage table says `Tester: n/a — no UI`.

`test-cases/README.md` holds the shared fixtures, the coverage matrix, the developer endpoint checklist and how to run the developer cases.

**IDs:**
- `<CODE>-<NN>-T<nn>` for tester cases and `<CODE>-<NN>-D<nn>` for developer cases, for example `TBL-02-T03`, `TBL-02-D05`.
- IDs are stable and never reused. A test that no longer applies stays in the file, marked `Removed YYYY-MM-DD: <reason>`.

**Coverage rules:**
- Every acceptance criterion and alternative flow is covered by at least one test case, tester or developer.
- Every endpoint a use case touches has developer cases for each category below, or the endpoint checklist marks the category `n/a` with a reason. Never silently skipped.

| Category | What it proves |
|---|---|
| Authentication | No token, a malformed token, a revoked (logged-out) token, a token of a deleted user → rejected with 401; nothing written. Public endpoints say `n/a` with the reason. |
| Account state | A token of a user who was switched off (`users.status = 0`), or of a restaurant that is pending, rejected (`isApproved`) or inactive (`restaurants.status = 0`), is refused as the spec defines. See *Known gaps* in `project-overview.md`. |
| Restaurant isolation | IDs of another restaurant's rows (zone, table, item, order, KOT…), sent with a valid token of restaurant A, are rejected or return nothing; nothing written; no data of the other restaurant leaks. This includes foreign keys inside the body (a `zoneId` or `categoryId` of restaurant B). |
| Role | A waiter or chef token calls an owner-only action directly (and vice versa where the spec restricts it). Expected result as the spec defines it — see *Known gaps*. |
| Super-admin boundary | A POS app token cannot reach `/admin/*` panel actions; the panel's session cannot be used on `/api/*` unless the spec says so. Write `n/a` when the feature has no panel page. |
| Input validation | Missing, empty, whitespace-only, over-long, wrong-type, negative / zero / huge numbers, malformed UUIDs, unknown, deleted or inactive references → rejected with 422 and a clear message. |
| Business-rule bypass | Calls the app never makes (actions it hides or disables, values it never sends — a price, a status jump, a KOT on a billed order) are still refused or neutralised by the server. |
| Over-posting | Extra or forbidden fields (`restaurantId`, `userId`, `role`, `createdBy`, `isDelete`, totals, prices…) in the body are ignored. Identity and restaurant come from the token; prices and totals come from the database. |
| Concurrency / idempotency | Double submits, parallel edits and races (two waiters on one table, two payments for one bill) leave consistent data and no duplicates. |
| Rate limiting | Throttled endpoints return 429 after the limit and nothing is written. Write `n/a` when the endpoint has no limiter and the spec doesn't ask for one. |
| Injection / encoding | SQL, script and Unicode text (including Indian scripts and emoji) are stored and returned as plain data. |
| Error hygiene | Failures use the standard response and never expose stack traces, SQL, exception text (`error` field) or internal names. |

**Expected results:**
- Both kinds state the **exact message text**.
- Developer cases also state the HTTP status and the database state. Use the real shapes in [API conventions](./project-overview.md#api-conventions). A rejected request leaves no trace.
- Test data is described by fixture name and state ("REST-A, approved, with an active zone ZN-A1 and its table TB-A1"), never by hard-coded IDs.

**Results are not recorded here.** Execution results (actual, Pass / Fail / Not run, date, build) go in `laravel-pos-api-dev_guid/specs/<feature>/test-results.md`, keyed by test ID.

**API-level developer cases come after the API design.** They need endpoint names, request fields and messages, so they are written once `api-contract.md` is approved.

**Test cases never introduce behaviour.** If a test needs an expected result the spec doesn't define, raise it as an open question in the use case first, and mark the test `Blocked: <question>`.

**Change the tests with the spec.** When a use case, data model, API contract or UI/UX changes, update the related test cases and the coverage matrix in the same change.

### Architecture Decision Records (`adr/`)

Numbered, immutable once accepted. Format:

```
# NNNN. Title

Status: Proposed | Accepted | Superseded by NNNN

## Context
## Decision
## Consequences
```

## When the conventions don't fit

If the template fights you, raise it: either deviate and note why, or propose a change to this file. Don't silently invent new section names or reorder things.
