---
name: personalize-webstore-callback
description: >-
  Implements the Appcharge Personalize Web Store callback in a Go or Python
  publisher server. Verifies headers and signature, loads player segments,
  balances, and offer overrides, returns personalization JSON (version 2).
  Use for web store personalization, player sync callback, offersOrder,
  segments, or docs.appcharge.com personalize-webstore-callback.
metadata:
  author: Appcharge
  version: 0.1.0
  tags: [appcharge, callback, webstore, personalization, golang, python]
---

# Personalize Web Store Callback

Implement the Personalize Web Store callback. Official spec: https://docs.appcharge.com/api-reference/webstore/personalization/personalize-webstore-callback.md

## When to Use

- Web store needs dynamic offers, balances, segments, or theme per player
- User mentions personalize webstore, store sync, or 5-minute player refresh

## When NOT to Use

- Grant award, authenticate player, or initiate game auth endpoints

## Workflow

1. **Detect stack** — Go or Python; match existing HTTP and catalog/segmentation modules.
2. **Fetch official docs** (required) — Run the `curl` commands in [references/api-contract.md](references/api-contract.md) and [secure communication](../../../docs/callbacks/secure-communication.md); implement from the fetched markdown only.
3. **Secure ingress** — Per fetched secure-communication spec.
4. **Add route** — `POST` e.g. `/callbacks/personalize-webstore`.
5. **Handler**
   - Input: `{ "playerId": "..." }`
   - Load player profile, wallets (`balances`), eligible offers (`offers` + `productsSequence`), segments, A/B `attributes`, theme asset IDs
   - Set `version: 2`, `status: "valid"` when the player exists; `"invalid"` when not
   - Populate `sessionMetadata` (passed through to Grant Award when enabled)
6. **Performance** — Called on login, post-purchase, and periodic sync; avoid unbounded N+1; cache catalog slices where the project already does.
7. **Tests** — Signature gate; valid player returns required fields; unknown player → `status: "invalid"` or documented error behavior.
8. **Config** — Register callback URL and secrets in Publisher Dashboard per fetched docs.

## Related skills

- `authenticate-player-callback` — provides `publisherPlayerId` used as `playerId` here
- `grant-award-callback` — receives `sessionMetadata` from this callback when enabled
