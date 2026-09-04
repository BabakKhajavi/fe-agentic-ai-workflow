# Impact Assessment: <Feature name>

## Document control

| Field | Value |
| --- | --- |
| Feature slug | `<feature-slug>` |
| Status | `READY_FOR_APPROVAL`, `NEEDS_CLARIFICATION`, or `BLOCKED` |
| Repository inspected | `smooth-web-*` (name the specific repositories) |
| Revision or state | `<commit, branch, or working-tree description>` |
| Human plan approval | `PENDING` |

## Repository baseline

| Concern | Observed implementation | Version or evidence | Alignment |
| --- | --- | --- | --- |
| React and TypeScript | <observed> | <package/file> | <Aligned/Gap> |
| Webpack / Module Federation | <observed> | <package/file> | <Aligned/Gap> |
| Routing and lazy loading | <observed> | <file> | <Aligned/Gap> |
| MUI theme and styling | <observed> | <file> | <Aligned/Gap> |
| Formik 2 and Yup | <observed> | <package/file> | <Aligned/Gap/Not used> |
| Redux Toolkit and RTK Query | <observed> | <package/file> | <Aligned/Gap/Not used> |
| Tests and quality scripts | <observed> | <package/file> | <Aligned/Gap> |

## Change summary

<Short explanation of how the feature fits the current application.>

## Affected areas

| ID | Action | Existing/proposed path | Change | Traceability |
| --- | --- | --- | --- | --- |
| `IMP-001` | `Create/Modify/Reuse/No change` | `<path>` | <impact> | `REQ-001`, `AC-001`, `CON-001` |

## State and data impact

### Form state

<Formik/Yup impact or `Not needed`.>

### Global client state

<Redux Toolkit slice impact or `Not needed`; explain why.>

### Server state and API

<RTK Query endpoints, cache tags, invalidation, mock/live mode, or `Not needed`.>

## Quality impact

- **Accessibility:** <impact and checks>
- **Security/privacy:** <impact and checks>
- **Performance/bundle:** <impact, including lazy route chunk>
- **Testing:** <unit, component, integration, or manual demo impact>

## Dependencies

| Dependency | Existing/new | Purpose | Required by | Risk |
| --- | --- | --- | --- | --- |
| `<package or system>` | `<Existing/New>` | <purpose> | `<ID>` | <risk> |

Use `None` when no dependency change is needed.

## POC shortcuts and production gaps

| ID | Shortcut or gap | POC rationale | Demo impact | Production follow-up |
| --- | --- | --- | --- | --- |
| `RISK-001` | <item> | <reason> | <impact> | <follow-up> |

## Open decisions

| ID | Decision | Affected records | Impact | Status |
| --- | --- | --- | --- | --- |
| `ADEC-001` | <decision> | <IDs> | <impact> | `OPEN` |

## Human plan approval

- **Decision:** `PENDING`, `APPROVED`, or `CHANGES_REQUESTED`
- **Reviewer:** —
- **Date:** —
- **Comments:** —
