# 01 — See KOTs by stage

## Summary

The chef sees the restaurant's KOTs in four tabs: New, Preparing, Completed and Ready. Each card shows the KOT number, its place (zone and table, or room), the items with quantities, the note and how long ago it was sent. **Today any non-empty list fails with a 500, so the tabs always show `No Orders Here` (KITCH-D1, To fix).**

## Prerequisites

- **Backend:** `POST /api/chef/pending-kots` — [api-contract.md](./api-contract.md#post-apichefpending-kots)
- **Data:** `kots`, `order_items`, `orders`, `tables`, `zones`, `rooms`, `floors`

## Actors & preconditions

- **Actor:** chef. Today any role through the API; target chef only (KITCH-D2 / X-D1).
- **Preconditions:** signed in; the restaurant has KOTs (sent through ORD).

## Main flow

1. A user whose role is `chef` in any letter case signs in. The app opens **Kitchen Orders** (`Manage and track food preparation`) instead of Home. The footer buttons are greyed out and do nothing.
2. The `New` tab is selected. The app posts `{ "status": 0 }`.
3. The server selects the KOTs whose `order_items.restaurantId` is the caller's restaurant and whose `status` matches, newest first, one entry per KOT with its items.
4. **Target:** the server answers 200 with hex IDs, `kotNumber` and `orderNumber`, and the app lists one card per KOT: `KOT #<kotNumber>`, status pill, age, place (`Zone : <zone>  <table>`, `Room: <room>` or `N/A`), items, note. **Today:** if any KOT matches, the reply is a 500 (KITCH-D1) and the tab stays empty.
5. The app polls the same tab every 15 s while the screen is open and the app is in the foreground. Pull to refresh asks again.
6. Changing tab posts that tab's status: `Preparing` 1, `Completed` 3, `Ready` 4. There is no tab for Rejected (2).

## Alternative flows

- **No KOT in that stage.** 200 `No pending KOTs found`, `data: []`. The app shows `No Orders Here` / `No orders in this section. Pull down to refresh.`
- **Server error.** The app keeps what it had (empty after a tab change) and shows no message.
- **`status` missing or not an integer.** 422 with the first validation message. *Target:* also outside 0–4 → 422 `The selected status is invalid.`
- **Waiter or owner token.** Today answered as for a chef. *Target:* 403.

## Acceptance criteria

1. Given a chef with no KOTs in the restaurant, when Kitchen Orders opens, then the `New` tab shows `No Orders Here`.
2. *(Target, KITCH-D1)* Given REST-A has one Pending KOT for Table 1, when the chef opens `New`, then one card shows `KOT #<n>`, `Pending`, `Zone : Main Hall  Table 1`, its items and note. (Today: 500, and the tab shows `No Orders Here`.)
3. Given REST-B has Pending KOTs and REST-A has none, when REST-A's chef asks for status 0, then 200 with `data: []`.
4. *(Target, KITCH-D2)* Given a waiter or owner token, then 403. (Today: answered as for a chef.)

## Platform notes

### POS API

`StaffController::getPendingKots` (validated by `updateKOTStatusRequest`: `status` required integer) → `StaffService::getPendingKots`. Left joins from `kots` to `order_items`, `orders`, `tables`, `zones`, `rooms`, `floors`; the `WHERE` on `oi.restaurantId` drops KOTs without lines. Today no `bin2hex` on `kotId`, and the raw `tableId` / `roomId` are selected too.

### Data — MySQL

Read only.

### POS app

`KOT/kotScreen.tsx`, `useKot` (polling with `usePolling`, 15 s), `kot.slice` (the list is cleared on a tab change; a reply for another tab is ignored). The tab badges are always 0, so they never show. The card title is `KOT #<kotId>` today (target `KOT #<kotNumber>` — KITCH-D1).

### Super-admin panel

None.

### Integrations

None.

## Configuration

Polling interval 15 s (`DEFAULT_INTERVAL_MS` in `usePolling.ts`).

## Out of scope

- Ready KOTs for the waiter — WAIT.

## Open questions

- None open. Related decisions: KITCH-D1, KITCH-D2.
