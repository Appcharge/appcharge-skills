# Appcharge → Publisher Events V2 secure communication

Fetch the official markdown before implementing; the algorithm below summarizes the normative contract.

## Official docs

| Topic | Markdown URL |
|-------|----------------|
| Events V2 introduction (setup, signature, retry) | https://docs.appcharge.com/api-reference/events/v2/introduction.md |
| Callback secure communication (same HMAC algorithm) | https://docs.appcharge.com/api-reference/appcharge-publisher-secure-communication.md |
| Events Center (Dashboard registration) | https://docs.appcharge.com/guides/events/about-the-events-center.md |
| Documentation index (all event payloads) | https://docs.appcharge.com/llms.txt |

## Fetch

```bash
curl -fsSL 'https://docs.appcharge.com/api-reference/events/v2/introduction.md'
```

## Signature verification (implement exactly)

Appcharge sends three headers on every Events V2 webhook:

| Header | Purpose |
|--------|---------|
| `x-publisher-token` | Publisher token from Dashboard → Settings → Integration |
| `x-project-id` | Project ID from Dashboard → Settings → Company → Project ID |
| `signature` | HMAC of the payload; format `t=<unix_ms>,v1=<hex>` |

**Main key** (signing secret) comes from Dashboard → Settings → Integration. Load it from the env var or config key the user confirmed in Phase 0.

### Verification steps

1. **Read raw body** — Buffer the request body as a string/bytes **before** JSON parsing. Prefer signing the raw bytes (same as callback endpoints). The official Events V2 example uses `JSON.stringify(req.body)` after parsing; if verification fails with raw body, confirm against the fetched introduction doc — do not re-serialize with different key ordering or whitespace.
2. **Parse `signature` header** — Split on `,`; read `t` (Unix timestamp in **milliseconds**, UTC) and `v1` (hex digest).
3. **Replay window** — Reject if `t` is older than ~5 minutes from now.
4. **Compute expected signature**:

```text
message = t + "." + rawBody
expected = HMAC-SHA256(mainKey, message) as lowercase hex
```

5. **Compare** — Constant-time compare `expected` with `v1`. On mismatch → reject (`401`).
6. **Validate `x-publisher-token`** — Must equal the configured publisher token. On mismatch → reject.
7. **Validate `x-project-id`** — Must equal the configured project ID (optional hardening; confirm with user in Phase 0).
8. **Parse JSON** — Only after steps 1–7 pass.

### Middleware pattern

Extract verification into shared middleware or a util reused by all Appcharge webhooks:

```text
readRawBody → verifySignature(headers, rawBody, mainKey) → verifyPublisherToken(header, config) → verifyProjectId(header, config) → parseJSON(rawBody) → handler
```

If the project already has Appcharge verification code (e.g. from callback skills), **extend it** — do not add a second implementation.

## Publisher token and project ID

Separate from the main key. Confirm env var or config key names with the user in Phase 0.
