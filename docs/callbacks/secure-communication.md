# Appcharge → Publisher secure communication

Official reference: [Appcharge to Publisher Secure Communication](https://docs.appcharge.com/api-reference/appcharge-publisher-secure-communication)

Every Appcharge **callback** POST includes:

| Header | Purpose |
|--------|---------|
| `Content-Type` | `application/json` |
| `x-publisher-token` | Publisher token from **Settings → Integration** in the Publisher Dashboard |
| `signature` | Signed raw request body |

## Signature format

Header value: `t=<unix_ms>,v1=<hex_hmac>`

- `t`: UTC time as Unix timestamp in **milliseconds**
- `v1`: HMAC-SHA256 of `"{t}.{raw_body}"` using the **main key** from the Publisher Dashboard (Integration tab), digest as **lowercase hex**

```text
message = "{t}.{raw_http_body}"   # raw body bytes/string exactly as received
v1 = hex_hmac_sha256(main_key, message)
```

Reject requests when:

- `signature` is missing or malformed
- `t` is older than ~5 minutes (replay protection)
- `v1` does not match (constant-time compare)
- `x-publisher-token` does not match configured publisher token

## Configuration (publisher server)

| Env var | Description |
|---------|-------------|
| `APPCHARGE_PUBLISHER_TOKEN` | Expected `x-publisher-token` |
| `APPCHARGE_MAIN_KEY` | HMAC secret (main key) |

Never log the main key or full request bodies in production.

## Implementation notes

1. Read the **raw body** before JSON parsing for signature verification.
2. Mount each callback on a dedicated route (e.g. `/callbacks/grant-award`).
3. Return documented status codes and JSON shapes only.
4. Use the project's existing HTTP framework (stdlib `net/http`, Gin, Echo, FastAPI, Flask, etc.).
