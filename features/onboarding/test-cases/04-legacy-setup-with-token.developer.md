# 04 — Legacy setup with a token — developer cases

Use case: [04-legacy-setup-with-token.md](../04-legacy-setup-with-token.md). Contract: [api-contract.md](../api-contract.md#post-apirestaurantssetup). Fixtures: [README.md](./README.md#fixtures).

All requests: `POST /api/restaurants/setup` with `Authorization: Bearer <token>` and body "full" **without** `phone` unless stated, using different values from the restaurant's current ones (name `Changed Name`, etc.). Snapshot the caller's restaurant, config, zones, tables, channels and user row before each case. Most cases are **accepted risks** (ONB-D1).

| ID | Category | Caller | Manipulation | Expected | Database after |
|---|---|---|---|---|---|
| ONB-04-D01 | — (happy path) | OWN-LEG | — | 200 · `message: "Restaurant onboarded successfully"` · `data.isApproved: "Pending"` · new `token` · `mustChangePin` = OWN-LEG's value · admin e-mail; **no** PIN e-mail | REST-LEG: details set, `isOnboardingDone = 1`, `isApproved` still 0; config with a new code; zone Main Hall + 4 tables; 4 channels; OWN-LEG `name` = owner name; 1 new token. REST-LEG now appears in the panel. |
| ONB-04-D02 | Role / Account state (**accepted risk**) | WTR-A (REST-A Accepted, has zones and tables) | — | 200 · `data.isApproved: "Accepted"` | REST-A: name, address, owner name, e-mail replaced; `restaurantContact` = WTR-A's phone; `createdBy = 'waiter'`; config flags replaced, `restaurantCode` unchanged; REST-A's first zone renamed `Main Hall` and active; no tables added; channels not duplicated; `isApproved` still 1. WTR-A `name` = the owner name sent. |
| ONB-04-D03 | Over-posting | OWN-A | add `"phone": "<PH-NEW>"`, then `"phone": "<OWN-B phone>"` | 200 both | no new user or restaurant; OWN-B unchanged; REST-A `restaurantContact` = OWN-A's phone |
| ONB-04-D04 | Account state | NOREST | — | 404 · `{ "status": false, "message": "Restaurant not found", "data": null, "token": null, "mustChangePin": null }` | nothing written |
| ONB-04-D05 | Authentication | (a) revoked token, body without `phone` (b) revoked token, body with a fresh PH-NEW | (a) 422 `Phone number is required` (b) 200 as [01-D01](./01-register-restaurant.developer.md) — treated as a brand-new registration | (a) nothing (b) a new restaurant + owner |
| ONB-04-D06 | Restaurant isolation / Over-posting | OWN-A | add `"restaurantId": "<REST-B id>"` | 200 | only REST-A changed; REST-B unchanged |
| ONB-04-D07 | Concurrency / idempotency | OWN-A | send the same call twice in a row, then twice in parallel | all 200; each issues a token | still one config, one Main Hall zone, its tables unchanged, four channels — no duplicates; one token per call |
| ONB-04-D08 | Business rule | OWN-B (REST-B's first zone has tables) | — | 200 · `data.tables` = that zone's existing tables | no tables added |
| ONB-04-D09 | Input validation | OWN-A | body without `restaurantEmail` | 422 · `The restaurant email field is required.` | nothing written |
| ONB-04-D10 | Account state (**accepted risk**) | OWN-R (Rejected) | — | 200 · `data.isApproved: "Rejected"` | REST-R details replaced; still Rejected |
| ONB-04-D11 | Rate limiting | OWN-A | 11 calls in one minute from one IP | 1–10: 200 · 11: 429 | 10 overwrites, 10 tokens |
