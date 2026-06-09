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

Complete **Phase 0** and **Phase 1** before writing any implementation code.

### Phase 0 — Confirm with user (required)

Ask the user explicitly. If research already suggests an answer, propose it and ask to confirm or override. **Do not implement until answered.**

**Routing and secrets (all callbacks)**

1. Which **route path** should host this webhook? (suggestion: `/callbacks/initiate-game-auth`)
2. Which **file** registers HTTP routes for this service?
3. Which **env var or config key** stores the Appcharge **main key** (webhook signature signing secret)?
4. Which **env var or config key** stores the **publisher token** (`x-publisher-token`)?

**Domain mapping (this callback)**

5. Where should pending **`accessToken`** sessions be stored so `authenticate-player-callback` can validate `otp.accessToken`?
6. How is the game **`deepLink`** URL built (base URL, query params, device-specific paths)?
7. Should **`desktopAutoRedirect`** be `true` or `false` for desktop players?

### Phase 1 — Research (required)

#### 1.1 Project structure and conventions

- Detect language, framework, auth/session modules, deep-link utilities, config.

#### 1.2 Existing Appcharge integration

Search for authenticate callback, shared middleware, token/session stores. **Reuse** and ensure OTP flow shares `accessToken` storage.

#### 1.3 Test conventions

- Find test framework, handler test patterns, existing auth/callback tests.

#### 1.4 Fetch official docs (required)

Run `curl` from skill-local references (shipped with `npx skills add`):

- [references/api-contract.md](references/api-contract.md)
- [references/secure-communication.md](references/secure-communication.md)

Implement from the **fetched markdown only**.

### Phase 2 — Implementation

Use Phase 0 answers and Phase 1 findings throughout.

**Confirm feature** — Game Redirect Login is enabled; otherwise stop.

#### Signature verification

Follow [references/secure-communication.md](references/secure-communication.md):

1. Register route as `POST` at the user-confirmed path in the user-confirmed router file.
2. Read **raw body** before JSON parsing.
3. Verify `signature` (HMAC-SHA256, `t=<ms>.<rawBody>`, ~5 min replay window) using the user-confirmed main-key env var.
4. Validate `x-publisher-token` against the user-confirmed publisher-token env var.

#### Payload handling

After verification, parse `{ "device", "date" }`:

| Step | Action |
|------|--------|
| Validate input | `device` is `DESKTOP` or `APPCHARGE`; `date` is ISO 8601 |
| Create `accessToken` | Generate short-lived token; persist in Phase 0 Q5 store |
| Build `deepLink` | Use Phase 0 Q6 URL builder; embed token/key params |
| Respond | `200` + `{ "deepLink", "accessToken", "desktopAutoRedirect" }` per Phase 0 Q7 |

Error mapping per fetched docs:

- Invalid signature → `400` + `{ "error": "Invalid signature" }`
- Bad publisher token → `401` + `{ "error": "Unauthorized" }`
- Missing/invalid fields → `403` + `{ "error": "Parameters not correct" }`

**Chain** — `accessToken` here must match `otp.accessToken` in `authenticate-player-callback`.

#### Tests

- Valid signature → `200` with `deepLink` and `accessToken`
- Invalid signature → `400`
- Missing fields → `403`

#### Dashboard

Register full callback URL; align env vars with Dashboard → Integration.

## Flow

```text
Web store → Initiate Game Auth (this) → deepLink → Game → OTP → Authenticate Player
```

## Related skills

- `authenticate-player-callback` — completes login with `otp.playerCode` + `otp.accessToken`
