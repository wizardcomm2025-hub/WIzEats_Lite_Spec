# 01 — <!-- TODO: verb phrase, e.g. "Add a table to a zone" -->

## Summary

<!-- 3–5 sentences: the actor, the action, the outcome, and why it matters. No implementation detail. -->

## Prerequisites

<!-- Bullets linking to where each is defined, or "none". -->
- **Backend:** <!-- endpoint(s) in [api-contract.md](./api-contract.md) -->
- **Data:** <!-- tables in [data-model.md](./data-model.md) -->
- **Depends on:** <!-- sibling use cases / features, or none -->

## Actors & preconditions

- **Actor:** <!-- owner / waiter / chef / public customer / super admin / system -->
- **Trigger:** <!-- what the actor does -->
- **Preconditions:** <!-- signed in; restaurant approved and onboarding done; channel enabled in restaurant config; … -->

## Main flow

1. <!-- What happens, present tense, 5–10 steps -->

## Alternative flows

- **Validation failure.** <!-- … -->
- **Duplicate.** <!-- … -->
- **Not found / another restaurant's ID.** <!-- … -->
- **Inactive or deleted reference.** <!-- … -->
- **Wrong role.** <!-- what a waiter / chef sees or gets -->
- **Double submit.** <!-- … -->
- **No network.** <!-- what the app shows; is anything queued? -->

## Acceptance criteria

1. Given <!-- … -->, when <!-- … -->, then <!-- … -->.

## Platform notes

### POS API

<!-- Endpoint(s), middleware (auth:sanctum + restaurant.tenant / throttle / public), FormRequest rules, role check, restaurantId and userId from the token, every message + HTTP status. Link api-contract.md. -->

### Data — MySQL

<!-- Tables read / written, restaurantId filter, isActive / isDelete, status values, consistency rules. Link data-model.md. -->

### POS app

<!-- Screen, roles that reach it, states, toasts, offline behaviour. Link ui-ux.md. -->

### Super-admin panel

<!-- Pages that show or change the same data, or "Not touched." -->

### Integrations

<!-- Mail, Google Sheets error log, image storage, import / export, printer, or "Not touched." -->

## Configuration

<!-- What is data-driven (restaurant_configs switches, master tables, .env) vs hard-coded. Describe what is configurable, not values. -->

## Out of scope

- <!-- Link where it is covered, or mark TBD / future. -->

## Open questions

<!-- {{FEATURE_CODE}}-01-Q1 … Resolved: append "_Resolved YYYY-MM-DD: answer._" -->
