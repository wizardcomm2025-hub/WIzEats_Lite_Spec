# 02 — Move a KOT through its stages

## Summary

From a KOT card the chef accepts or rejects a new KOT, then marks it Completed and Ready. The server stores the stage, the stage time and the chef. **Today the update never matches a row but still says it succeeded (KITCH-D1), and no stage rule is enforced (KITCH-D2).**

## Prerequisites

- **Backend:** `POST /api/chef/update-status/{kotId}` — [api-contract.md](./api-contract.md#post-apichefupdate-statuskotid)
- **Data:** `kots`

## Actors & preconditions

- **Actor:** chef. Any role through the API (KITCH-D2).
- **Preconditions:** a KOT card on screen. In the app this can't happen today, because the list never loads (KITCH-D1), so this use case is reachable only through the API.

## Main flow

1. On a `Pending` card the chef taps `Accept` (sends status 1) or `Reject` (status 2).
2. On a `Preparing` card the button is `Mark Completed` (status 3); on a `Completed` card, `Mark Ready` (status 4). A `Ready` card has no button.
3. While one change is being sent, every card's buttons are disabled; the other cards are dimmed.
4. The app posts `{ "status": <n> }` to `/chef/update-status/<kotId>`.
5. The server runs `UPDATE kots SET status, updated_at, chefId = caller, preparing_at / rejected_at / completed_at WHERE kotId = '<path text>'`. It then answers 200 `KOT status updated successfully` with `data.kotStatus` = the label of the new status.
6. On success the app removes the card from the current tab.

## Alternative flows

- **Hex `kotId` (what the app would send).** No row matches; 200 as in step 5 (KITCH-D1).
- **Raw ID** (the 16 bytes percent-encoded in the path). The row is updated, whatever restaurant it belongs to (KITCH-D2).
- **Any stage jump** (Ready → Pending, Ready → Rejected). Accepted (KITCH-D2).
- **Status outside 0–4 but within −128…127.** The row (if matched) is written, then 500 `Something went wrong`.
- **Status outside −128…127.** For a matched row, the database rejects it: 500 `Something went wrong`, nothing written. For an unmatched ID: the same 500 (the label lookup fails).
- **`status` missing or not an integer.** 422 with the first validation message.
- **Failure in the app.** The card stays; no message is shown (the error is kept in the store only).

## Acceptance criteria

1. Given KOT-P (Pending, REST-A) and its hex ID, when CHF-A posts status 1, then 200 `KOT status updated successfully`, `kotStatus` `Preparing`, and KOT-P is still status 0 with no `chefId` (KITCH-D1).
2. Given KOT-P's raw ID in the path, when CHF-A posts status 1, then KOT-P is status 1, `chefId` = CHF-A, `preparing_at` set, `rejected_at` / `completed_at` NULL.
3. Given KOT-B (REST-B) by raw ID, when CHF-A posts status 2, then KOT-B is Rejected (KITCH-D2).
4. Given status 9 on KOT-P by raw ID, then 500 `Something went wrong` and KOT-P's status is 9.

## Platform notes

### POS API

`StaffController::updateKOTStatus` → `StaffService::updateKOTStatus`. The path parameter is used as is (no `hex2bin`). The label comes from `$statusMap[$status]`; a missing key raises a PHP warning, which Laravel turns into an exception.

### Data — MySQL

`kots.status` is `tinyint(1)`, which still holds −128…127. `chefId` is `binary(16)`.

### POS app

`useKot.handleUpdateStatus` / `handleRejectStatus` → `updateKotStatusThunk`. The app types `kotId` as a number but passes on whatever the list gave it.

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

- None open.
