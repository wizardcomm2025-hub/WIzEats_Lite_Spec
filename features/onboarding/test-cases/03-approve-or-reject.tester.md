# 03 — Approve or reject a restaurant — tester cases

Use case: [03-approve-or-reject.md](../03-approve-or-reject.md). Screen text: [ui-ux.md §5–6](../ui-ux.md#5-panel--restaurant-list-approval-part). Fixtures: [README.md](./README.md#fixtures).

Run in a desktop browser, signed in to the super-admin panel as ADMIN-1. Testers read the restaurant inboxes.

| ID | Covers | Precondition | Steps | Expected |
|---|---|---|---|---|
| ONB-03-T01 | AC1 | REST-P Pending | Find REST-P; click `Accept`. | No confirmation; the list reloads; the row shows the green `Approved` badge and no buttons. REST-P's inbox: `Your restaurant has been approved` — `Good news! <name> has been approved. You can now log in…` |
| ONB-03-T02 | AC2 | REST-P2 Pending | Click `Reject`. | Row shows red `Rejected`. Inbox: `Your restaurant registration was not approved`. |
| ONB-03-T03 | AC3 | T01, T02 done | Look at both rows; refresh the page. | Only the badges; no `Accept` / `Reject` buttons, no way to change the decision. |
| ONB-03-T04 | AC6 | REST-LEG exists | Search the list for REST-LEG. | Not listed. |
| ONB-03-T05 | List | Several restaurants registered | Open the list; page through. | Newest first; 10 per page; each row shows name, `ID: <hex>`, owner, contact, address, `Active` / `Inactive`, approval. |
| ONB-03-T06 | AC7 | REST-P reset to Pending; mail down (ask a developer) | Click `Accept`. | Row shows `Approved`; no error on the page; no e-mail. |
| ONB-03-T07 | AC8 | REST-P Pending | Open the list; in another tab log out of the panel; back in the first tab click `Accept`. | Panel login page; REST-P still Pending. |
| ONB-03-T08 | AC8 | REST-P Pending | Leave the list open past the session lifetime (or clear the site cookies), then click `Accept`. | `Page Expired` (or the login page); REST-P still Pending. |
| ONB-03-T09 | No e-mail | REST-P-NOMAIL | Click `Accept`. | Row shows `Approved`; no e-mail sent anywhere. |
| ONB-03-T10 | Encoding | Restaurant registered as `রেস্তোরাঁ 🍛 Café` (ONB-01-T15) | Find it in the list; expand its details. | Name displays correctly. |
