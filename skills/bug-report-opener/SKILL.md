---
name: bug-report-opener
description: >-
  Generates a professional bug report and creates it directly in Jira (project ACDEV)
  as a Bug issue. Use when the user says "open a bug", "bug opener", "write a bug report",
  "create a bug ticket", or describes a bug and wants it filed. Guides the user through
  a minimal two-step flow: ask for media, show a pre-flight summary, then create on confirmation.
---

# Bug Report Opener

## Trigger

Activate when the user says "open a bug", "bug opener", "write a bug report", "create a bug",
or provides a bug description and wants it filed in Jira.

---

## Phase 1 — Parse and infer fields

From the user's description, derive:

### Title format
```
[AREA | SUBJECT] - Short precise description
```
- **AREA** — one of the valid areas below; include version if provided (e.g. `Checkout v3`)
- **SUBJECT** — specific component/feature (e.g. `Address Field`, `APM`, `Login`)
- **Description** — sentence case, no trailing period, under ~90 chars total

**Valid areas:** Dashboard, Checkout, Store, Portal, Admin Dashboard
If the area is unclear, ask: *"Which area does this belong to? e.g. Dashboard, Checkout, Store, Portal, Admin Dashboard."*

### Team mapping

Infer `Appcharge Team` from the area/subject:

| Signals in title/description | Team |
|---|---|
| Store, Dashboard, Portal, Offers, Login, Admin Dashboard, UI | MonGroup |
| Checkout UI, payment UX, financial reports | PayEx |
| Payments backend, checkout backend, payment processing | PayCore |
| Mobile SDK, app | PayMob |
| Automation, test, CI | Automation |
| DevOps, infra, servers, deployment, pipeline | DevOps |
| Security, vulnerability, threat | Security |
| IT | IT |
| Platform, AI, company process | Platform |

If no match → default to **QA**. Note to the user after creation that the team was defaulted and should be reassigned.

### Default field values

| Field | Default |
|---|---|
| Severity | Medium |
| Found By | QA |
| Appcharge environment | Staging |
| Component | SUBJECT from the title (e.g. `Address Field`) |
| Progression / Regression | Regression |
| Priority | Medium |

Override any default if the user explicitly provides a different value.

---

## Phase 2 — Ask for media

After parsing the description, ask exactly this (single message):

> "Any media to attach? Drop an image or video here, or say **no** to skip."

Wait for the response before proceeding.
- If the user attaches a file: note the filename; include a reminder after creation (see Phase 5).
- If the user says no: continue.

---

## Phase 3 — Pre-flight summary

Show the summary as a markdown table before creating anything.
Do not create the ticket yet.

```
**Ready to create — confirm or correct anything:**

| Field | Value |
|---|---|
| **Title** | [AREA \| SUBJECT] - description |
| **Team** | <inferred team> |
| **Severity** | <value> |
| **Environment** | <value> |
| **Found By** | <value> |
| **Progression/Regression** | <value> |
| **Reporter** | <name from atlassianUserInfo> |
| **Media** | <filename or None> |

*Type "create" to confirm, or tell me what to change.*
```

For the Reporter field: call `atlassianUserInfo` (user-atlassian-mcp) to get the current
logged-in Atlassian user's display name and accountId. Use the display name in the table
and the accountId when creating the issue.

Wait for user confirmation before proceeding to Phase 4.

---

## Phase 4 — Create the Jira issue

Execute these steps in order:

### Step 1 — Get cloudId
Call `getAccessibleAtlassianResources` (user-atlassian-mcp).
Use the cloudId of the Appcharge Jira site.

### Step 2 — Get field metadata
Call `getJiraProjectIssueTypesMetadata` (user-atlassian-mcp) with:
- `projectIdOrKey`: `ACDEV`

Find the issue type ID for **Bug**.

Then call `getJiraIssueTypeMetaWithFields` with the Bug issue type ID to discover
the custom field IDs for: Severity, Found By, Appcharge Team, Appcharge environment,
Components, Progression / Regression. Map field names to their `customfield_XXXXX` keys.

### Step 3 — Build the ADF description

Use `contentFormat: "adf"` in `createJiraIssue`.

