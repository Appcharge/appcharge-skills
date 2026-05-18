# Appcharge → Publisher secure communication

Do **not** duplicate signing rules in this repo. Fetch the official markdown before implementing callback verification.

## Official docs

| Topic | Markdown URL |
|-------|----------------|
| Webhook `signature` + `x-publisher-token` (normative for callbacks) | https://docs.appcharge.com/api-reference/appcharge-publisher-secure-communication.md |
| Platform secure communication (overview) | https://docs.appcharge.com/merchant-of-record/security/appcharge-to-publisher-secure-communication.md |
| Documentation index | https://docs.appcharge.com/llms.txt |

## Fetch

```bash
curl -fsSL 'https://docs.appcharge.com/api-reference/appcharge-publisher-secure-communication.md'
```

```bash
curl -fsSL 'https://docs.appcharge.com/merchant-of-record/security/appcharge-to-publisher-secure-communication.md'
```

Implement verification exactly as described in the fetched **api-reference** page (raw body, `signature` header, replay window, main key from Publisher Dashboard → Integration).
