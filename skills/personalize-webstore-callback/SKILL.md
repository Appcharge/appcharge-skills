---
name: personalize-webstore-callback
description: >-
  Implement Appcharge Personalize Web Store callback — offers, balances,
  segments per player.
metadata:
  author: Appcharge
  version: 0.2.0
  tags: [appcharge, callback, webstore, personalization]
---

# Personalize Web Store Callback

Implement the Personalize Web Store callback. Official spec: https://docs.appcharge.com/api-reference/webstore/personalization/personalize-webstore-callback.md

## When to Use

- Web store needs dynamic offers, balances, segments, or theme per player
- User mentions personalize webstore, store sync, or 5-minute player refresh

## When NOT to Use

- Grant award, authenticate player, or initiate game auth endpoints

## Workflow

Complete **Phase 0** and **Phase 1** before writing any implementation code.

### Phase 0 — Confirm with user (required)

Ask the user explicitly. If research already suggests an answer, propose it and ask to confirm or override. **Do not implement until answered.**

**Routing and secrets (all callbacks)**

1. Which **route path** should host this webhook? (suggestion: `/callbacks/personalize-webstore`)
2. Which **file** registers HTTP routes for this service?
3. Which **env var or config key** stores the Appcharge **main key** (webhook signature signing secret)?
4. Which **env var or config key** stores the **publisher token** (`x-publisher-token`)?

**Domain mapping (this callback)**

5. Which **model and field** does incoming `playerId` map to? (Same ID returned as `publisherPlayerId` from authenticate.)
6. Which **services or queries** load player **balances** (`publisherProductId` + `quantity`)?
7. Which **services or queries** load eligible **offers** (`publisherOfferId`, `productsSequence`)?
8. Where do **segments**, **attributes**, and **storeTheme** asset IDs come from?

### Phase 1 — Research (required)

#### 1.1 Project structure and conventions

- Detect language, framework, catalog/segmentation modules, caching, config.

#### 1.2 Existing Appcharge integration

Search for other callbacks, signature verification, personalization/catalog code. **Reuse** before adding new code.

#### 1.3 Test conventions

- Find test framework, handler test patterns, existing callback/catalog tests.

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

After verification, parse request `{ "playerId": "..." }`:

| Step | Action |
|------|--------|
| Resolve player | Look up by Phase 0 Q5 mapping; if missing → `status: "invalid"` |
| Load balances | Map wallet/inventory to `balances[]` via Phase 0 Q6 |
| Load offers | Map catalog to `offers[]` + `productsSequence` via Phase 0 Q7 |
| Segments & attributes | Populate from Phase 0 Q8 sources |
| Theme | Map asset IDs to `storeTheme` fields |
| Session passthrough | Set `sessionMetadata` for Grant Award when enabled |

Response shape (version 2):

- `version: 2`
- `status: "valid"` when player exists; `"invalid"` when not
- Required fields per fetched docs: `balances`, `offers`, `segments`, etc.

**Performance** — Called on login, post-purchase, and every ~5 minutes; avoid N+1; reuse existing caches.

#### Tests

- Valid vs invalid signature
- Known player → `status: "valid"` with required fields
- Unknown `playerId` → `status: "invalid"`

#### Dashboard

Register full callback URL; align env vars with Dashboard → Integration.

## Related skills

- `authenticate-player-callback` — provides `publisherPlayerId` used as `playerId` here
- `grant-award-callback` — receives `sessionMetadata` from this callback when enabled
