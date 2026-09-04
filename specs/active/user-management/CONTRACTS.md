# Contracts: User Management Dashboard

## Document control

| Field | Value |
| --- | --- |
| Feature slug | `user-management` |
| Status | `READY_FOR_PLANNING` |
| Integration mode | `MOCK` |
| Human spec approval | `APPROVED` |

The entire feature integrates with the public DummyJSON Users API
(`https://dummyjson.com`), which the product source itself designates as this
feature's mock backend (`REQUIREMENTS.md` `DEP-001`, `DEP-002`). DummyJSON is a
real, reachable API — it is not an invented endpoint — but it does not persist
mutations server-side, so every operation below is labeled `MOCK` and every
mutation documents a session-scoped cache update rather than assuming durable
server storage.

## Route contract

### CON-001 — User Management route

- **Path:** `/users`
- **Access:** Public within the application (no authentication or authorization; explicitly out of scope)
- **Loading:** Lazy-loaded route module (`React.lazy`) rendered through the application shell's existing accessible Suspense fallback and route-level error boundary
- **Inputs:** None (no path or query parameters)
- **Outputs:** Rendered User Management page: header, table, pagination controls, and session-only notice
- **Covers:** `REQ-001`, `REQ-002`, `REQ-018`, `REQ-019`; `AC-001`, `AC-002`, `AC-018`, `AC-019`

## Form and validation contract

### CON-010 — Add/Edit User form

- **Form state:** Formik 2
- **Validation:** Yup
- **Submission owner:** A feature custom hook (name assigned during planning) responsible for: mapping the selected Role label to its lowercase API value (`BR-014`), calling the add or update mutation, applying the session-scoped cache update, and surfacing success/error notifications. Presentation components render the form; they do not call RTK Query directly (`TECH-120`–`TECH-122`).

| Field | Type | Required | Validation | MUI control |
| --- | --- | --- | --- | --- |
| `firstName` | `string` | Yes | Trim before checking; non-empty; max 50 chars (`BR-001`) | `TextField` |
| `lastName` | `string` | Yes | Trim before checking; non-empty; max 50 chars (`BR-002`) | `TextField` |
| `email` | `string` | Yes | Non-empty; valid email format; max 100 chars (`BR-003`) | `TextField` (`type="email"`) |
| `phone` | `string` | Yes | Non-empty; 7–25 chars; digits, spaces, parentheses, hyphens, optional leading `+` only (`BR-004`) | `TextField` (`type="tel"`) |
| `role` | `RoleLabel` (`'Admin' \| 'Moderator' \| 'User'`) | Yes | Non-empty; one of the three labels (`BR-005`); mapped to a lowercase `RoleValue` on submit (`BR-014`) | `Select` |

Validation runs on blur/touch and on submit; the submit action is disabled
while the form is invalid or a submission is already in flight (`BR-006`).

## Data shapes

```ts
type RoleLabel = 'Admin' | 'Moderator' | 'User'
type RoleValue = 'admin' | 'moderator' | 'user'

// One-to-one mapping per BR-014, used at the form/API boundary.
const ROLE_LABEL_TO_VALUE: Record<RoleLabel, RoleValue> = {
  Admin: 'admin',
  Moderator: 'moderator',
  User: 'user',
}

// Formik form values (UI-facing; role kept as a display label).
interface UserFormValues {
  firstName: string
  lastName: string
  email: string
  phone: string
  role: RoleLabel
}

// Domain/API-facing user record (role kept as the lowercase API value).
interface User {
  id: number
  firstName: string
  lastName: string
  email: string
  phone: string
  role: RoleValue
}

// GET /users response shape (Section 12).
interface GetUsersResponse {
  users: User[]
  total: number
  skip: number
  limit: number
}

// POST /users/add and PATCH /users/{id} request body.
type UpsertUserRequest = Omit<User, 'id'>
```

## API and RTK Query contract

All endpoints below share `tagTypes: ['User']` on a single RTK Query API slice.
`getUsers` is the only endpoint that provides a cache entry; every mutation
performs a targeted, documented client-side cache update instead of
invalidating and refetching the full list — this avoids a second network
request (`REQ-003`) and lets the add flow control insertion position
(`BR-013`), satisfying `TECH-116`'s allowance for an explicitly documented
cache update strategy in place of tag invalidation.

### CON-020 — List users

- **Mode:** `MOCK`
- **Method and path:** `GET https://dummyjson.com/users?limit=0&select=id,firstName,lastName,email,phone,role`
- **Request:** None
- **Response:** `GetUsersResponse`
- **RTK Query endpoint:** `getUsers`
- **Provides tags:** `[{ type: 'User', id: 'LIST' }]`
- **Invalidates tags:** Not applicable (query)
- **Error behavior:** On failure, show the initial-load error state with a `Try again` action that refetches (`REQ-007`); the query is not retried automatically to avoid masking the error state
- **Covers:** `REQ-001`, `REQ-003`, `REQ-004`, `REQ-005`, `REQ-006`, `REQ-007`; `AC-001`, `AC-003`, `AC-004`, `AC-005`, `AC-006`, `AC-007`

### CON-021 — Add user

- **Mode:** `MOCK`
- **Method and path:** `POST https://dummyjson.com/users/add`
- **Request:** `UpsertUserRequest` (with `role` mapped to `RoleValue` per `BR-014`)
- **Response:** `User` (DummyJSON echoes the submitted fields with a generated `id`; not persisted server-side, per `BR-010`)
- **RTK Query endpoint:** `addUser`
- **Provides tags:** None
- **Invalidates tags:** None — on success, the handler inserts the returned `User` at the beginning of the cached `getUsers` list entry via `updateQueryData` and the page is reset to 1 (`BR-013`)
- **Error behavior:** On failure, keep the Add User form open with entered values and show a retryable error message (`REQ-016`)
- **Covers:** `REQ-008`, `REQ-009`, `BR-010`, `BR-013`, `BR-014`; `AC-008`, `AC-009`

