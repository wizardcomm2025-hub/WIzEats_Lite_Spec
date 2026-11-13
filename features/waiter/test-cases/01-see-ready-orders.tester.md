# 01 — See ready orders — tester cases

Use case: [01-see-ready-orders.md](../01-see-ready-orders.md). Screen text: [ui-ux.md §1–2](../ui-ux.md#2-ready-orders). Fixtures: [README.md](./README.md#fixtures).

Signed in as WTR-A.

| ID | Covers | Steps | Expected |
|---|---|---|---|
| WAIT-01-T01 | AC1 | Phone: drawer → `Operations` → `Ready Orders`. REST-A has no Ready KOT. | `Ready Orders` · `Orders prepared by kitchen`, count bubble `0`; `Pickup` tab: `No Pending Pickups` · `Kitchen hasn't marked any orders as ready yet. New ones appear here automatically.` |
| WAIT-01-T02 | AC1 | Tablet: tap the `Ready` rail button. | Same screen as T01. |
| WAIT-01-T03 | AC2 (WAIT-D1) | Ask a developer to set KOT-R to Ready (SQL). Stay on `Pickup` for 20 s. | Still `No Pending Pickups`; count bubble `0`; no `Pickup` badge; no error message. |
| WAIT-01-T04 | AC2 | After T03 pull down on `Pickup`. | Spinner shows and stops; still empty; no error message. |
| WAIT-01-T05 | AC1 | Tap `Delivered`. | `No Deliveries Yet` · `Orders you deliver will appear here.` |
