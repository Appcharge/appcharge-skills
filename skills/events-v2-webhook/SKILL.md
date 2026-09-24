---
name: events-v2-webhook
description: >-
  Implement Appcharge Events V2 webhook — receive real-time player activity
  events (logins, orders, web store, portal) with signature verification and
  eventId deduplication.
metadata:
  author: Appcharge
  version: 0.1.0
  tags: [appcharge, webhook, events, analytics]
---

# Events V2 Webhook

Implement the Events V2 webhook endpoint on the publisher backend. Appcharge sends HTTPS POST requests with structured JSON payloads for player activities (logins, orders, web store interactions, portal events). Official spec: https://docs.appcharge.com/api-reference/events/v2/introduction.md

## When to Use

- New integration or missing Events V2 handler in the publisher backend
- User mentions Events V2, Events Center, event webhooks, `eventId`, or real-time player activity tracking

## When NOT to Use

- Grant award, authenticate player, personalize webstore, or initiate game auth — use those **callback** skills instead (request/response callbacks, not fire-and-forget events)
- Events V1 (legacy) — prefer Events V2

## Workflow

Complete **Phase 0** and **Phase 1** before writing any implementation code.

### Phase 0 — Confirm with user (required)

Ask the user explicitly. If research already suggests an answer, propose it and ask to confirm or override. **Do not implement until answered.**

**Routing and secrets**

1. Which **route path** should host this webhook? (suggestion: `/events`)
2. Which **file** registers HTTP routes for this service?
3. Which **env var or config key** stores the Appcharge **main key** (webhook signature signing secret)?
4. Which **env var or config key** stores the **publisher token** (`x-publisher-token`)?
5. Which **env var or config key** stores the **project ID** (`x-project-id`)? Should mismatches be rejected?

**Domain mapping (this webhook)**

6. Which **`eventName` values** should the handler process? (e.g. `order.completed`, `webstore.login.completed`) — or route all enabled events through one handler?
7. Should event processing be **synchronous** (inline in the request) or **async** (ack fast, process in background queue/worker)?
8. Where should **`eventId` deduplication** be stored (table, cache, idempotency store)? What TTL if applicable?
9. Which downstream **services or workflows** should each event type trigger?

### Phase 1 — Research (required)

#### 1.1 Project structure and conventions

- Detect language, framework, routes/controllers, analytics/BI modules, job queues, config.

#### 1.2 Existing Appcharge integration

Search for other Appcharge webhooks (callbacks or Events V2), signature verification, shared middleware/utils. **Reuse** before adding new code.

#### 1.3 Test conventions

- Find test framework, handler test patterns, existing webhook/signature tests.

#### 1.4 Fetch official docs (required)

Run `curl` from skill-local references (shipped with `npx skills add`):

- [references/api-contract.md](references/api-contract.md)
- [references/secure-communication.md](references/secure-communication.md)

For each `eventName` the user confirmed in Phase 0 Q6, fetch its payload page from the index:

```bash
curl -fsSL 'https://docs.appcharge.com/llms.txt' | grep 'api-reference/events/v2/'
```

Implement from the **fetched markdown only**.

### Phase 2 — Implementation

Use Phase 0 answers and Phase 1 findings throughout.

#### Signature verification

Follow [references/secure-communication.md](references/secure-communication.md):

1. Register route as `POST` at the user-confirmed path in the user-confirmed router file.
2. Read **raw body** before JSON parsing.
3. Verify `signature` (HMAC-SHA256, `t=<ms>.<payload>`, ~5 min replay window) using the user-confirmed main-key env var.
4. Validate `x-publisher-token` against the user-confirmed publisher-token env var.
5. Validate `x-project-id` if the user confirmed hardening in Phase 0 Q5.

#### Payload handling

After verification, parse the event envelope and route by `eventName`:

| Envelope field | Handler action |
|----------------|----------------|
| `eventId` | Dedup key per Phase 0 Q8 — skip already-processed events |
| `eventName` | Route to handler per Phase 0 Q6/Q9 |
| `timestamp` | Event time (ms epoch); use for ordering/logging |
| `sessionId` | Group related events in the same player session |
| `requestId` | Correlate with Appcharge logs for debugging |
| `result`, `reason` | Success/failure context when present |
| `attributes` | Custom labels (persona, A/B tests) |
| Domain objects | `customer`, `offer`, `order`, `device`, `geolocation`, etc. — fields vary by event; tolerate missing optional fields |

**Response** — Return **`2xx` promptly** to acknowledge receipt. No response body schema is required. Appcharge retries on non-2xx:

| Attempt | Delay from previous |
|---------|---------------------|
| Initial | Immediate |
| 1st retry | 15 seconds |
| 2nd retry | 15 seconds |
| 3rd retry | 15 minutes |

If processing is slow, use Phase 0 Q7 async pattern: ack with `2xx` immediately, enqueue for background processing.

**Critical:** Do not return non-2xx for successfully received and verified events that fail business logic — log and handle internally. Non-2xx triggers unnecessary retries.

#### Tests

- Valid signature + known `eventName` → `2xx`
- Invalid or missing signature → `401`
- Expired timestamp → reject (replay protection)
- Duplicate `eventId` → deduped (no double side-effects)
- Event with missing optional fields → handled gracefully
- Async mode: `2xx` returned before background work completes

#### Dashboard

1. **Settings → Integration → Events V2 Webhook** — register full URL (e.g. `https://{server}/events`)
2. **Events Center** — enable the event types from Phase 0 Q6
3. Align env vars with Dashboard → Integration and Settings → Company (project ID)

## Handler sketch

```text
verify(headers, rawBody) → parse(event) → dedup(eventId) → route(eventName) → 2xx
```

## Related skills

- `grant-award-callback` — fulfillment callback (complements `order.completed` event)
- `authenticate-player-callback` — login callback (complements `webstore.login.*` events)
- `personalize-webstore-callback` — store sync (complements `webstore.*` / `offers.*` events)