The description is an ADF document. Each section label (`Image/Video of the issue:`,
`Summary:`, `Steps to Reproduce:`, `Expected Result:`, `Actual Result:`, `Env:`) must
have **both bold and underline** marks. Content follows on the next node or paragraph.

ADF mark structure for labels:
```json
{
  "type": "text",
  "text": "Summary:",
  "marks": [{ "type": "strong" }, { "type": "underline" }]
}
```

Full ADF template structure:
```json
{
  "version": 1,
  "type": "doc",
  "content": [
    {
      "type": "paragraph",
      "content": [
        { "type": "text", "text": "Image/Video of the issue:", "marks": [{ "type": "strong" }, { "type": "underline" }] }
      ]
    },
    { "type": "paragraph", "content": [{ "type": "text", "text": "<blank or media note>" }] },
    {
      "type": "paragraph",
      "content": [
        { "type": "text", "text": "Summary:", "marks": [{ "type": "strong" }, { "type": "underline" }] }
      ]
    },
    { "type": "paragraph", "content": [{ "type": "text", "text": "<summary text>" }] },
    {
      "type": "paragraph",
      "content": [
        { "type": "text", "text": "Steps to Reproduce:", "marks": [{ "type": "strong" }, { "type": "underline" }] }
      ]
    },
    {
      "type": "orderedList",
      "content": [
        { "type": "listItem", "content": [{ "type": "paragraph", "content": [{ "type": "text", "text": "<step 1>" }] }] }
      ]
    },
    {
      "type": "paragraph",
      "content": [
        { "type": "text", "text": "Expected Result:", "marks": [{ "type": "strong" }, { "type": "underline" }] }
      ]
    },
    { "type": "paragraph", "content": [{ "type": "text", "text": "<expected result>" }] },
    {
      "type": "paragraph",
      "content": [
        { "type": "text", "text": "Actual Result:", "marks": [{ "type": "strong" }, { "type": "underline" }] }
      ]
    },
    { "type": "paragraph", "content": [{ "type": "text", "text": "<actual result>" }] },
    {
      "type": "paragraph",
      "content": [
        { "type": "text", "text": "Env:", "marks": [{ "type": "strong" }, { "type": "underline" }] }
      ]
    },
    { "type": "paragraph", "content": [{ "type": "text", "text": "<env or blank>" }] }
  ]
}
```

### Step 4 — Create the issue

Call `createJiraIssue` (user-atlassian-mcp):
- `projectKey`: `ACDEV`
- `issueTypeName`: `Bug`
- `summary`: the full title string `[AREA | SUBJECT] - description`
- `description`: the ADF JSON from Step 3
- `contentFormat`: `adf`
- `additional_fields`:
```json
{
  "priority": { "name": "<priority>" },
  "reporter": { "id": "<accountId from atlassianUserInfo>" },
  "<severity_field_id>": { "value": "<severity>" },
  "<found_by_field_id>": { "value": "<found by>" },
  "<appcharge_team_field_id>": { "value": "<team>" },
  "<appcharge_env_field_id>": { "value": "<environment>" },
  "<components_field_id>": [{ "name": "<component>" }],
  "<progression_regression_field_id>": { "value": "<value>" }
}
```

Use the actual `customfield_XXXXX` keys discovered in Step 2.

---

## Phase 5 — Return the result

After successful creation, respond with:

1. The Jira ticket URL: `https://appcharge.atlassian.net/browse/<ISSUE-KEY>`
2. If the team was defaulted to QA: *"Note: team was set to QA by default — reassign to the correct team in Jira."*
3. If media was provided: *"Note: please attach [filename] directly in the Jira ticket — file upload via this flow is not supported yet."*
4. *"You can now add a parent ticket, additional context, or any missing fields directly in Jira."*

---

## Content rules

- Write in clear, neutral English. No slang, no filler words.
- Steps: atomic, sequential, one action per step.
- Summary: answers *what* fails, *where*, and *what the user cannot do as a result*.
- Never invent steps, environments, or expected results not provided by the user.
- Leave `Env:` blank in the description if not provided — do not write a placeholder.
