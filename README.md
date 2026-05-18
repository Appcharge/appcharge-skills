# Appcharge Skills

[![skills.sh](https://skills.sh/b/Appcharge/appcharge-skills)](https://skills.sh/Appcharge/appcharge-skills)

Official [Agent Skills](https://agentskills.io/) library for Appcharge clients and partners. Skills teach AI coding agents how to work with the Appcharge platform — APIs, dashboards, integrations, and operational workflows.

**Repository:** [github.com/Appcharge/appcharge-skills](https://github.com/Appcharge/appcharge-skills) (public)

Works with **Cursor**, **Claude Code**, **Codex**, and other agents that support the Agent Skills standard.

## Quick start

Install all skills into your project (recommended for app repos):

```bash
npx skills add Appcharge/appcharge-skills --agent cursor claude-code -y
```

Install globally (available in every project):

```bash
npx skills add Appcharge/appcharge-skills -g --agent cursor claude-code -y
```

List available skills without installing:

```bash
npx skills add Appcharge/appcharge-skills --list
```

Install a single skill:

```bash
npx skills add Appcharge/appcharge-skills --skill <skill-name> -y
```

After installing, **start a new agent session** so skills are picked up.

## Available skills

### Publisher callback endpoints (Go / Python)

These skills guide an agent to add **Appcharge → publisher** HTTP callbacks in your backend: verify `x-publisher-token` and `signature`, implement the request/response contract, and wire tests. Shared signing rules: [docs/callbacks/secure-communication.md](docs/callbacks/secure-communication.md).

| Skill | Endpoint | Docs |
|-------|----------|------|
| `grant-award-callback` | Fulfill purchases and free offers; return `publisherPurchaseId` | [Grant Award Callback](https://docs.appcharge.com/api-reference/checkout/awards/grant-award-callback) |
| `personalize-webstore-callback` | Sync player store UI, offers, balances, segments | [Personalize Web Store Callback](https://docs.appcharge.com/api-reference/webstore/personalization/personalize-webstore-callback) |
| `authenticate-player-callback` | Validate web store login (SSO, password, OTP) | [Authenticate Player Callback](https://docs.appcharge.com/api-reference/webstore/player-authentication/authenticate-player-callback) |
| `initiate-game-auth-callback` | Start **Game Redirect Login** (`deepLink` + `accessToken`) | [Initiate Game Auth Callback](https://docs.appcharge.com/api-reference/webstore/player-authentication/initiate-game-auth-callback) |

Install one callback skill:

```bash
npx skills add Appcharge/appcharge-skills --skill grant-award-callback -y
```

Typical web store flow:

```text
initiate-game-auth-callback  →  (game)  →  authenticate-player-callback
        ↓ (optional)                              ↓
personalize-webstore-callback  ←──────── login / purchase / sync
        ↓
grant-award-callback  ←  checkout complete
```

## Update skills

Refresh installed skills to the latest version from this repository:

```bash
npx skills update
```

Update only global or only project skills:

```bash
npx skills update -g    # global
npx skills update -p    # current project
```

Update one skill by name:

```bash
npx skills update <skill-name>
```

## Other CLI commands

| Command | Description |
|---------|-------------|
| `npx skills list` | Show skills installed in the current project |
| `npx skills ls -g` | Show globally installed skills |
| `npx skills remove <name>` | Remove a skill |
| `npx skills find <query>` | Search [skills.sh](https://skills.sh/) |

Discover more community skills: [skills.sh](https://skills.sh/)

## Install by agent

### Cursor

**Option A — skills CLI (recommended)**

```bash
npx skills add Appcharge/appcharge-skills --agent cursor -y
```

Skills are linked under `.cursor/skills/` (or `.agents/skills/`). View them in **Cursor Settings → Rules → Agent Decides**.

**Option B — remote rule from GitHub**

1. **Cursor Settings → Rules**
2. **Add Rule → Remote Rule (GitHub)**
3. Repository: `https://github.com/Appcharge/appcharge-skills`

### Claude Code

**Option A — skills CLI**

```bash
npx skills add Appcharge/appcharge-skills --agent claude-code -y
```

**Option B — Claude plugin (from this repo)**

```text
/plugin marketplace add https://github.com/Appcharge/appcharge-skills
/plugin marketplace update
/plugin install appcharge-skills@appcharge-skills
```

Plugin skills are namespaced: `/appcharge-skills:<skill-name>`.

## Repository layout

```text
appcharge-skills/
├── docs/callbacks/              # Shared signature verification notes
└── skills/
    └── <skill-name>/
        ├── SKILL.md             # Required
        ├── scripts/             # Optional
        ├── references/          # Optional
        └── assets/              # Optional
```

Each skill is a folder with a `SKILL.md` file ([Agent Skills](https://agentskills.io/) format). The `npx skills` CLI discovers skills under `skills/` recursively.

## Support

- Issues and requests: [github.com/Appcharge/appcharge-skills/issues](https://github.com/Appcharge/appcharge-skills/issues)
- Appcharge support: [support@appcharge.com](mailto:support@appcharge.com)

## License

[MIT](LICENSE) — this repository is public on GitHub; skills may be installed and used per the license terms.
