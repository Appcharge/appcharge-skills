# Initiate Game Auth Callback — API contract

Docs: https://docs.appcharge.com/api-reference/webstore/player-authentication/initiate-game-auth-callback

**Game Redirect Login only.** Appcharge calls when the player starts game-redirect authentication from the web store.

## Request

`POST /{your-initiate-game-auth-path}`

### Body

| Field | Type | Notes |
|-------|------|-------|
| `device` | enum | `DESKTOP` (PC/laptop) \| `APPCHARGE` (mobile) |
| `date` | string (date-time) | ISO 8601 UTC |

## Responses

| Status | Body |
|--------|------|
| 200 | `{ "deepLink": "<url>", "accessToken": "<token>" }` — both **required** |
| 400 | Invalid signature (docs example: `{ "error": "Invalid signature" }`) |
| 401 | `{ "error": "Unauthorized" }` |
| 403 | `{ "error": "Parameters not correct" }` |

`deepLink` redirects the player into the game to complete auth. `accessToken` is later sent back via `otp.accessToken` on the [Authenticate Player Callback](https://docs.appcharge.com/api-reference/webstore/player-authentication/authenticate-player-callback).
