---
name: initiate-game-auth-callback
description: >-
  Implement Appcharge Initiate Game Auth callback — Game Redirect Login only,
  deepLink and accessToken.
metadata:
  author: Appcharge
  version: 0.2.0
  tags: [appcharge, callback, webstore, authentication, game-redirect]
---

# Initiate Game Auth Callback

Implement the Initiate Game Auth callback. Official spec: https://docs.appcharge.com/api-reference/webstore/player-authentication/initiate-game-auth-callback.md

**Scope:** Game Redirect Login only. Other login methods use `authenticate-player-callback` only.

## When to Use

- Publisher enables Game Redirect Login and needs the initiation endpoint
- User mentions deep link auth, `accessToken` for OTP flow, or initiate game auth

## When NOT to Use

- SSO, username/password, or OTP verification after the game returns — use `authenticate-player-callback`
- Grant award or personalize callbacks

## Workflow

Complete **Phase 1 (Research)** before writing any implementation code.

### Phase 1 — Research (required)

#### 1.1 Project structure and conventions

- Detect language, framework, package manager, and HTTP server.
- Map project layout: routes/controllers, auth/session modules, deep-link or token utilities, config.
- Note naming conventions for handlers, routes, token storage, and env vars.
- Identify dependencies for HTTP, JSON, crypto, and short-lived token/session storage.

#### 1.2 Existing Appcharge integration

Search the codebase for Appcharge-related code before adding anything new:

- `authenticate-player-callback` handler and shared Appcharge middleware/utils (OTP flow must share `accessToken` storage)
- Signature verification, publisher token, main-key config
- Existing deep-link builders or pending-auth session stores
- Env vars or config for callback URLs and secrets

**Reuse** existing helpers; wire `accessToken` persistence so `authenticate-player-callback` can validate `otp.accessToken`.

#### 1.3 Test conventions

- Find the test framework and how HTTP handlers and session/token stores are tested.
- Locate existing callback or auth tests as templates.
- Note how mock signed requests and error status codes are asserted.

#### 1.4 Fetch official docs (required)

Run the `curl` commands in skill-local references only (these ship with the skill when installed via `npx skills add`; repo-root `docs/` is not available in client projects):

- [references/api-contract.md](references/api-contract.md) — endpoint request/response contract
- [references/secure-communication.md](references/secure-communication.md) — signature and token verification

Implement from the **fetched markdown only**.

### Phase 2 — Implementation

Apply findings from Phase 1 throughout:

1. **Confirm feature** — Game Redirect Login is enabled; otherwise stop and use authenticate-only flow.
2. **Secure ingress** — Per fetched secure-communication spec; use existing Appcharge middleware if present.
3. **Add route** — `POST`; align path with repo conventions (default suggestion: `/callbacks/initiate-game-auth`).
4. **Handler**
   - Parse `{ device, date }`
   - Create short-lived `accessToken` and game `deepLink` (include token/key query params per your game's convention)
   - Persist pending auth session keyed by `accessToken` using the project's existing session/token store
5. **Respond** — `200` + `{ "deepLink", "accessToken" }`; map verification failures to 400/401/403 with `{ "error": "..." }` per fetched docs
6. **Tests** — Add unit tests using project conventions from §1.3:
   - Valid signature + 200 body with `deepLink` and `accessToken`
   - Invalid signature → documented 400/401/403
   - Missing required fields → documented error
7. **Chain** — Document that `accessToken` must match `otp.accessToken` in `authenticate-player-callback`

## Flow

```text
Web store → Initiate Game Auth (this) → deepLink → Game → OTP → Authenticate Player
```

## Related skills

- `authenticate-player-callback` — completes login with `otp.playerCode` + `otp.accessToken`
