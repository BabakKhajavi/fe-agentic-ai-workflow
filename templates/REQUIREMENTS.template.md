# Requirements: <Feature name>

## Document control

| Field | Value |
| --- | --- |
| Feature slug | `<feature-slug>` |
| Status | `READY_FOR_SPEC`, `NEEDS_CLARIFICATION`, or `BLOCKED` |
| Intake date | `<ISO-8601 date and time>` |
| Prepared by | `Intake Agent` |
| Human intake approval | `PENDING` |
| Approved by | — |
| Approval date | — |

## Sources

| Source ID | Source type | Title or summary | Reference (URL, page ID, or issue key) | Version | Last updated | Sections or notes used |
| --- | --- | --- | --- | --- | --- | --- |
| `SRC-001` | `Confluence`, `Jira`, or `Developer-provided` | `<title>` | `<reference>` | `<version or unknown>` | `<timestamp or unknown>` | `<sections>` |

## Business objective

<State the problem, desired outcome, and business value supported by the source.>

## Users and stakeholders

| Actor | Need or responsibility | Source |
| --- | --- | --- |
| `<actor>` | `<need or responsibility>` | `SRC-001: <section>` |

## Scope

### In scope

- <Explicitly supported scope item.>

### Out of scope

- <Explicit exclusion, or state that none is specified.>

## User journeys

### Journey 1 — <Name>

1. <Actor and starting condition.>
2. <User action or system event.>
3. <Observable outcome.>

Source: `SRC-001: <section>`

## Functional requirements

### REQ-001 — <Requirement title>

- **Statement:** The system must <single observable obligation>.
- **Actor:** <actor>
- **Trigger:** <event or precondition>
- **Expected outcome:** <observable result>
- **Rationale:** <why it matters, if stated>
- **Priority:** <source priority or `Not specified`>
- **Source:** `SRC-001: <section>`
- **Original source ID:** <identifier or `None`>
- **Acceptance evidence:** <observable evidence>

## Business rules

### BR-001 — <Rule title>

- **Rule:** <single business rule>
- **Applies to:** `REQ-001`
- **Source:** `SRC-001: <section>`
- **Verification evidence:** <observable evidence>

## Feature-specific non-functional requirements

Do not duplicate the global `TECH-*` requirements here.

### NFR-001 — <Requirement title>

- **Statement:** <measurable quality or operational constraint>
- **Measure:** <threshold or observable evidence>
- **Source:** `SRC-001: <section>`

## Dependencies and constraints

| ID | Type | Description | Source | Affected requirements |
| --- | --- | --- | --- | --- |
| `DEP-001` | `<dependency or constraint>` | `<description>` | `SRC-001: <section>` | `REQ-001` |

## Assumptions

| ID | Assumption | Reason it is not confirmed | Affected requirements | Resolution needed |
| --- | --- | --- | --- | --- |
| `ASM-001` | `<assumption>` | `<reason>` | `REQ-001` | `Yes/No` |

## Ambiguities, conflicts, and missing information

| ID | Classification | Finding | Sources | Affected requirements | Impact |
| --- | --- | --- | --- | --- | --- |
| `GAP-001` | `<Ambiguity, Conflict, or Missing>` | `<finding>` | `SRC-001: <section>` | `REQ-001` | `<impact>` |

## Clarification questions

### Q-001 — <Question>

- **Impact:** `Blocking` or `Non-blocking`
- **Why it matters:** <scope, behavior, contract, architecture, or acceptance impact>
- **Affected records:** `REQ-001`, `GAP-001`
- **Owner:** <person or role if known>
- **Answer:** `PENDING`

## Applicable global technical requirements

| Technical requirement | Relevance to this feature |
| --- | --- |
| `TECH-<ID>` | <why it applies> |

## Traceability summary

| Local ID | Source | Acceptance or verification evidence | Open questions |
| --- | --- | --- | --- |
| `REQ-001` | `SRC-001: <section>` | <evidence> | `Q-001` or `None` |

## Human intake approval

- **Decision:** `PENDING`, `APPROVED`, or `CHANGES_REQUESTED`
- **Reviewer:** —
- **Date:** —
- **Comments:** —

Approval authorizes the Spec Agent to begin. It does not authorize Jira creation
or application implementation.
