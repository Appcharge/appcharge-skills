# Authenticate Player Callback — API contract

Docs: https://docs.appcharge.com/api-reference/webstore/player-authentication/authenticate-player-callback

Appcharge calls when a player logs into the web store.

## Request

`POST /{your-authenticate-player-path}`

### Body

| Field | Type | Notes |
|-------|------|-------|
| `authMethod` | enum | `facebook`, `apple`, `google`, `userToken`, `userPassword`, `otp` (also used for Pre-Auth / Game Redirect) |
| `token` | string \| null | SSO / Player ID login |
| `date` | string (date-time) | UTC login attempt |
| `appId` | string | Dashboard app ID; `"NA"` if none |
| `userName` | string \| null | Username/password login |
| `password` | string \| null | |
| `otp` | object \| null | `{ playerCode, accessToken }` — Pre-Auth / Game Redirect only |
| `os` | enum | `ios`, `android`, `web` |
| `utmSource` | string | Optional |
| `utmMedium` | string | Optional |
| `utmCampaign` | string | Optional |
| `sessionId` | string | |
| `playerLocation` | object | `{ countryCode2, state? }` ISO-3166 alpha-2 |

## Response (200)

Required on success: `status`, `publisherPlayerId`, `playerName`.

| Field | Type | Notes |
|-------|------|-------|
| `status` | enum | Must be `valid` for success |
| `publisherPlayerId` | string | Your canonical player ID |
| `playerName` | string | Display name |
| `playerProfileImage` | string | URL or `""` for default avatar |
| `sessionMetadata` | object | Echoed in Grant Award / Personalize when enabled |
| `playerOverrideCountry` | string | Optional ISO-3166 alpha-2 pricing override |
| `publisherErrorMessageType` | enum | `none`, `plainText`, `enrichedText` — failure UX |
| `publisherErrorMessage` | string | Shown on failure |
| `publisherErrorMessageTitle` | string | Failure title |

### Login failure example

```json
{
  "status": "Invalid",
  "publisherErrorMessageType": "plainText",
  "publisherErrorMessage": "Lost internet connection.",
  "publisherErrorMessageTitle": "Whoops! Login Failed."
}
```

Success requires `status: "valid"` (lowercase per OpenAPI enum).
