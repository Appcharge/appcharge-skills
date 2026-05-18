---
name: authenticate-player-callback
description: >-
  Implements the Appcharge Authenticate Player callback in Go or Python.
  Verifies signature and publisher token, validates SSO/OTP/password logins,
  returns publisherPlayerId, playerName, and sessionMetadata. Use for web
  store login callback, player authentication endpoint, or authenticate-player-callback docs.
metadata:
  author: Appcharge
  version: 0.1.0
  tags: [appcharge, callback, webstore, authentication, golang, python]
---

# Authenticate Player Callback

Implement the [Authenticate Player Callback](https://docs.appcharge.com/api-reference/webstore/player-authentication/authenticate-player-callback).

## When to Use

- Web store login must be validated against the game/account backend
- User mentions authenticate player, SSO login callback, OTP login, or `publisherPlayerId`

## When NOT to Use

- Game Redirect **initiation** (deep link + access token before OTP) — use `initiate-game-auth-callback`
- Grant award or personalize endpoints

## Workflow

1. **Detect stack** — Go or Python; reuse existing auth/session services.
2. **Read contract** — [references/api-contract.md](references/api-contract.md)
3. **Secure ingress** — [secure communication](../../../docs/callbacks/secure-communication.md)
4. **Add route** — `POST` e.g. `/callbacks/authenticate-player`
5. **Branch on `authMethod`**
   - `google` / `facebook` / `apple`: validate `token` with IdP or your token store
   - `userToken`: map token → player
   - `userPassword`: verify `userName` + `password`
   - `otp`: verify `otp.playerCode` + `otp.accessToken` (after Game Redirect flow)
6. **Success response** — `200`, `status: "valid"`, `publisherPlayerId`, `playerName`, `playerProfileImage` (empty string if none), optional `sessionMetadata`, optional `playerOverrideCountry`
7. **Failure response** — Non-valid `status` with `publisherErrorMessageType`, `publisherErrorMessage`, `publisherErrorMessageTitle` per docs
8. **Tests** — Per `authMethod` happy path; bad credentials; invalid signature
9. **Dashboard** — Register callback URL; ensure token/main key env vars are set

## Related skills

- `initiate-game-auth-callback` — Game Redirect Login only; runs before OTP authenticate
- `personalize-webstore-callback` — uses `publisherPlayerId` as `playerId`
- `grant-award-callback` — receives `sessionMetadata` from successful auth
