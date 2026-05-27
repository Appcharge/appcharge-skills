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

-   **AREA** — one of the valid areas below; include version if provided (e.g. `Checkout v3`)
-   **SUBJECT** — specific component/feature (e.g. `Address Field`, `APM`, `Login`)
-   **Description** — sentence case, no trailing period, under ~90 chars total

**Valid areas:** Dashboard, Checkout, Store, Portal, Admin Dashboard
If the area is unclear, ask: _"Which area does this belong to? e.g. Dashboard, Checkout, Store, Portal, Admin Dashboard."_

### Team mapping

Infer `Appcharge Team` from the area/subject:

| Signals in title/description                                 | Appcharge Team value |
| ------------------------------------------------------------ | -------------------- |
| Store, Dashboard, Portal, Offers, Login, Admin Dashboard, UI | `MonGroup`           |
| Checkout UI, payment UX, financial reports                   | `PayEx Team`         |
| Payments backend, checkout backend, payment processing       | `PayCore Team`       |
| Mobile SDK, app                                              | `PayMob Team`        |
| Automation, test, CI                                         | `Automation Team`    |
| DevOps, infra, servers, deployment, pipeline                 | `Devops Team`        |
| Security, vulnerability, threat                              | `Security Team`      |
| IT                                                           | `IT Team`            |
| Platform, AI, company process                                | `Platform Team`      |

Full valid values: `Architects`, `Automation Team`, `Devops Team`, `Monetization Team`, `IT Team`, `PayCore Team`, `PayEx Team`, `PayMob Team`, `Pixels Team`, `Platform Team`, `QA Team`, `Security Team`, `Support Team`, `Tech Writers`, `FinOps`, `MonGroup`

If no match → default to **`QA Team`**. Note to the user after creation that the team was defaulted and should be reassigned.

### Default field values

| Field                    | Default                                                                                                                                                                                                                             |
| ------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Severity                 | Normal (valid values: Minor, Normal, Major, Critical, Blocker)                                                                                                                                                                      |
| Found By                 | QA                                                                                                                                                                                                                                  |
| Appcharge environment    | Staging                                                                                                                                                                                                                             |
| Component                | Search recent bugs in the same area to find the matching component name (e.g. `Checkout V3`, `Checkout`, `Payment Links SDKs`). Use `searchJiraIssuesUsingJql` with `component is not EMPTY AND summary ~ "<area>"` to discover it. |
| Progression / Regression | Regression                                                                                                                                                                                                                          |
| Priority                 | Medium                                                                                                                                                                                                                              |

Override any default if the user explicitly provides a different value.

---

## Phase 2 — Pre-flight summary

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

All stable IDs are hardcoded below — do not make discovery calls.

**Hardcoded values:**

-   `cloudId`: `38515cfd-c8c7-4d33-80dc-c473e0ae2550`
-   `projectKey`: `ACDEV`
-   `issueTypeName`: `Bug`
-   `customfield_10071` = Severity (valid: `Minor`, `Normal`, `Major`, `Critical`, `Blocker`)
-   `customfield_10075` = Appcharge Team (e.g. `PayEx Team`, `MonGroup Team`, `PayCore Team`, `PayMob Team`)
-   `customfield_10282` = Found By (e.g. `QA`)
-   `customfield_10042` = Appcharge environment (e.g. `Staging`, `Production`, `Sandbox`)
-   `customfield_10044` = Progression / Regression (e.g. `Regression`, `Progression`)
-   `customfield_10381` = Description (ADF)

**Known component names by area:**

| Area               | Component name           |
| ------------------ | ------------------------ |
| Checkout v3        | `Checkout V3`            |
| Checkout (general) | `Checkout`               |
| Store              | search via JQL if unsure |
| Dashboard          | search via JQL if unsure |
| Portal             | search via JQL if unsure |

If the component is not in the table above, call `searchJiraIssuesUsingJql` with
`project = ACDEV AND issuetype = Bug AND component is not EMPTY AND summary ~ "<area>"` to find it.

### Step 1 — Get reporter

Call `atlassianUserInfo` (user-atlassian-mcp) to get the current user's `account_id`.
Use it as `reporter.id` in `additional_fields`.

### Step 2 — Build the ADF description

Use `contentFormat: "adf"` in `createJiraIssue`.

The description is an ADF document. Each section label (`Summary:`, `Steps to Reproduce:`,
`Expected Result:`, `Actual Result:`, `Env:`) must have an underline mark. Content follows
on the next paragraph.

ADF structure for labels — use `heading` level 4 with `underline` mark (matches existing Jira bugs in this project):

```json
{
    "type": "heading",
    "attrs": { "level": 4 },
    "content": [{ "type": "text", "text": "Summary:", "marks": [{ "type": "underline" }] }]
}
```

Full ADF template structure:

```json
{
    "version": 1,
    "type": "doc",
    "content": [
        {
            "type": "heading",
            "attrs": { "level": 4 },
            "content": [{ "type": "text", "text": "Summary:", "marks": [{ "type": "underline" }] }]
        },
        { "type": "paragraph", "content": [{ "type": "text", "text": "<summary text>" }] },
        {
            "type": "heading",
            "attrs": { "level": 4 },
            "content": [{ "type": "text", "text": "Steps to Reproduce:", "marks": [{ "type": "underline" }] }]
        },
        {
            "type": "orderedList",
            "content": [
                {
                    "type": "listItem",
                    "content": [{ "type": "paragraph", "content": [{ "type": "text", "text": "<step 1>" }] }]
                }
            ]
        },
        {
            "type": "heading",
            "attrs": { "level": 4 },
            "content": [{ "type": "text", "text": "Expected Result:", "marks": [{ "type": "underline" }] }]
        },
        { "type": "paragraph", "content": [{ "type": "text", "text": "<expected result>" }] },
        {
            "type": "heading",
            "attrs": { "level": 4 },
            "content": [{ "type": "text", "text": "Actual Result:", "marks": [{ "type": "underline" }] }]
        },
        { "type": "paragraph", "content": [{ "type": "text", "text": "<actual result>" }] },
        {
            "type": "heading",
            "attrs": { "level": 4 },
            "content": [{ "type": "text", "text": "Env:", "marks": [{ "type": "underline" }] }]
        },
        { "type": "paragraph", "content": [{ "type": "text", "text": "<env or blank>" }] }
    ]
}
```

### Step 3 — Create the issue

Call `createJiraIssue` (user-atlassian-mcp):

-   `projectKey`: `ACDEV`
-   `issueTypeName`: `Bug`
-   `summary`: the full title string `[AREA | SUBJECT] - description`
-   `description`: the ADF JSON from Step 3
-   `contentFormat`: `adf`
-   `additional_fields`:

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
2. If the team was defaulted to QA: _"Note: team was set to QA by default — reassign to the correct team in Jira."_
3. Always: _"If you have any media (image or video), you can attach it directly in the Jira ticket. You can also add a parent ticket or any additional context there."_

---

## Content rules

-   Write in clear, neutral English. No slang, no filler words.
-   Steps: atomic, sequential, one action per step.
-   Summary: answers _what_ fails, _where_, and _what the user cannot do as a result_.
-   Never invent steps, environments, or expected results not provided by the user.
-   Leave `Env:` blank in the description if not provided — do not write a placeholder.
