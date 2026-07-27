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

Complete **Phase 0** and **Phase 1** before writing any implementation code.

### Phase 0 — Confirm with user (required)

Ask the user explicitly. If research already suggests an answer, propose it and ask to confirm or override. **Do not implement until answered.**

**Routing and secrets (all callbacks)**

1. Which **route path** should host this webhook? (suggestion: `/callbacks/grant-award`)
2. Which **file** registers HTTP routes for this service?
3. Which **env var or config key** stores the Appcharge **main key** (webhook signature signing secret)?
4. Which **env var or config key** stores the **publisher token** (`x-publisher-token`)?

**Domain mapping (this callback)**

5. Which **model and field** does incoming `playerId` map to in your domain?
6. Which **service** grants inventory/currency from `products[]` (by `sku` / `amount`)?
7. Which **field or transaction ID** should be returned as `publisherPurchaseId` after a successful grant?
8. How should **`orderId`** / **`purchaseId`** be used for idempotency (existing table, unique constraint)?

### Phase 1 — Research (required)

#### 1.1 Project structure and conventions

- Detect language, framework, routes/controllers, inventory/wallet modules, config.

#### 1.2 Existing Appcharge integration

Search for other callbacks, signature verification, shared middleware/utils. **Reuse** before adding new code.

#### 1.3 Test conventions

- Find test framework, handler test patterns, existing webhook/fulfillment tests.

#### 1.4 Fetch official docs (required)

Run `curl` from skill-local references (shipped with `npx skills add`):

- [references/api-contract.md](references/api-contract.md)
- [references/secure-communication.md](references/secure-communication.md)

Implement from the **fetched markdown only**.

### Phase 2 — Implementation

Use Phase 0 answers and Phase 1 findings throughout.

#### Signature verification

Follow [references/secure-communication.md](references/secure-communication.md):

1. Register route as `POST` at the user-confirmed path in the user-confirmed router file.
2. Read **raw body** before JSON parsing.
3. Verify `signature` (HMAC-SHA256, `t=<ms>.<rawBody>`, ~5 min replay window) using the user-confirmed main-key env var.
4. Validate `x-publisher-token` against the user-confirmed publisher-token env var.

#### Payload handling

After verification, parse `PlayerOrderReportRequest` and delegate to domain services:

| Payload field | Handler action |
|---------------|----------------|
| `playerId` | Resolve player via Phase 0 Q5 model/field; reject unknown player → `400` |
| `products[]` | Grant each product (`sku`, `amount`) via Phase 0 Q6 service |
| `orderId`, `purchaseId` | Idempotent key per Phase 0 Q8 — skip duplicate grants |
| `action` | `purchase` vs `bonus` — route to correct grant logic if they differ |
| `awardFlow` | Respect `manual_retry` when applicable |
| `sessionMetadata` | Parse JSON string if the game stores session context |
| `bundleId`, `sku` | Offer reference for logging/validation |

On success → `200` + `{ "publisherPurchaseId": "<Phase 0 Q7 value>" }`.

On client error (invalid player, bad SKU) → `400` + `{ "publisherErrorMessage": "..." }`.

On transient failure → `500` + `{ "publisherErrorMessage": "..." }`.

**Critical:** success responses must include a valid `publisherPurchaseId` or Appcharge marks the award failed.

#### Tests

- Valid signature + happy path returns `publisherPurchaseId`
- Invalid signature → `401`/`403`
- Idempotent re-delivery of same `orderId`
- Unknown `playerId` → `400`

#### Dashboard

Register full callback URL; align env vars with Dashboard → Integration.

## Handler sketch

```text
verify(headers, rawBody) → parse(order) → grant(playerId, products, orderId) → { publisherPurchaseId }
```

## Related skills

- `authenticate-player-callback` — supplies `playerId` and `sessionMetadata`
- `personalize-webstore-callback` — optional; uses same `sessionMetadata` when enabled
