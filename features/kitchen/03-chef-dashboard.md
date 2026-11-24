# 03 — Chef dashboard

## Summary

From the drawer (Kitchen → `Dashboard`) the chef sees **Today's Overview** with three counts: Received, Accepted and Rejected. Today the counts come from KOTs this chef last changed (KITCH-D3), and since status changes never reach the database (KITCH-D1), they stay at 0. Target: the restaurant's KOTs of the day.

## Prerequisites

- **Backend:** `POST /api/chef/dashboard` — [api-contract.md](./api-contract.md#post-apichefdashboard)
- **Data:** `kots`, `order_items`

## Actors & preconditions

- **Actor:** chef. Today any role through the API; target chef only (KITCH-D2 / X-D1).
- **Preconditions:** signed in.

## Main flow

1. The chef opens the drawer and taps `Dashboard` under `Kitchen`.
2. If `role` is exactly `Chef` (target: `chef` in any case — KITCH-D3), the app posts `/chef/dashboard` (once per sign-in; pull to refresh asks again).
3. The server counts today's KOTs (`created_at` today, server time). Today: those whose `chefId` is the caller — `total_received` all, `total_accepted` status 1, `total_rejected` status 2. *Target:* the restaurant's KOTs — `total_received` all, `total_accepted` status 1, 3 or 4, `total_rejected` status 2.
4. The app shows three cards: `Received`, `Accepted`, `Rejected`. The header shows the user's name and `CHEF-<userId>`.

## Alternative flows

- **Role saved as `chef` / `CHEF`** (panel or API). Today the app treats the user as a waiter here: it calls the waiter dashboard and shows `Total Orders` (WAIT), header `WAIT-<userId>`. *Target:* the chef cards.
- **A KOT moved to Completed or Ready.** Today it leaves Accepted; target it stays in Accepted.
- **Server error.** 500 `Something went wrong`; the app keeps its loader or last counts.

## Acceptance criteria

1. Given CHF-A and no KOTs today, when the dashboard opens, then `Received 0`, `Accepted 0`, `Rejected 0`.
2. *(Target, KITCH-D3)* Given REST-A has three KOTs today — one Pending, one Ready, one Rejected — then `Received 3`, `Accepted 1`, `Rejected 1`, whoever changed them. (Today: only KOTs CHF-A last changed count.)
3. *(Target, KITCH-D3)* Given a chef saved as `chef`, then the app shows the chef cards. (Today: the waiter dashboard.)

## Platform notes

### POS API

`StaffController::getChefDashboardDetails` → `StaffService::getChefKotStats`. Today no restaurant filter; it filters by `chefId` only.

### Data — MySQL

Read only.

### POS app

`waiterDashboard/waiterScreen.tsx` (`isChef = user?.role === 'Chef'`, case-sensitive, unlike the rest of the app), `StatsOverview`.

### Super-admin panel

None.

### Integrations

None.

## Configuration

None.

## Out of scope

- The waiter dashboard — WAIT.

## Open questions

- None open. Related decision: KITCH-D3.
