# 03 — Chef dashboard

## Summary

From the drawer (Kitchen → `Dashboard`) the chef sees **Today's Overview** with three counts: Received, Accepted and Rejected. The counts come from KOTs this chef last changed (KITCH-D3), and since status changes never reach the database today (KITCH-D1), they stay at 0.

## Prerequisites

- **Backend:** `POST /api/chef/dashboard` — [api-contract.md](./api-contract.md#post-apichefdashboard)
- **Data:** `kots`

## Actors & preconditions

- **Actor:** chef. Any role through the API (KITCH-D2).
- **Preconditions:** signed in.

## Main flow

1. The chef opens the drawer and taps `Dashboard` under `Kitchen`.
2. If `role` is exactly `Chef`, the app posts `/chef/dashboard` (once per sign-in; pull to refresh asks again).
3. The server counts today's KOTs (`created_at` today, server time) whose `chefId` is the caller:
   - `total_received` = all of them,
   - `total_accepted` = those with status 1,
   - `total_rejected` = those with status 2.
4. The app shows three cards: `Received`, `Accepted`, `Rejected`. The header shows the user's name and `CHEF-<userId>`.

## Alternative flows

- **Role saved as `chef` / `CHEF`** (panel or API). The app treats the user as a waiter here: it calls the waiter dashboard and shows `Total Orders` (WAIT). The header shows `WAIT-<userId>`.
- **A KOT moved to Completed or Ready.** It stays in Received but leaves Accepted.
- **Another chef changes the KOT later.** It moves to their counts.
- **Server error.** 500 `Something went wrong`; the app keeps its loader or last counts.

## Acceptance criteria

1. Given CHF-A (role `Chef`) and any KOTs in REST-A, when the dashboard opens, then `Received 0`, `Accepted 0`, `Rejected 0` (KITCH-D1).
2. Given CHF-A changed KOT-P to status 1 by raw ID today, then `total_received` 1 and `total_accepted` 1; after changing it to 3, `total_received` 1 and `total_accepted` 0 (KITCH-D3).
3. Given a chef saved as `chef`, then the app shows the waiter dashboard (KITCH-D3).

## Platform notes

### POS API

`StaffController::getChefDashboardDetails` → `StaffService::getChefKotStats`. No restaurant filter; it filters by `chefId` only.

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

- None open.
