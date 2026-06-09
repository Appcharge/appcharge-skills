# Appcharge → Publisher secure communication

Fetch the official markdown before implementing; the algorithm below summarizes the normative contract.

## Official docs

| Topic | Markdown URL |
|-------|----------------|
| Webhook `signature` + `x-publisher-token` (normative) | https://docs.appcharge.com/api-reference/appcharge-publisher-secure-communication.md |
| Platform secure communication (overview) | https://docs.appcharge.com/merchant-of-record/security/appcharge-to-publisher-secure-communication.md |
| Documentation index | https://docs.appcharge.com/llms.txt |

## Fetch

```bash
curl -fsSL 'https://docs.appcharge.com/api-reference/appcharge-publisher-secure-communication.md'
```

## Signature verification (implement exactly)

Appcharge sends two headers on every callback:

| Header | Purpose |
|--------|---------|
| `x-publisher-token` | Publisher token from Dashboard → Settings → Integration |
| `signature` | HMAC of the raw body; format `t=<unix_ms>,v1=<hex>` |

**Main key** (signing secret) comes from Dashboard → Admin → Integration. Load it from the env var or config key the user confirmed in Phase 0.

### Verification steps

1. **Read raw body** — Buffer the request body as a string/bytes **before** JSON parsing. Any re-serialization breaks verification.
2. **Parse `signature` header** — Split on `,`; read `t` (Unix timestamp in **milliseconds**, UTC) and `v1` (hex digest).
3. **Replay window** — Reject if `t` is older than ~5 minutes from now.
4. **Compute expected signature**:

```text
message = t + "." + rawBody
expected = HMAC-SHA256(mainKey, message) as lowercase hex
```

5. **Compare** — Constant-time compare `expected` with `v1`. On mismatch → reject (typically `401`/`403` per endpoint docs).
6. **Validate `x-publisher-token`** — Must equal the configured publisher token. On mismatch → reject.
7. **Parse JSON** — Only after steps 1–6 pass.

### Middleware pattern

Extract verification into shared middleware or a util reused by all Appcharge callbacks:

```text
readRawBody → verifySignature(headers, rawBody, mainKey) → verifyPublisherToken(header, config) → parseJSON(rawBody) → handler
```

If the project already has Appcharge verification code, **extend it** — do not add a second implementation.

## Publisher token

Separate from the main key. Confirm the env var or config key name with the user in Phase 0.
