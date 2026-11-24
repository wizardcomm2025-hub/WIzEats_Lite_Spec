# 03 — Approve or reject a restaurant — developer cases

Use case: [03-approve-or-reject.md](../03-approve-or-reject.md). Contract: [api-contract.md](../api-contract.md#post-adminupdate-statusrestaurantid-web). Fixtures and how to call the panel: [README.md](./README.md#panel-requests-use-case-03).

All requests: `POST /admin/update-status/<id>` as form data, with ADMIN-1's session cookies and a valid `_token` unless stated. "Redirect back" = 302 to the referring page.

| ID | Category | Request / manipulation | Expected | Database after |
|---|---|---|---|---|
| ONB-03-D01 | — (happy path) | REST-P, `isApproved=1` | redirect back · e-mail `Your restaurant has been approved` to REST-P's address · panel row shows `Approved` · OWN-P loginPin → `isApproved: "Accepted"` | REST-P `isApproved = 1`, `updated_at` now |
| ONB-03-D02 | — (happy path) | REST-P2, `isApproved=2` | redirect back · e-mail `Your restaurant registration was not approved` · row shows `Rejected` | REST-P2 `isApproved = 2` |
| ONB-03-D03 | Business rule | REST-P (now 1), `isApproved=1` again | redirect back · **no** e-mail | unchanged value |
| ONB-03-D04 | Business rule (reversal kept on purpose — ONB-D2) | REST-P (Accepted), `isApproved=2` | redirect back · rejected e-mail sent | REST-P `isApproved = 2` |
| ONB-03-D05 | Business rule (reversal kept on purpose — ONB-D2) | REST-P, `isApproved=0` | redirect back · no e-mail · row shows `Accept` / `Reject` again | `isApproved = 0` |
| ONB-03-D06 | Input validation — *Fails until ONB-D2 is fixed* | REST-P (Pending), `isApproved=7`; then `isApproved=-1`; repeat both with `Accept: application/json` | redirect back with validation errors · no e-mail · row still shows `Accept` / `Reject` · JSON: 422 `The selected is approved is invalid.` (today: stored; row shows neither badge nor buttons) | REST-P `isApproved = 0` |
| ONB-03-D07 | Input validation / Error hygiene — *Fails until ONB-D2 is fixed* | REST-P, (a) no `isApproved` (b) `isApproved=abc`; repeat with `Accept: application/json` | redirect back with validation errors, **no** 500 · JSON: 422 (a) `The is approved field is required.` (b) `The selected is approved is invalid.` (today: 500 error page) | REST-P unchanged |
| ONB-03-D08 | Over-posting | REST-P2 (Pending), `isApproved=1&status=0&restaurantEmail=x@example.test&restaurantName=Hacked` | redirect back | only `isApproved` changes; status, e-mail, name unchanged |
| ONB-03-D09 | Input validation — *Fails until ONB-D2 is fixed* | (a) `<id>` = 32 hex chars of no restaurant (b) `<id>` = `xyz`, both `isApproved=1` | 404 · no e-mail (today: redirect back, looks like success) | no restaurant changed |
| ONB-03-D10 | Authentication | no panel session (fresh cookie jar with a guest `_token` from `GET /`) | 302 to `/`; with `Accept: application/json`: 401 `{ "message": "Unauthenticated." }` | unchanged |
| ONB-03-D11 | Authentication | ADMIN-1 session, **no** `_token` (or a wrong one) | 419 `Page Expired`; with `Accept: application/json`: 419 `{ "message": "CSRF token mismatch." }` | unchanged |
| ONB-03-D12 | Super-admin boundary | no session; `Authorization: Bearer <OWN-A token>` (a) without `_token` (b) with a guest `_token` | (a) 419 (b) 302 to `/` — a POS token never authenticates the panel | unchanged |
| ONB-03-D13 | Integration failure | REST-P2 reset to 0; MAIL-DOWN; `isApproved=1` | redirect back; no error shown | `isApproved = 1` |
| ONB-03-D14 | Rate limiting | 30 requests in a minute | all processed; no 429 (no limit on this route) | last value stored |
| ONB-03-D15 | Business rule | `GET /admin/update-status/<REST-P id>` | 405 Method Not Allowed | unchanged |
| ONB-03-D16 | Account state | REST-P-NOMAIL, `isApproved=1` | redirect back; no e-mail | `isApproved = 1` |
| ONB-03-D17 | Concurrency | REST-P reset to 0; `isApproved=1` and `isApproved=2` posted in parallel | both redirect back; final value 1 **or** 2; up to two e-mails, possibly one of each (accepted: no state check, decisions may change — ONB-D2) | `isApproved` ∈ {1, 2} |
| ONB-03-D18 | Injection / Restaurant isolation — *Fails until ONB-D2 is fixed* | `<id>` = `' OR 1=1 --` (URL-encoded), `isApproved=2` | 404 · no e-mail (today: redirect back) | **no** restaurant changed |
