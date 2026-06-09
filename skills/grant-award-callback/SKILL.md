---
name: grant-award-callback
description: >-
  Implement Appcharge Grant Award callback — post-purchase fulfillment,
  publisherPurchaseId.
metadata:
  author: Appcharge
  version: 0.2.0
  tags: [appcharge, callback, checkout, awards]
---

# Grant Award Callback

Implement the Grant Award callback on the publisher backend Appcharge calls after paid or free purchases. Official spec: https://docs.appcharge.com/api-reference/checkout/awards/grant-award-callback.md

## When to Use

- New integration or missing grant-award handler in the publisher backend
- User mentions grant award, fulfillment callback, `publisherPurchaseId`, or post-purchase webhook

## When NOT to Use

- Authenticate player, personalize webstore, or initiate game auth — use those skills instead
- Client-side or Dashboard-only work

## Workflow

Complete **Phase 1 (Research)** before writing any implementation code.

### Phase 1 — Research (required)

#### 1.1 Project structure and conventions

- Detect language, framework, package manager, and HTTP server.
- Map project layout: routes/controllers, middleware, services, inventory/wallet modules, config.
- Note naming conventions for handlers, routes, env vars, and error responses.
- Identify dependencies for HTTP, JSON, crypto, and idempotent persistence.

#### 1.2 Existing Appcharge integration

Search the codebase for Appcharge-related code before adding anything new:

- Other callback handlers (authenticate-player, personalize, initiate-game-auth)
- Signature verification, `x-publisher-token` validation, main-key config
- Shared middleware, utils, or services for Appcharge callbacks
- Env vars or config for publisher token, main key, and registered callback URLs

**Reuse** existing helpers; do not duplicate verification or config loading.

#### 1.3 Test conventions

- Find the test framework, test file locations, and how HTTP handlers are exercised.
- Locate existing callback, webhook, or fulfillment tests as templates.
- Note how valid/invalid signatures and error responses are asserted.

#### 1.4 Fetch official docs (required)

Run the `curl` commands in skill-local references only (these ship with the skill when installed via `npx skills add`; repo-root `docs/` is not available in client projects):

- [references/api-contract.md](references/api-contract.md) — endpoint request/response contract
- [references/secure-communication.md](references/secure-communication.md) — signature and token verification

Implement from the **fetched markdown only**.

### Phase 2 — Implementation

Apply findings from Phase 1 throughout:

1. **Secure ingress** — Per fetched secure-communication spec: raw body → verify `signature` → validate `x-publisher-token` → parse JSON. Use existing Appcharge middleware if present.
2. **Add route** — `POST` only; align path with repo conventions (default suggestion: `/callbacks/grant-award`).
3. **Business logic** — Idempotent grant by `orderId` / `purchaseId`; delegate to existing inventory/wallet services:
   - Resolve `playerId`, credit `products`, persist publisher-side purchase ID
   - Handle `action` `purchase` vs `bonus`; respect `awardFlow` `manual_retry` if applicable
   - Parse `sessionMetadata` if the game stores JSON there
4. **Respond**
   - Success: `200` + `{ "publisherPurchaseId": "<your-transaction-id>" }`
   - Client errors (unknown player, invalid SKU): `400` + `publisherErrorMessage`
   - Transient failures: `500` + `publisherErrorMessage`
5. **Tests** — Add unit tests using project conventions from §1.3:
   - Valid signature + happy path returns `publisherPurchaseId`
   - Invalid signature → documented `401`/`403`
   - Idempotent re-delivery of same order
   - Missing `publisherPurchaseId` on success path forbidden
6. **Wire config** — Document callback URL and secrets using the project's config/env naming; align with Publisher Dashboard.

## Handler sketch

Delegate to existing inventory/wallet services; do not duplicate domain logic in the HTTP layer.

```text
verify(headers, rawBody) → parse(PlayerOrderReportRequest) → grant(order) → JSON 200
```

## Related skills

- `authenticate-player-callback` — supplies `playerId` and `sessionMetadata`
- `personalize-webstore-callback` — optional; uses same `sessionMetadata` when enabled
