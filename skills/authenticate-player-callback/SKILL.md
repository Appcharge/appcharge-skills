---
name: authenticate-player-callback
description: >-
  Implement Appcharge Authenticate Player callback — web store login,
  SSO/OTP/password, publisherPlayerId.
metadata:
  author: Appcharge
  version: 0.2.0
  tags: [appcharge, callback, webstore, authentication]
---

# Authenticate Player Callback

Implement the Authenticate Player callback. Official spec: https://docs.appcharge.com/api-reference/webstore/player-authentication/authenticate-player-callback.md

## When to Use

- Web store login must be validated against the game/account backend
- User mentions authenticate player, SSO login callback, OTP login, or `publisherPlayerId`

## When NOT to Use

- Game Redirect **initiation** (deep link + access token before OTP) — use `initiate-game-auth-callback`
- Grant award or personalize endpoints

## Workflow

Complete **Phase 0** and **Phase 1** before writing any implementation code.

### Phase 0 — Confirm with user (required)

Ask the user explicitly. If research already suggests an answer, propose it and ask to confirm or override. **Do not implement until answered.**

**Routing and secrets (all callbacks)**

1. Which **route path** should host this webhook? (suggestion: `/callbacks/authenticate-player`)
2. Which **file** registers HTTP routes for this service?
3. Which **env var or config key** stores the Appcharge **main key** (webhook signature signing secret)?
4. Which **env var or config key** stores the **publisher token** (`x-publisher-token`)?

**Domain mapping (this callback)**

5. Which **model and field** hold the player's stable ID returned as `publisherPlayerId`?
6. Which **model and field** hold the display name returned as `playerName`?
7. Which **auth service or function** validates each `authMethod` the project supports (`google`/`facebook`/`apple`, `userToken`, `userPassword`, `otp`)?

### Phase 1 — Research (required)

#### 1.1 Project structure and conventions

- Detect language, framework, package manager, and HTTP server.
- Map project layout: routes/controllers, middleware, services, DTOs, config.
- Note naming conventions for handlers, routes, env vars, and error responses.

#### 1.2 Existing Appcharge integration

Search for other callbacks, signature verification, shared middleware/utils, and configured secrets. **Reuse** before adding new code.

#### 1.3 Test conventions

- Find test framework, handler test patterns, and existing webhook/signature tests.

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
2. Ensure middleware reads **raw body** before JSON parsing.
3. Verify `signature` (HMAC-SHA256, `t=<ms>.<rawBody>`, ~5 min replay window) using the user-confirmed main-key env var.
4. Validate `x-publisher-token` against the user-confirmed publisher-token env var.
5. Reject unsigned/invalid requests before handler logic.

#### Payload handling

After verification, parse the request body and branch on `authMethod`:

| `authMethod` | Request fields | Handler action |
|--------------|----------------|----------------|
| `google` / `facebook` / `apple` | `token`, `appId`, `date` | Validate SSO token via auth service from Phase 0 Q7 |
| `userToken` | `token` | Map token → player |
| `userPassword` | `userName`, `password` | Verify credentials |
| `otp` | `otp.playerCode`, `otp.accessToken` | Validate against pending session from `initiate-game-auth-callback` |

Map the authenticated player to response fields using Phase 0 Q5–Q6:

- `publisherPlayerId` ← user's model/field for stable player ID
- `playerName` ← user's model/field for display name
- `playerProfileImage` ← profile image URL or `""`
- `sessionMetadata` ← optional object passed through to Grant Award
- `playerOverrideCountry` ← optional ISO country override

**Success** — `200`, `status: "valid"`, required fields above.

**Failure** — non-valid `status` with `publisherErrorMessageType`, `publisherErrorMessage`, `publisherErrorMessageTitle` per fetched docs.

#### Tests

Using project test conventions:

- Valid signature + happy path per supported `authMethod`
- Bad credentials
- Invalid/missing signature
- Mock signed requests using the confirmed main key

#### Dashboard

Register the full callback URL (base + confirmed path); ensure env vars match Dashboard → Integration.

## Related skills

- `initiate-game-auth-callback` — Game Redirect Login only; runs before OTP authenticate
- `personalize-webstore-callback` — uses `publisherPlayerId` as `playerId`
- `grant-award-callback` — receives `sessionMetadata` from successful auth
