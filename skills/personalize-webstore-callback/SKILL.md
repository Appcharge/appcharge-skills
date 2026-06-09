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

Complete **Phase 1 (Research)** before writing any implementation code.

### Phase 1 — Research (required)

#### 1.1 Project structure and conventions

- Detect language, framework, package manager, and HTTP server.
- Map project layout: routes/controllers, catalog/segmentation modules, caching layers, config.
- Note naming conventions for handlers, routes, DTOs, and env vars.
- Identify dependencies for HTTP, JSON, and any existing personalization or catalog services.

#### 1.2 Existing Appcharge integration

Search the codebase for Appcharge-related code before adding anything new:

- Other callback handlers and shared Appcharge middleware/utils
- Signature verification, publisher token, main-key config
- Existing personalization, segment, or catalog code the handler should call
- Env vars or config for callback URLs and secrets

**Reuse** existing helpers and domain services; do not duplicate verification or catalog logic.

#### 1.3 Test conventions

- Find the test framework and how HTTP handlers and service mocks are structured.
- Locate existing callback or catalog tests as templates.
- Note how signature failures and response shape assertions are done.

#### 1.4 Fetch official docs (required)

Run the `curl` commands in skill-local references only (these ship with the skill when installed via `npx skills add`; repo-root `docs/` is not available in client projects):

- [references/api-contract.md](references/api-contract.md) — endpoint request/response contract
- [references/secure-communication.md](references/secure-communication.md) — signature and token verification

Implement from the **fetched markdown only**.

### Phase 2 — Implementation

Apply findings from Phase 1 throughout:

1. **Secure ingress** — Per fetched secure-communication spec; use existing Appcharge middleware if present.
2. **Add route** — `POST`; align path with repo conventions (default suggestion: `/callbacks/personalize-webstore`).
3. **Handler** — Delegate to existing catalog/segmentation services:
   - Input: `{ "playerId": "..." }`
   - Load player profile, wallets (`balances`), eligible offers (`offers` + `productsSequence`), segments, A/B `attributes`, theme asset IDs
   - Set `version: 2`, `status: "valid"` when the player exists; `"invalid"` when not
   - Populate `sessionMetadata` (passed through to Grant Award when enabled)
4. **Performance** — Called on login, post-purchase, and periodic sync; avoid unbounded N+1; cache catalog slices where the project already does.
5. **Tests** — Add unit tests using project conventions from §1.3:
   - Signature gate (valid vs invalid)
   - Valid player returns required fields per fetched docs
   - Unknown player → `status: "invalid"` or documented error behavior
6. **Config** — Register callback URL and secrets in Publisher Dashboard using project config naming.

## Related skills

- `authenticate-player-callback` — provides `publisherPlayerId` used as `playerId` here
- `grant-award-callback` — receives `sessionMetadata` from this callback when enabled
