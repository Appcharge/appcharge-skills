---
name: bug-report-opener
description: >
  Generates a professional Jira bug ticket from a plain-language description and
  creates it directly in project ACDEV via the Atlassian MCP. Use when the user
  says "open a bug", "bug opener", "write a bug report", or "create a bug ticket".
metadata:
  author: appcharge-qa
  version: 0.1.0
  tags: [qa, jira, bug-report, atlassian]
---

# Bug Report Opener

Creates a properly formatted Jira Bug in project ACDEV from a plain-language description.
Infers all fields automatically, shows a pre-flight summary for confirmation, then creates
the ticket via the Atlassian MCP.

## When to Use This Skill

- User says "open a bug", "bug opener", "write a bug report", or "create a bug ticket"
- User provides a bug description and wants it filed in Jira

## When NOT to Use

- Creating non-bug issue types (Tasks, Stories, Epics)
- Filing bugs in projects other than ACDEV

## Workflow

### Step 1: Parse and infer fields

From the user's description, derive:

**Title format:**
```
[AREA | SUBJECT] - Short precise description
```
- **AREA** — one of: `Dashboard`, `Checkout`, `Store`, `Portal`, `Admin Dashboard`; include version if provided (e.g. `Checkout v3`)
- **SUBJECT** — specific component/feature (e.g. `Address Field`, `Edge Browser`, `Bundle`)
- **Description** — sentence case, no trailing period, under ~90 chars total

If the area is unclear, ask: *"Which area does this belong to? e.g. Dashboard, Checkout, Store, Portal, Admin Dashboard."*

**Team mapping (`customfield_10075`):**

| Signals in title/description | Appcharge Team value |
|---|---|
| Store, Dashboard, Portal, Offers, Login, Admin Dashboard, UI | `MonGroup` |
| Checkout UI, payment UX, financial reports | `PayEx Team` |
| Payments backend, checkout backend, payment processing | `PayCore Team` |
| Mobile SDK, app | `PayMob Team` |
| Automation, test, CI | `Automation Team` |
| DevOps, infra, servers, deployment, pipeline | `Devops Team` |
| Security, vulnerability, threat | `Security Team` |
| IT | `IT Team` |
| Platform, AI, company process | `Platform Team` |

Full valid values: `Architects`, `Automation Team`, `Devops Team`, `Monetization Team`, `IT Team`, `PayCore Team`, `PayEx Team`, `PayMob Team`, `Pixels Team`, `Platform Team`, `QA Team`, `Security Team`, `Support Team`, `Tech Writers`, `FinOps`, `MonGroup`

If no match → default to `QA Team` and note it to the user after creation.

**Default field values:**

| Field | Default |
|---|---|
| Severity (`customfield_10071`) | `Normal` (valid: `Minor`, `Normal`, `Major`, `Critical`, `Blocker`) |
| Found By (`customfield_10282`) | `QA` |
| Environment (`customfield_10042`) | `Staging` |
| Progression/Regression (`customfield_10044`) | `Regression` |
| Priority | `Medium` |
| Component | Inferred from area — see known mappings below |

**Known component names by area:**

| Area | Component name |
|---|---|
| Checkout v3 | `Checkout V3` |
| Checkout (general) | `Checkout` |
| Store | `Store 2.0` |

If the component is not in the table, call `searchJiraIssuesUsingJql` with
`project = ACDEV AND issuetype = Bug AND component is not EMPTY AND summary ~ "<area>"` to discover it.

### Step 2: Show pre-flight summary

Show this table and wait for confirmation before creating anything:

```
**Ready to create — confirm or correct anything:**

| Field | Value |
|---|---|
| **Title** | [AREA | SUBJECT] - description |
| **Team** | <inferred team> |
| **Severity** | <value> |
| **Environment** | <value> |
| **Found By** | <value> |
| **Progression/Regression** | <value> |
| **Reporter** | <name from atlassianUserInfo> |

*Type "create" to confirm, or tell me what to change.*
```

Call `atlassianUserInfo` (user-atlassian-mcp) to get the reporter's display name and `account_id`.

### Step 3: Create the Jira issue

All stable IDs are hardcoded — skip all discovery calls.

**Hardcoded values:**
- `cloudId`: `38515cfd-c8c7-4d33-80dc-c473e0ae2550`
- `projectKey`: `ACDEV`
- `issueTypeName`: `Bug`
- `customfield_10381` = Description (ADF)

Call `createJiraIssue` (user-atlassian-mcp) with `contentFormat: "adf"`.

**ADF description template** — section labels use heading level 4 with underline mark,
matching the existing Jira bug format in this project:

```json
{
  "version": 1,
  "type": "doc",
  "content": [
    { "type": "heading", "attrs": { "level": 4 }, "content": [{ "type": "text", "text": "Summary:", "marks": [{ "type": "underline" }] }] },
    { "type": "paragraph", "content": [{ "type": "text", "text": "<summary text>" }] },
    { "type": "heading", "attrs": { "level": 4 }, "content": [{ "type": "text", "text": "Steps to Reproduce:", "marks": [{ "type": "underline" }] }] },
    { "type": "orderedList", "content": [{ "type": "listItem", "content": [{ "type": "paragraph", "content": [{ "type": "text", "text": "<step 1>" }] }] }] },
    { "type": "heading", "attrs": { "level": 4 }, "content": [{ "type": "text", "text": "Expected Result:", "marks": [{ "type": "underline" }] }] },
    { "type": "paragraph", "content": [{ "type": "text", "text": "<expected result>" }] },
    { "type": "heading", "attrs": { "level": 4 }, "content": [{ "type": "text", "text": "Actual Result:", "marks": [{ "type": "underline" }] }] },
    { "type": "paragraph", "content": [{ "type": "text", "text": "<actual result>" }] },
    { "type": "heading", "attrs": { "level": 4 }, "content": [{ "type": "text", "text": "Env:", "marks": [{ "type": "underline" }] }] },
    { "type": "paragraph", "content": [{ "type": "text", "text": "<env or blank>" }] }
  ]
}
```

**`additional_fields` object:**
```json
{
  "priority": { "name": "<priority>" },
  "reporter": { "id": "<accountId from atlassianUserInfo>" },
  "customfield_10071": { "value": "<severity>" },
  "customfield_10075": { "value": "<appcharge team>" },
  "customfield_10282": { "value": "<found by>" },
  "customfield_10042": { "value": "<environment>" },
  "customfield_10044": { "value": "<progression/regression>" },
  "customfield_10381": "<ADF doc>",
  "components": [{ "name": "<component name>" }]
}
```

### Step 4: Return the result

Respond with:
1. The Jira ticket URL: `https://appcharge.atlassian.net/browse/<ISSUE-KEY>`
2. If team was defaulted to QA Team: *"Note: team was set to QA Team by default — reassign to the correct team in Jira."*
3. Always: *"If you have any media (image or video), you can attach it directly in the Jira ticket. You can also add a parent ticket or any additional context there."*

## Safety Guardrails

- Never create issue types other than `Bug` via this skill
- Never invent steps, environments, or expected results not provided by the user
- Leave `Env:` blank in the ADF if not provided — do not write a placeholder
- Do not ask for media — file upload is not supported via MCP; remind the user to attach manually after creation

## References

- Jira project: `https://appcharge.atlassian.net/projects/ACDEV`
- ADF format reference: `https://developer.atlassian.com/cloud/jira/platform/apis/document/structure/`
