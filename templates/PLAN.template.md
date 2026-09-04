# Implementation Plan: <Feature name>

## Document control

| Field | Value |
| --- | --- |
| Feature slug | `<feature-slug>` |
| Status | `READY_FOR_APPROVAL`, `NEEDS_CLARIFICATION`, or `BLOCKED` |
| Based on | `REQUIREMENTS.md`, `SPEC.md`, `CONTRACTS.md`, `IMPACT.md` |
| Human plan approval | `PENDING` |

## Implementation strategy

<Smallest coherent technical approach for the POC.>

## Ordered tasks

### TASK-001 — <Outcome-oriented title>

- **Outcome:** <completed technical/user-visible result>
- **Depends on:** `None` or task IDs
- **Expected files:**
  - `Create: <path>`
  - `Modify: <path>`
  - `Reuse: <path>`
- **Implementation notes:** <important constraints, not line-by-line coding>
- **Requirements:** `REQ-001`, `BR-001`, `NFR-001`, `TECH-<ID>`
- **Acceptance criteria:** `AC-001`
- **Contracts:** `CON-001`
- **Verification:** <test/check and expected evidence>
- **Suggested Jira type:** `Story` or `Task`

## Final integration verification

Run the repository's actual scripts for:

1. Type checking.
2. Linting.
3. Automated tests.
4. Repository's configured production build (Webpack for `smooth-web-*/`).
5. POC user-flow demonstration.

Record any manual acceptance criterion and why automation is not appropriate for
this POC.

## Traceability matrix

| Requirement | Acceptance criteria | Contracts | Impact | Tasks | Verification |
| --- | --- | --- | --- | --- | --- |
| `REQ-001` | `AC-001` | `CON-001` | `IMP-001` | `TASK-001` | <evidence planned> |

## Recommended Jira breakdown

- **Epic:** <feature outcome>
- **Child work:** One item per `TASK-*`, combining only tasks that cannot be
  independently delivered or reviewed.

Jira creation remains prohibited until this plan and `IMPACT.md` are approved.

## Human plan approval

- **Decision:** `PENDING`, `APPROVED`, or `CHANGES_REQUESTED`
- **Reviewer:** —
- **Date:** —
- **Comments:** —

Approval authorizes Jira creation and implementation according to this plan. It
does not authorize unplanned scope or production deployment.
