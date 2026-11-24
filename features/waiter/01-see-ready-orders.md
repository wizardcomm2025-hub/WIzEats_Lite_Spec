# 01 — See ready orders

## Summary

The waiter opens **Ready Orders** and sees the KOTs the kitchen has marked Ready and nobody has picked up yet, with the KOT number, the place, the items and the note. **Today the list is always empty in the app (WAIT-D1, KITCH-D1 — To fix).**

## Prerequisites

- **Backend:** `POST /api/waiter/ready-kots` — [api-contract.md](./api-contract.md#post-apiwaiterready-kots)
- **Data:** `kots`, `order_items`, `orders`, `tables`, `zones`, `rooms`, `floors`

## Actors & preconditions

- **Actor:** waiter. Today any role through the API; target waiter only (WAIT-D2 / X-D1).
- **Preconditions:** signed in as a waiter.

## Main flow

1. The waiter opens the drawer → `Operations` → `Ready Orders` (phone), or the `Ready` rail button (tablet).
2. The app posts `/waiter/ready-kots` (no body) and polls it every 15 s while the screen is open and the app is in the foreground. Pull down asks again with a spinner.
3. The server selects the restaurant's KOTs (through `order_items.restaurantId`) with status 4 and `isPickedUp` 0, newest first, one entry per KOT with its items.
4. **Target:** 200 with hex IDs, `kotNumber` and `orderNumber`; one green card per KOT — `KOT #<kotNumber>`, `READY`, age, place (`<zone>  ·  <table>`, `Room: <room>` or `Takeaway`), items, note, `Delivered to Customer` — and the count in the header and on the `Pickup` tab. **Today:** a non-empty result is a 500 (WAIT-D1); the app keeps showing what it had (empty).

## Alternative flows

- **No ready KOT.** 200 `No ready KOTs found`, `data: []`. The app shows `No Pending Pickups` · `Kitchen hasn't marked any orders as ready yet. New ones appear here automatically.`
- **Server error.** No message; the list stays as it was.
- **Another restaurant's ready KOTs.** Not listed.
- **Chef or owner token.** Today answered as for a waiter. *Target:* 403.

## Acceptance criteria

1. Given REST-A has no Ready KOT, when WTR-A opens Ready Orders, then `No Pending Pickups` shows and the header count is 0.
2. *(Target, WAIT-D1)* Given the chef marked a Table 1 KOT Ready, when the list refreshes, then a card shows `KOT #<n>`, `READY`, `Main Hall  ·  Table 1`, its items and note, and the header count is 1. (Today: 500, and `No Pending Pickups`.)
3. Given only REST-B has a Ready KOT, then REST-A's request answers 200 with `data: []`.

## Platform notes

### POS API

`StaffController::getReadyKots` → `StaffService::getReadyKots`. No request validation. Same joins as the chef list ([KITCH 01](../kitchen/01-see-kots-by-stage.md)); `status` is sent as the number 4, not a label.

### Data — MySQL

Read only.

### POS app

`waiterDashboard/WaiterOrderScreen.tsx`, `fetchReadyKots` thunk, `usePolling` (15 s). Card title `KOT #<kotId>` today (target `KOT #<kotNumber>`); place `<zone>  ·  <table>`, `Room: <room>` or `Takeaway`.

### Super-admin panel

None.

### Integrations

None.

## Configuration

Polling interval 15 s.

## Out of scope

- Moving KOTs to Ready — KITCH.

## Open questions

- None open. Related decisions: WAIT-D1, WAIT-D2, KITCH-D1.
