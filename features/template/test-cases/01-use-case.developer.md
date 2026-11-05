# 01 — <!-- verb phrase --> — developer cases

Use case: [01-use-case.md](../01-use-case.md). Contract: [api-contract.md](../api-contract.md). Shared response shapes: [project-overview.md → API conventions](../../../project-overview.md#api-conventions). Fixtures and standard headers: [README.md](./README.md#fixtures).

Every case states the caller, what is manipulated, the expected HTTP status, `status` / `message`, and the database state after the call. A rejected request leaves no trace. Worked example: [features/auth/test-cases/](../../auth/test-cases/).

All requests: `POST /api/<!-- module/action -->`.

| ID | Category | Caller | Request body / manipulation | Expected | Database after |
|---|---|---|---|---|---|
| {{FEATURE_CODE}}-01-D01 | — (happy path) | <!-- OWN-A token --> | <!-- valid body --> | <!-- 200 · status: true · message: "…" --> | <!-- row created / changed --> |

<!-- Minimum set per endpoint (see conventions.md for each category):
     no token; malformed token; revoked token (after logout);
     token of a switched-off user; token of a pending / rejected restaurant;
     REST-B's IDs (path and body foreign keys) with OWN-A's token;
     WTR-A / CHF-A token on an owner action;
     missing / empty / whitespace / 10 000-char / wrong-type / negative / malformed-UUID fields;
     forged restaurantId, userId, role, createdBy, isDelete, price, totals in the body;
     same request twice in parallel;
     SQL / <script> / Unicode text round-trips as plain data;
     a 500 never returns an `error` field or exception text.
     Undecided expected result → raise an open question and mark the case "Blocked: <question>". -->
