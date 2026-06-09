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

Complete **Phase 1 (Research)** before writing any implementation code.

### Phase 1 — Research (required)

#### 1.1 Project structure and conventions

- Detect language, framework, package manager, and HTTP server (e.g. Go/Gin, Python/FastAPI, Node/NestJS, Java/Spring, C#/ASP.NET, Ruby/Rails, PHP/Laravel).
- Map project layout: where routes/controllers, middleware, services, DTOs, and config live.
- Note naming conventions for handlers, routes, env vars, and error responses.
- Identify dependencies already used for HTTP, JSON parsing, crypto, and auth.

#### 1.2 Existing Appcharge integration

Search the codebase for Appcharge-related code before adding anything new:

- Other callback handlers (grant-award, personalize, initiate-game-auth)
- Signature verification, `x-publisher-token` validation, main-key config
- Shared middleware, utils, or services under names like `appcharge`, `publisher`, `callback`, `webstore`
- Env vars or config keys for `APPCHARGE_PUBLISHER_TOKEN`, `APPCHARGE_MAIN_KEY`, callback URLs

**Reuse** existing helpers and patterns; do not duplicate verification or config loading.

#### 1.3 Test conventions

- Find the test framework and runner (e.g. `go test`, `pytest`, `jest`, `vitest`, `JUnit`, `xUnit`).
- Locate where unit vs integration tests live and how HTTP handlers are tested (mocks, test clients, fixtures).
- Find existing callback or webhook tests to mirror structure and assertion style.
- Note how signatures, headers, and auth failures are tested in this repo.

#### 1.4 Fetch official docs (required)

Run the `curl` commands in skill-local references only (these ship with the skill when installed via `npx skills add`; repo-root `docs/` is not available in client projects):

- [references/api-contract.md](references/api-contract.md) — endpoint request/response contract
- [references/secure-communication.md](references/secure-communication.md) — signature and token verification

Implement from the **fetched markdown only**; do not rely on cached or assumed API shapes.

### Phase 2 — Implementation

Apply findings from Phase 1 throughout:

1. **Secure ingress** — Per fetched secure-communication spec; plug into existing Appcharge middleware/utils if present.
2. **Add route** — `POST`; align path with repo conventions (default suggestion: `/callbacks/authenticate-player`).
3. **Branch on `authMethod`** (delegate to existing auth/session services):
   - `google` / `facebook` / `apple`: validate `token` with IdP or your token store
   - `userToken`: map token → player
   - `userPassword`: verify `userName` + `password`
   - `otp`: verify `otp.playerCode` + `otp.accessToken` (after Game Redirect flow)
4. **Success response** — `200`, `status: "valid"`, `publisherPlayerId`, `playerName`, `playerProfileImage` (empty string if none), optional `sessionMetadata`, optional `playerOverrideCountry`
5. **Failure response** — Non-valid `status` with `publisherErrorMessageType`, `publisherErrorMessage`, `publisherErrorMessageTitle` per fetched docs
6. **Tests** — Add unit tests using the project's test framework and patterns from §1.3:
   - Happy path per relevant `authMethod`
   - Bad credentials / unknown player
   - Invalid or missing signature → documented status code
   - Reuse existing test helpers for signing mock requests if available
7. **Dashboard** — Register callback URL; ensure token/main key env vars match project config conventions

## Related skills

- `initiate-game-auth-callback` — Game Redirect Login only; runs before OTP authenticate
- `personalize-webstore-callback` — uses `publisherPlayerId` as `playerId`
- `grant-award-callback` — receives `sessionMetadata` from successful auth