### CON-022 — Update user

- **Mode:** `MOCK`
- **Method and path:** `PATCH https://dummyjson.com/users/{id}`
- **Request:** `UpsertUserRequest` (with `role` mapped to `RoleValue` per `BR-014`)
- **Response:** `User`
- **RTK Query endpoint:** `updateUser`
- **Provides tags:** None
- **Invalidates tags:** None — on success, the handler replaces the matching record in place within the cached `getUsers` list entry via `updateQueryData`, preserving the current page (`REQ-010`)
- **Error behavior:** On failure, keep the Edit User form open with entered values and show a retryable error message (`REQ-016`)
- **Covers:** `REQ-010`, `REQ-011`, `BR-010`, `BR-014`; `AC-011`, `AC-012`

### CON-023 — Delete user

- **Mode:** `MOCK`
- **Method and path:** `DELETE https://dummyjson.com/users/{id}`
- **Request:** None (`id` in path)
- **Response:** Confirmation payload from DummyJSON; only success/failure and the echoed `id` are relied upon (`POC-004` — the full response shape is not specified by the source and is not otherwise used)
- **RTK Query endpoint:** `deleteUser`
- **Provides tags:** None
- **Invalidates tags:** None — on success, the handler removes the matching record from the cached `getUsers` list entry via `updateQueryData` and the page/table hook recalculates pagination, moving to the previous valid page if the current page is emptied (`BR-009`)
- **Error behavior:** On failure, keep the record visible in the table and show a retryable error message (`REQ-016`)
- **Covers:** `REQ-012`, `REQ-013`, `REQ-014`, `BR-009`, `BR-011`; `AC-013`, `AC-014`, `AC-015`

## Global client-state contract

Use Redux Toolkit only for global client state not owned by Formik, the URL, or
RTK Query.

| Slice | State | Actions | Persistence |
| --- | --- | --- | --- |
| `Not needed` | — | — | — |

Pagination selection (current page, page size), the open/closed state of the
Add/Edit form and delete-confirmation dialog, and success/error notification
content are transient, single-page UI state. Per `ARCHITECTURE.md`'s
narrowest-state-mechanism guidance, these belong in component or feature-hook
state, not a Redux slice; the retrieved user collection itself lives only in
the RTK Query cache (`CON-020`), never duplicated into a slice or component
state (`TECH-114`).

## Important UI and hook boundaries

| Contract ID | Boundary | Inputs | Outputs/responsibility |
| --- | --- | --- | --- |
| `CON-030` | Feature orchestration hook for the `/users` page (name assigned in `PLAN.md`) | `getUsers` query result; administrator's page/page-size selections; delete-confirmation target | The current page's records, pagination summary/handlers, loading/empty/error state selection, and delete-confirmation open/confirm/cancel handling |
| `CON-031` | Add/Edit User form hook (name assigned in `PLAN.md`) | Optional user being edited; Formik submit event | Formik/Yup wiring per `CON-010`, Role label↔value mapping (`BR-014`), dispatching `addUser`/`updateUser`, and translating mutation success/failure into form state and notifications |

## Contract traceability

| Contract | Requirements | Acceptance criteria |
| --- | --- | --- |
| `CON-001` | `REQ-001`, `REQ-002`, `REQ-018`, `REQ-019` | `AC-001`, `AC-002`, `AC-018`, `AC-019` |
| `CON-010` | `REQ-008`, `REQ-009`, `REQ-010`, `REQ-011`, `REQ-015` | `AC-008`, `AC-009`, `AC-010`, `AC-011`, `AC-012` |
| `CON-020` | `REQ-001`, `REQ-003`, `REQ-004`, `REQ-005`, `REQ-006`, `REQ-007` | `AC-001`, `AC-003`, `AC-004`, `AC-005`, `AC-006`, `AC-007` |
| `CON-021` | `REQ-008`, `REQ-009`, `REQ-016`, `REQ-017` | `AC-008`, `AC-009`, `AC-016`, `AC-017` |
| `CON-022` | `REQ-010`, `REQ-011`, `REQ-016`, `REQ-017` | `AC-011`, `AC-012`, `AC-016`, `AC-017` |
| `CON-023` | `REQ-012`, `REQ-013`, `REQ-014`, `REQ-016`, `REQ-017` | `AC-013`, `AC-014`, `AC-015`, `AC-016`, `AC-017` |
| `CON-030` | `REQ-001`, `REQ-003`, `REQ-004`, `REQ-012`, `REQ-013`, `REQ-014` | `AC-001`, `AC-003`, `AC-004`, `AC-013`, `AC-014`, `AC-015` |
| `CON-031` | `REQ-008`, `REQ-009`, `REQ-010`, `REQ-011`, `REQ-015` | `AC-008`, `AC-009`, `AC-010`, `AC-011`, `AC-012` |

## Open contract decisions

| ID | Decision | Impact | Status |
| --- | --- | --- | --- |
| `CDEC-001` | Exact field shape of the DummyJSON delete response beyond success/`id` is unconfirmed by the source. | None for this POC — the feature does not depend on any field beyond confirming success. | `Non-blocking (POC)` |

## Human spec approval

- **Decision:** `APPROVED`
- **Reviewer:** Babak Khajavi
- **Date:** 2026-08-13
- **Comments:** Approved for planning. Route, form, data shapes, and API/RTK Query contracts are accepted as specified.
