# CLAUDE.md — spec-reading rules for the WhizEats POS Lite implementation repos

> **Where this file goes:** these sections are merged into the `CLAUDE.md` of the API repo (`C:\xampp82\htdocs\laravel-pos-api\api`) and the app (`D:\whizeats_Lite_Spec\frontend\POS_APP-kot`). **Merged on 2026-11-13** into the repos' earlier locations; **not present at the current locations** (checked 2026-11-24) — merge again ([GETTING-STARTED item 9](../GETTING-STARTED.md)). Both repos now carry a "Specs" section adapted to each repo. Edit those copies in step with this file. Keep each repo's own project-specific sections; this file adds the spec-reading rules and general behaviour.
>
> The specs live in `D:\whizeats_Lite_Spec\spec` (from the app repo: `../../spec/`; from the API repo: use the absolute path). That folder has its own `CLAUDE.md`, which governs **writing or editing specs**.

Behavioral guidelines to reduce common LLM coding mistakes. They bias toward caution over speed; for trivial tasks, use judgment.

## 1. Read the Specs First

**Specs are the source of truth. Read them before forming any plan.**

Before starting any feature task:
- Read `<specs>/README.md` and `<specs>/project-overview.md` (platform decisions, API conventions, known gaps, bug-fix index, glossary) if you haven't already in this session — `<specs>` = `D:\whizeats_Lite_Spec\spec`
- Read `<specs>/features/<feature>/README.md` (including its **Decisions**) and the relevant use-case file(s), including the Platform-notes sub-section for the layer you touch (**POS API**, **Data — MySQL**, **POS app**, **Super-admin panel**, **Integrations**)
- Read `api-contract.md`, `data-model.md`, `ui-ux.md` and the feature's `test-cases/` for the layers you touch

If a spec is missing, ambiguous, or contradicts another spec, stop and ask. Do not infer.

**Fixing a known bug.** Each bug is a decision `<CODE>-Dn` in the feature README. Build its *Target* exactly (status codes and messages as written); the tests tagged `Fails until <CODE>-Dn is fixed` must then pass. Don't change behaviour that a decision marks *Accepted*. When done, propose the spec change that sets the decision to *Fixed in code <date>* and removes the tags.

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

Turn each task into a verifiable goal tied to test IDs from `<specs>/features/<feature>/test-cases/`, run them, and record actual results in `laravel-pos-api/api/specs/<feature>/test-results.md`. Never mark a test Pass that was not executed.

## 6. Plan First — Never Execute Without Explicit Approval

- Present the spec changes and the plan (`laravel-pos-api/api/specs/<feature>/plan.md`), then wait for an explicit "go" before editing application code, app files, migrations or `db_script/`.
- Approval of a plan covers only the steps in it.

## 7. Always Ask Before Creating a New Branch

Default to the current branch. Ask before any branch-creating command.
