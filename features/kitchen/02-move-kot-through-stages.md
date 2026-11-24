# 02 — Move a KOT through its stages

## Summary

From a KOT card the chef accepts or rejects a new KOT, then marks it Completed and Ready. The server stores the stage, the stage time and the chef. **Today the update never matches a row but still says it succeeded (KITCH-D1), and no stage rule is enforced (KITCH-D2) — both To fix.**

## Prerequisites

- **Backend:** `POST /api/chef/update-status/{kotId}` — [api-contract.md](./api-contract.md#post-apichefupdate-statuskotid)
- **Data:** `kots`

## Actors & preconditions

- **Actor:** chef. Today any role through the API; target chef only (KITCH-D2 / X-D1).
- **Preconditions:** a KOT card on screen. Today this can't happen in the app, because the list never loads (KITCH-D1).

## Main flow

1. On a `Pending` card the chef taps `Accept` (sends status 1) or `Reject` (status 2).
2. On a `Preparing` card the button is `Mark Completed` (status 3); on a `Completed` card, `Mark Ready` (status 4). A `Ready` card has no button.
3. While one change is being sent, every card's buttons are disabled; the other cards are dimmed.
4. The app posts `{ "status": <n> }` to `/chef/update-status/<kotId>`.
5. **Target:** the server checks the role, finds the KOT by hex ID in the caller's restaurant, checks the value (0–4) and that the change is allowed, then sets `status`, `updated_at`, the new stage's time (keeping earlier ones), and `chefId` on accept / reject. It answers 200 `KOT status updated successfully` with `data.kotStatus` = the new label. **Today:** `UPDATE kots … WHERE kotId = '<path text>'` matches nothing, yet answers 200.
6. On success the app removes the card from the current tab; it appears in the next stage's tab (Rejected KOTs have no tab).

## Alternative flows

- **Hex `kotId` (what the app sends).** Today no row matches; 200 as in step 5 (KITCH-D1). *Target:* updated.
- **Raw ID** (the 16 bytes percent-encoded in the path). Today the row is updated, whatever restaurant it belongs to. *Target:* 404 `KOT not found` (not hex).
- **Another restaurant's KOT.** *Target:* 404 `KOT not found`.
- **A change outside the path** (Ready → Pending, Ready → Rejected, Pending → Ready…). Today accepted. *Target:* 409 `Invalid status change`; nothing written.
- **Status outside 0–4.** Today written (if matched) and then 500 `Something went wrong`. *Target:* 422 `The selected status is invalid.`; nothing written.
- **`status` missing or not an integer.** 422 with the first validation message.
- **Failure in the app.** The card stays; no message is shown (the error is kept in the store only).

## Acceptance criteria

1. *(Target, KITCH-D1)* Given KOT-P (Pending, REST-A) and its hex ID, when CHF-A posts status 1, then 200 `KOT status updated successfully`, `kotStatus` `Preparing`, and KOT-P is status 1 with `chefId` CHF-A and `preparing_at` set. (Today: KOT-P unchanged.)
2. *(Target, KITCH-D2)* Given KOT-P moved to Completed and then Ready, then `preparing_at` and `completed_at` are both still set.
3. *(Target, KITCH-D2)* Given KOT-B (REST-B), when CHF-A posts any status, then 404 `KOT not found` and KOT-B is unchanged.
4. *(Target, KITCH-D2)* Given a Ready KOT, when status 0 is posted, then 409 `Invalid status change`; given status 9, then 422.
5. *(Target)* Given the app shows KOT-P in `New`, when the chef taps `Accept`, then the card leaves `New` and shows in `Preparing` with `Mark Completed`.

## Platform notes

### POS API

`StaffController::updateKOTStatus` → `StaffService::updateKOTStatus`. Today the path parameter is used as is (no `hex2bin`). The label comes from `$statusMap[$status]`; a missing key raises a PHP warning, which Laravel turns into an exception.

### Data — MySQL

`kots.status` is `tinyint(1)`, which still holds −128…127. `chefId` is `binary(16)`.

### POS app

`useKot.handleUpdateStatus` / `handleRejectStatus` → `updateKotStatusThunk`. The app types `kotId` as a number but passes on whatever the list gave it (hex once KITCH-D1 is fixed).

### Super-admin panel

None.

### Integrations

None. (The WhatsApp message on status change belongs to `orders/update-status` — DLV.)

## Configuration

None.

## Out of scope

- Picking up a Ready KOT — WAIT.
- Cancelling or editing items on a KOT — not offered.

## Open questions

- None open. Related decisions: KITCH-D1, KITCH-D2.
