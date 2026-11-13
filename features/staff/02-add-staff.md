# 02 — Add a staff member

## Summary

The owner taps `Add Staff`, enters a name and a phone number, picks Chef or Waiter, and taps `Create Staff`. The server creates an active user with the first PIN `123456` and a forced PIN change. The new staff member then signs in with their phone and `123456` (AUTH 02, AUTH 04).

## Prerequisites

- **Backend:** `POST /api/staff/create` — [api-contract.md](./api-contract.md#post-apistaffcreate)
- **Data:** `users` — [data-model.md](./data-model.md)
- **Depends on:** [01](./01-see-staff-list.md); AUTH for the first sign-in

## Actors & preconditions

- **Actor:** the owner. Through the API, any signed-in user (STAFF-D3).
- **Trigger:** taps `Add Staff`.
- **Preconditions:** fewer than one active chef plus one active waiter; the phone number has no account anywhere.

## Main flow

1. A bottom sheet `Add Staff` opens with:
   - `Name` (placeholder `Enter staff name`);
   - `Phone` (placeholder `Enter phone number`, phone keypad, max 10);
   - `Role`: two chips, `Chef` and `Waiter`. A role that already has an active member is greyed, marked `Already added`, and can't be picked.
2. On `Create Staff` the app checks:
   - name filled, else `Name is required`;
   - phone filled, else `Phone number is required`;
   - phone matches `^[6-9][0-9]{9}$`, else `Enter a valid 10-digit number starting with 6-9`;
   - a role picked, else `Please select a role`.

   Each message shows under its field, in red.
3. The app sends `{ name (trimmed), phone (as a number), role: "Chef" | "Waiter" }`.
4. The server validates the formats and creates the user:
   - the caller's restaurant;
   - `status = 1`;
   - PIN `123456` (hashed), `mustChangePin = 1`;
   - `createdBy` = the caller's role.
5. It answers 200 `Staff member created successfully`. The app shows the toast `Staff created successfully`, closes the sheet and reloads the list.
6. The new member signs in with the phone and `123456`, and is asked for a new PIN.

## Alternative flows

- **Phone already used** (in this restaurant or any other). 500 `Failed to create staff member`, shown as a toast; the sheet stays open with the values (STAFF-D3).
- **Name of spaces only.** It passes the app's check, is sent trimmed as `""`, and the server answers 422 `The name field is required.`, shown as a toast.
- **No network.** Toast `Failed to create staff`.
- **Double tap.** The button shows a spinner and is disabled while saving, so one request is sent.
- **Limit reached.** `Add Staff` is greyed. Through the API, a third active waiter can be created (STAFF-D3).
- **Role `owner` or any other text through the API.** Created as given (STAFF-D3). A user with role `owner` gets the full owner UI after sign-in. An unknown role gets no menus.
- **Someone else signs in first** with the phone and `123456`, and sets their own PIN (STAFF-D4).
- **Close the sheet.** The fields are cleared.

## Acceptance criteria

1. Given OWN-A with no active waiter, when `Waiter` is created with a new valid phone, then the toast `Staff created successfully` shows, the sheet closes, and the list shows the new row as `Active` with role `Waiter`.
2. Given (1), when the new waiter signs in with `123456`, then they must set a new PIN, then land on the waiter Home.
3. Given an active waiter exists, then the `Waiter` chip is greyed with `Already added`.
4. Given both roles are taken, then `Add Staff` is greyed.
5. Given each invalid input in step 2, then the matching message shows under the field and nothing is sent.
6. Given a phone already in use, then the toast `Failed to create staff member` shows and nothing is created.
7. Given the API is called by WTR-A with role `owner`, then the user is created with role `owner` (STAFF-D3).

## Platform notes

### POS API

- `StaffController::store` → `StaffService::create`.
- Rules: `name` required string max 255; `phone` required, regex `^[6-9][0-9]{9}$`; `role` required string.
- No uniqueness rule: the database's `users_phone_unique` key refuses a duplicate, which gives a 500.

### Data — MySQL

One `users` insert. A new `userId` (GUID), the caller's `restaurantId`, and the role as sent.

### POS app

- `AddStaffBottomSheet` checks the fields.
- `CreateStaffProfile` thunk; on success the screen refetches the list.
- `fullRoles` computes the taken roles from the list (active members only).

### Super-admin panel

`+ Add User` is shown but does nothing ([04](./04-edit-users-in-panel.md)).

### Integrations

None — no SMS or e-mail tells the staff member they were added.

## Configuration

`MAX_ACTIVE_PER_ROLE = 1` in the app (`staffLimits.ts`).

## Out of scope

- The first sign-in and PIN change — AUTH 02 / 04.

## Open questions

- None open. Related decisions: STAFF-D3, STAFF-D4.
