# Specification: <Feature name>

## Document control

| Field | Value |
| --- | --- |
| Feature slug | `<feature-slug>` |
| Status | `READY_FOR_PLANNING`, `NEEDS_CLARIFICATION`, or `BLOCKED` |
| Requirements baseline | `<REQUIREMENTS.md approval date or revision>` |
| Human spec approval | `PENDING` |

## Objective

<Concise desired user and business outcome.>

## Scope

### In scope

- <Approved scope item and requirement IDs.>

### Out of scope

- <Explicit exclusion.>

## User flow

1. <Starting state and actor.>
2. <User action.>
3. <System response.>
4. <Completion state.>

## Behavioral states

| State | Expected behavior | Related requirements |
| --- | --- | --- |
| Initial | <behavior> | `REQ-001` |
| Loading | <behavior or Not applicable> | `TECH-060` |
| Empty | <behavior or Not applicable> | `TECH-060` |
| Success | <behavior> | `REQ-001` |
| Validation error | <behavior or Not applicable> | `TECH-103` |
| Service error | <behavior or Not applicable> | `TECH-061` |

## Acceptance criteria

### AC-001 — <Observable outcome>

- **Given:** <starting condition>
- **When:** <action or event>
- **Then:** <observable result>
- **Covers:** `REQ-001`, `BR-001`
- **Verification:** `Automated` or `Manual POC demonstration`

## POC implementation assumptions

| ID | Assumption or shortcut | Reason | Demo effect | Production follow-up |
| --- | --- | --- | --- | --- |
| `POC-001` | <for example, mocked API> | <reason> | <effect> | <needed follow-up> |

## Applicable technical requirements

- `TECH-<ID>` — <feature relevance>

## Open decisions

| ID | Decision needed | Impact | Owner | Status |
| --- | --- | --- | --- | --- |
| `DEC-001` | <question> | <impact> | <role> | `OPEN` |

## Traceability

| Requirement | Acceptance criteria | Contract references |
| --- | --- | --- |
| `REQ-001` | `AC-001` | `CON-001` |

## Human spec approval

- **Decision:** `PENDING`, `APPROVED`, or `CHANGES_REQUESTED`
- **Reviewer:** —
- **Date:** —
- **Comments:** —

Approval authorizes architecture impact analysis and planning only.
