# CLAUDE.md — spec-reading rules for the WhizEats POS Lite implementation repos

> **Where this file goes:** these sections are merged into the `CLAUDE.md` of `C:\whizeats-lite\laravel-pos-api-dev_guid` (which has none yet — create it) and `C:\whizeats-lite\POS_APP-master` (add a "Specs" section). Keep each repo's own project-specific sections; this file adds the spec-reading rules and general behaviour.
>
> The specs live in `C:\whizeats-lite\App-Specs` (`../App-Specs/` from both repos). That folder has its own `CLAUDE.md`, which governs **writing or editing specs**.

Behavioral guidelines to reduce common LLM coding mistakes. They bias toward caution over speed; for trivial tasks, use judgment.

## 1. Read the Specs First

**Specs are the source of truth. Read them before forming any plan.**

Before starting any feature task:
- Read `../App-Specs/README.md` and `../App-Specs/project-overview.md` (platform decisions, API conventions, known gaps, glossary) if you haven't already in this session
- Read `../App-Specs/features/<feature>/README.md` and the relevant use-case file(s), including the Platform-notes sub-section for the layer you touch (**POS API**, **Data — MySQL**, **POS app**, **Super-admin panel**, **Integrations**)
- Read `api-contract.md`, `data-model.md`, `ui-ux.md` and the feature's `test-cases/` for the layers you touch

If a spec is missing, ambiguous, or contradicts another spec, stop and ask. Do not infer.

## 2. Think Before Coding

- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them — don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.

## 3. Simplicity First

Minimum code that solves the problem. No features, abstractions or configurability beyond what the spec asks for.

## 4. Surgical Changes

- Touch only what the approved plan names. Match existing style.
- Don't "improve" adjacent code, comments or formatting; mention unrelated issues instead.
- Remove only the orphans your own change created.

## 5. Goal-Driven Execution

Turn each task into a verifiable goal tied to test IDs from `../App-Specs/features/<feature>/test-cases/`, run them, and record actual results in `laravel-pos-api-dev_guid/specs/<feature>/test-results.md`. Never mark a test Pass that was not executed.

## 6. Plan First — Never Execute Without Explicit Approval

- Present the spec changes and the plan (`laravel-pos-api-dev_guid/specs/<feature>/plan.md`), then wait for an explicit "go" before editing application code, app files, migrations or `db_script/`.
- Approval of a plan covers only the steps in it.

## 7. Always Ask Before Creating a New Branch

Default to the current branch. Ask before any branch-creating command.
