---
name: initiate-game-auth-callback
description: >-
  Implements the Appcharge Initiate Game Auth callback for Game Redirect Login
  in Go or Python. Returns deepLink and accessToken after signature verification.
  Use only for game redirect login, initiate game authentication, or
  initiate-game-auth-callback docs — not for SSO or password login.
metadata:
  author: Appcharge
  version: 0.1.0
  tags: [appcharge, callback, webstore, authentication, game-redirect, golang, python]
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

1. **Confirm feature** — Game Redirect Login is enabled; otherwise stop and use authenticate-only flow.
2. **Detect stack** — Go or Python.
3. **Fetch official docs** (required) — Run the `curl` commands in [references/api-contract.md](references/api-contract.md) and [secure communication](../../../docs/callbacks/secure-communication.md); implement from the fetched markdown only.
4. **Secure ingress** — Per fetched secure-communication spec.
5. **Add route** — `POST` e.g. `/callbacks/initiate-game-auth`
6. **Handler**
   - Parse `{ device, date }`
   - Create short-lived `accessToken` and game `deepLink` (include token/key query params per your game's convention)
   - Persist pending auth session keyed by `accessToken` for later OTP validation
7. **Respond** — `200` + `{ "deepLink", "accessToken" }`; map verification failures to 400/401/403 with `{ "error": "..." }` per docs
8. **Tests** — Valid signature + 200 body; invalid signature → 400; missing fields → 403
9. **Chain** — Document that `accessToken` must match `otp.accessToken` in authenticate-player callback

## Flow

```text
Web store → Initiate Game Auth (this) → deepLink → Game → OTP → Authenticate Player
```

## Related skills

- `authenticate-player-callback` — completes login with `otp.playerCode` + `otp.accessToken`
