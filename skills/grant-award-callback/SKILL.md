---
name: grant-award-callback
description: >-
  Implements the Appcharge Grant Award callback endpoint in a Go or Python
  publisher server. Verifies x-publisher-token and signature, parses the order
  payload, grants inventory, and returns publisherPurchaseId. Use when adding
  grant award webhook, fulfillment callback, purchase completion handler, or
  docs.appcharge.com grant-award-callback integration.
metadata:
  author: Appcharge
  version: 0.1.0
  tags: [appcharge, callback, checkout, awards, golang, python]
---

# Grant Award Callback

Implement the [Grant Award Callback](https://docs.appcharge.com/api-reference/checkout/awards/grant-award-callback) on the publisher backend Appcharge calls after paid or free purchases.

## When to Use

- New integration or missing grant-award handler in a Go/Python service
- User mentions grant award, fulfillment callback, `publisherPurchaseId`, or post-purchase webhook

## When NOT to Use

- Authenticate player, personalize webstore, or initiate game auth — use those skills instead
- Client-side or Dashboard-only work

## Workflow

1. **Detect stack** — Prefer Go (`net/http`, Gin, Echo) or Python (FastAPI, Flask). Match existing routing, middleware, and config patterns.
2. **Read contract** — [references/api-contract.md](references/api-contract.md)
3. **Secure ingress** — Follow [secure communication](../../../docs/callbacks/secure-communication.md): raw body → verify `signature` → validate `x-publisher-token` → parse JSON.
4. **Add route** — `POST` only. Default path suggestion: `/callbacks/grant-award` (align with repo conventions if present).
5. **Business logic** — Idempotent grant by `orderId` / `purchaseId`:
   - Resolve `playerId`, credit `products`, persist publisher-side purchase ID
   - Handle `action` `purchase` vs `bonus`; respect `awardFlow` `manual_retry` if applicable
   - Parse `sessionMetadata` if the game stores JSON there
6. **Respond**
   - Success: `200` + `{ "publisherPurchaseId": "<your-transaction-id>" }`
   - Client errors (unknown player, invalid SKU): `400` + `publisherErrorMessage`
   - Transient failures: `500` + `publisherErrorMessage`
7. **Tests** — Unit-test signature verification and handler: valid signature + happy path; invalid signature → `401`/`403`; missing `publisherPurchaseId` on success path forbidden.
8. **Wire config** — Document `APPCHARGE_PUBLISHER_TOKEN`, `APPCHARGE_MAIN_KEY`, and the public callback URL for the Publisher Dashboard.

## Handler sketch

Delegate to existing inventory/wallet services; do not duplicate domain logic in the HTTP layer.

```text
verify(headers, rawBody) → parse(PlayerOrderReportRequest) → grant(order) → JSON 200
```

## Related skills

- `authenticate-player-callback` — supplies `playerId` and `sessionMetadata`
- `personalize-webstore-callback` — optional; uses same `sessionMetadata` when enabled
