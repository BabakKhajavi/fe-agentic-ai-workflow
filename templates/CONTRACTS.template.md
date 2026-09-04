# Contracts: <Feature name>

## Document control

| Field | Value |
| --- | --- |
| Feature slug | `<feature-slug>` |
| Status | `READY_FOR_PLANNING`, `NEEDS_CLARIFICATION`, or `BLOCKED` |
| Integration mode | `LIVE`, `MOCK`, `MIXED`, or `NONE` |
| Human spec approval | `PENDING` |

Include only sections relevant to this feature. Write `Not needed` when a
category is intentionally unused.

## Route contract

### CON-001 — <Route name>

- **Path:** `<path>`
- **Access:** <public or permission requirement>
- **Loading:** Lazy-loaded route module with accessible fallback
- **Inputs:** <path, query, or navigation state>
- **Outputs:** <rendered outcome or redirect>
- **Covers:** `REQ-001`, `AC-001`

## Form and validation contract

### CON-010 — <Form name>

- **Form state:** Formik 2
- **Validation:** Yup
- **Submission owner:** `<custom hook name or responsibility>`

| Field | Type | Required | Validation | MUI control |
| --- | --- | --- | --- | --- |
| `<field>` | `<type>` | `Yes/No` | `<rule>` | `<control>` |

## Data shapes

```ts
interface Example {
  id: string
}
```

## API and RTK Query contract

### CON-020 — <Operation>

- **Mode:** `LIVE` or `MOCK`
- **Method and path:** `<METHOD /path>` or `Local mock`
- **Request:** `<type or None>`
- **Response:** `<type>`
- **RTK Query endpoint:** `<endpoint name>`
- **Provides tags:** `<tag/id or None>`
- **Invalidates tags:** `<tag/id or None>`
- **Error behavior:** <user-visible and retry behavior>
- **Covers:** `REQ-001`, `AC-001`

## Global client-state contract

Use Redux Toolkit only for global client state not owned by Formik, the URL, or
RTK Query.

| Slice | State | Actions | Persistence |
| --- | --- | --- | --- |
| `Not needed` | — | — | — |

## Important UI and hook boundaries

| Contract ID | Boundary | Inputs | Outputs/responsibility |
| --- | --- | --- | --- |
| `CON-030` | `<custom hook or externally meaningful component>` | `<inputs>` | `<outputs>` |

## Contract traceability

| Contract | Requirements | Acceptance criteria |
| --- | --- | --- |
| `CON-001` | `REQ-001` | `AC-001` |

## Open contract decisions

| ID | Decision | Impact | Status |
| --- | --- | --- | --- |
| `CDEC-001` | <decision> | <impact> | `OPEN` |

## Human spec approval

- **Decision:** `PENDING`, `APPROVED`, or `CHANGES_REQUESTED`
- **Reviewer:** —
- **Date:** —
- **Comments:** —
