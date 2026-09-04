# Specification: User Management Dashboard

## Document control

| Field | Value |
| --- | --- |
| Feature slug | `user-management` |
| Status | `READY_FOR_PLANNING` |
| Requirements baseline | `REQUIREMENTS.md`, human intake approval `APPROVED` by Babak Khajavi on 2026-08-13 |
| Human spec approval | `APPROVED` |

Acceptance criteria in this document use fresh sequential IDs (`AC-001`, `AC-002`, ...)
scoped to this specification. Where a criterion corresponds to an acceptance
criterion already identified in the original Confluence source, that source ID
is noted in parentheses for traceability back to `REQUIREMENTS.md`.

## Objective

Give an internal administrator a `/users` page to view, add, edit, and delete
user records in a paginated table, backed by the public DummyJSON Users API as
an explicit mock backend, so the AI-assisted SDD workflow can demonstrate a
complete, working, traceable feature to executive stakeholders. (`REQ-*`
overall; see `DEP-001` in `REQUIREMENTS.md`.)

## Scope

### In scope

- View users in a paginated table at `/users`, lazy-loaded. (`REQ-001`, `REQ-002`)
- Frontend-only pagination over a single, fully-fetched collection, with a page
  size of 5, 10, or 25 (default 10). (`REQ-003`, `REQ-004`)
- Add, edit, and delete a user via the mock API, with session-scoped cache
  updates. (`REQ-008`–`REQ-014`)
- Loading, empty, initial-error, validation, and mutation-error states.
  (`REQ-005`–`REQ-007`, `REQ-015`, `REQ-016`)
- A visible notice that changes persist only for the current browser session.
  (`REQ-017`)
- Accessible interaction and a responsive layout. (`REQ-018`, `REQ-019`)

### Out of scope

- Authentication or authorization.
- A production backend or database; permanent persistence after a browser refresh.
- Server-side pagination, sorting, or filtering.
- User profile pages, bulk edit/delete, file or avatar uploads.
- Production deployment, audit history, internationalization.

(`REQUIREMENTS.md` Scope section; unchanged from intake.)

## User flows

### Flow 1 — View and paginate users

1. The administrator navigates to `/users`; the route module loads lazily.
2. The application fetches the complete user collection once and shows page 1
   (10 records by default), with the record range/total (e.g., `1–10 of 208`).
3. The administrator changes page or page size; the table re-renders from the
   already-fetched collection with no further list request.

Covers: `REQ-001`–`REQ-004`

### Flow 2 — Add a user

1. The administrator selects `Add user` and completes First name, Last name,
   Email, Phone, and Role.
2. The administrator submits; invalid fields block submission with inline
   messages.
3. On success, the new user is created via the mock API, inserted at the
   beginning of the cached collection, the table resets to page 1, a
   `User added successfully` notification appears, and the form closes.

Covers: `REQ-008`, `REQ-009`, `REQ-015`

### Flow 3 — Edit a user

1. The administrator selects `Edit` on a row; the form opens prepopulated,
   with its primary action labeled `Save changes`.
2. The administrator submits valid changes.
3. On success, the user is updated via the mock API, the table reflects the
   new values, the current page is preserved where possible, a
   `User updated successfully` notification appears, and the form closes.

Covers: `REQ-010`, `REQ-011`, `REQ-015`

### Flow 4 — Delete a user

1. The administrator selects `Delete` on a row; a confirmation dialog naming
   the user appears.
2. The administrator confirms (or cancels, which closes the dialog with no
   change).
3. On confirmation, the user is deleted via the mock API, removed from the
   table, pagination is recalculated (moving to the previous valid page if the
   current page is emptied), a `User deleted successfully` notification
   appears, and the dialog closes.

Covers: `REQ-012`–`REQ-014`

## Behavioral states

| State | Expected behavior | Related requirements |
| --- | --- | --- |
| Initial | Route mounts and immediately begins fetching the full user collection. | `REQ-001`, `REQ-002` |
| Loading | Accessible loading indicator shown; page title stays visible; no empty-table message; no duplicate requests from re-rendering. | `REQ-005`, `NFR-001`, `TECH-060` |
| Empty | `No users found` is shown; `Add user` remains available; no empty row markup rendered. | `REQ-006`, `TECH-060` |
| Success | Table shows the current page's records across Name, Email, Phone, Role, Actions; pagination controls reflect the current range/total. | `REQ-001`, `REQ-003`, `REQ-004` |
| Validation error | Field-level messages appear after a field is touched or on submit attempt; submission is blocked while invalid or in flight. | `REQ-015`, `BR-001`–`BR-006`, `TECH-103` |
| Service error | Initial load failure shows a retryable banner (`REQ-007`); a failed add/edit/delete keeps the form or record and shows a retryable, understandable message without discarding input (`REQ-016`). | `REQ-007`, `REQ-016`, `TECH-061` |

Permission states are not applicable — the feature has no authentication or
authorization (explicitly out of scope).

## Acceptance criteria

### AC-001 — Users load and display

- **Given:** the administrator opens `/users`
- **When:** the mock API list request succeeds
- **Then:** the page header (`User Management`, `View and manage users`,
  `Add user`) and a table with Name, Email, Phone, Role, and Actions columns render
- **Covers:** `REQ-001`
- **Verification:** `Automated` (source `AC-001`)

### AC-002 — Route code-splits

- **Given:** the administrator loads the application
- **When:** they navigate to `/users` for the first time
- **Then:** the User Management route implementation loads as its own
  route-level bundle, separate from the initial application bundle
- **Covers:** `REQ-002`
- **Verification:** `Automated` (production build chunk inspection) (source `AC-002`)

### AC-003 — Pagination uses only the already-fetched collection

- **Given:** the full user collection has loaded
- **When:** the administrator changes pages
- **Then:** the correct records for the new page display and no additional
  users-list request is made
- **Covers:** `REQ-003`
- **Verification:** `Automated` (source `AC-003`)

### AC-004 — Page size selection

- **Given:** the table is displayed
- **When:** the administrator selects a page size of 5, 10, or 25
- **Then:** the table shows that many rows and returns to page 1
- **Covers:** `REQ-004`, `BR-007`, `BR-008`
- **Verification:** `Automated` (source `AC-004`)

### AC-005 — Accessible loading state

- **Given:** the initial users request is pending
- **When:** the page renders
- **Then:** an accessible loading indicator is shown, the page title remains
  visible, no empty-table message appears, and re-renders do not trigger a
  second request
- **Covers:** `REQ-005`, `NFR-001`
- **Verification:** `Automated` (source `AC-012`)

### AC-006 — Empty state

- **Given:** the locally displayed collection has no users
- **When:** the table area renders
- **Then:** `No users found` is shown and `Add user` remains available
- **Covers:** `REQ-006`
- **Verification:** `Automated` (source `AC-015`)

### AC-007 — Initial load error and retry

- **Given:** the initial users request fails
- **When:** the error state appears and the administrator selects `Try again`
- **Then:** the application retries the request
- **Covers:** `REQ-007`
- **Verification:** `Automated` (source `AC-013`)

### AC-008 — Add a user

- **Given:** valid values are entered in the Add User form
- **When:** the administrator submits
- **Then:** the mock add-user API is called, the returned user is inserted at
  the beginning of the collection, the table shows page 1 with the new record
  as the first row, `User added successfully` appears, and the form closes
- **Covers:** `REQ-008`, `BR-010`, `BR-013`, `BR-014`
- **Verification:** `Automated` (source `AC-005`)

### AC-009 — Cancel add

- **Given:** the Add User form is open
- **When:** the administrator selects `Cancel`
- **Then:** the form closes and no user is created
- **Covers:** `REQ-009`
- **Verification:** `Automated`

### AC-010 — Reject invalid form submission

- **Given:** one or more Add/Edit User form values are invalid per
  `BR-001`–`BR-006`
- **When:** the administrator attempts to submit
- **Then:** submission is prevented and field-level validation messages are
  displayed
- **Covers:** `REQ-015`, `BR-001`–`BR-006`
- **Verification:** `Automated` (source `AC-006`)

### AC-011 — Edit a user

- **Given:** an existing user is selected for edit and valid changes are made
- **When:** the administrator submits
- **Then:** the mock update-user API is called, the updated values appear in
  the table, and the current page is preserved where possible
- **Covers:** `REQ-010`, `BR-010`, `BR-014`
- **Verification:** `Automated` (source `AC-007`)

### AC-012 — Cancel edit

- **Given:** the Edit User form is open
- **When:** the administrator selects `Cancel`
- **Then:** the dialog closes and the user is unchanged
- **Covers:** `REQ-011`
- **Verification:** `Automated` (source `AC-008`)

### AC-013 — Delete requires confirmation

- **Given:** the administrator selects `Delete` on a row
- **When:** the confirmation dialog appears, naming the user
- **Then:** the record is not removed until the administrator confirms
- **Covers:** `REQ-012`, `BR-011`
- **Verification:** `Automated` (source `AC-009`)

### AC-014 — Cancel delete

- **Given:** the delete confirmation dialog is open
- **When:** the administrator selects `Cancel`
- **Then:** the dialog closes and the record is not deleted
- **Covers:** `REQ-013`
- **Verification:** `Automated`

### AC-015 — Confirmed delete and pagination recalculation

- **Given:** deletion is confirmed
- **When:** the mock delete-user API succeeds
- **Then:** the user is removed from the table, and if that emptied the
  current page, the table moves to the nearest valid page
- **Covers:** `REQ-014`, `BR-009`
- **Verification:** `Automated` (source `AC-010`, `AC-011`)

### AC-016 — Mutation failure preserves input

- **Given:** an add, edit, or delete request fails
- **When:** the error is returned
- **Then:** the relevant form or record remains available and an
  understandable, retryable error message is shown without discarding entered
  values
- **Covers:** `REQ-016`
- **Verification:** `Automated` (source `AC-014`)

### AC-017 — Session-only notice

- **Given:** the User Management page is displayed
- **When:** it renders
- **Then:** a notice communicates that changes persist only for the current
  browser session
- **Covers:** `REQ-017`, `BR-010`
- **Verification:** `Automated` (source `AC-016`)

### AC-018 — Accessible interaction

- **Given:** the table, forms, and dialogs are rendered
- **When:** interacted with via keyboard or assistive technology
- **Then:** semantic table markup is used; row actions have accessible names
  identifying the user (e.g., `Edit Emily Johnson`); forms are keyboard
  operable; dialog focus is managed and returned on close; validation and
  status messages are associated with their fields and exposed to assistive
  technology; destructive actions require confirmation; no information relies
  on color alone
- **Covers:** `REQ-018`, `BR-012`
- **Verification:** `Manual POC demonstration` plus automated accessible-name/role assertions

### AC-019 — Responsive layout

- **Given:** the page is viewed at desktop and small-screen widths
- **When:** rendered
- **Then:** desktop shows all columns with actions in the final column; small
  screens keep the table usable, scope any horizontal scrolling to the table
  container (never the page), and keep dialogs/forms within the viewport
- **Covers:** `REQ-019`, `NFR-002`
- **Verification:** `Manual POC demonstration`

## POC implementation assumptions

| ID | Assumption or shortcut | Reason | Demo effect | Production follow-up |
| --- | --- | --- | --- | --- |
| `POC-001` | The public DummyJSON Users API (`https://dummyjson.com`) is used as the entire backend, in `MOCK` mode. | Explicitly mandated by the source as the feature's mock backend (`DEP-001`). | List/add/edit/delete calls succeed and look real, but nothing is stored on a real server. | Replace with a real backend service and authentication before any production use. |
| `POC-002` | Added/edited/deleted records are reflected only in the RTK Query cache for the current session; a browser refresh may restore DummyJSON's original dataset. | DummyJSON does not persist mutations server-side (`BR-010`, `DEP-002`); accepted for this POC (`Q-004`). | The live demo must avoid refreshing mid-demonstration, or expect the dataset to reset. | Implement real persistence via a production backend. |
| `POC-003` | No authentication or authorization is implemented; `/users` is reachable by anyone with the URL. | Explicitly out of scope for this POC. | Acceptable for an internal demo; not acceptable for any shared or public environment. | Add authentication/authorization before wider release. |
| `POC-004` | The delete-user mock response's exact field shape beyond a success indicator and the deleted `id` is not specified by the source and is not otherwise relied upon by the feature. | Source Section 12 documents list/add/update payload shapes in detail but not the delete response shape. | No demo impact — the feature only needs to know the delete succeeded. | Confirm and type the real backend's delete response when one exists. |

## Applicable technical requirements

- `TECH-001`, `TECH-002` — React + TypeScript on Vite with a repeatable production build.
- `TECH-005`–`TECH-008` — `/users` is a lazy-loaded route with an accessible Suspense fallback and a recoverable route-loading error state.
- `TECH-090`–`TECH-094` — MUI is the only styling system; the table uses MUI Table/TablePagination; `CssBaseline` applies.
- `TECH-100`–`TECH-104` — Add/Edit User form uses Formik 2 + Yup with accessible, field-associated validation.
- `TECH-110`–`TECH-119` — List/add/update/delete use RTK Query with typed requests/responses and a documented cache update strategy.
- `TECH-120`–`TECH-123` — Orchestration, submission, and workflow logic live in feature custom hooks, not page/presentation components.
- `TECH-010`–`TECH-013` — Type checking, linting, tests, and production build must pass before the feature is presented as done.
- `TECH-020`–`TECH-024` — WCAG 2.2 AA target underlies `REQ-018`'s feature-specific accessibility detail.
- `TECH-030`–`TECH-031` — Usable, non-horizontally-scrolling layout on mobile and desktop.
- `TECH-040`–`TECH-043` — Route-level code splitting for `/users` must survive the production build.
- `TECH-060`–`TECH-062` — Loading/empty/success/failure states are intentional; user input is never silently discarded.
- `TECH-070`–`TECH-072` — Each requirement maps to automated evidence or documented manual verification.
- `TECH-080`–`TECH-082` — This document's IDs are referenced by `PLAN.md`, Jira work, tests, and review evidence.

## Open decisions

All decisions raised during intake (`Q-001`–`Q-005`) were resolved by the human
approver before this specification was written; see `REQUIREMENTS.md` for the
recorded answers. No open decision remains for planning.

| ID | Decision needed | Impact | Owner | Status |
| --- | --- | --- | --- | --- |
| — | None remaining | — | — | `CLOSED` |

## Traceability

| Requirement | Acceptance criteria | Contract references |
| --- | --- | --- |
| `REQ-001` | `AC-001` | `CON-001`, `CON-020` |
| `REQ-002` | `AC-002` | `CON-001` |
| `REQ-003` | `AC-003` | `CON-020` |
| `REQ-004` | `AC-004` | `CON-020` |
| `REQ-005` | `AC-005` | `CON-020` |
| `REQ-006` | `AC-006` | `CON-020` |
| `REQ-007` | `AC-007` | `CON-020` |
| `REQ-008` | `AC-008` | `CON-010`, `CON-021` |
| `REQ-009` | `AC-009` | `CON-010` |
| `REQ-010` | `AC-011` | `CON-010`, `CON-022` |
| `REQ-011` | `AC-012` | `CON-010` |
| `REQ-012` | `AC-013` | `CON-023` |
| `REQ-013` | `AC-014` | `CON-023` |
| `REQ-014` | `AC-015` | `CON-023` |
| `REQ-015` | `AC-010` | `CON-010` |
| `REQ-016` | `AC-016` | `CON-021`, `CON-022`, `CON-023` |
| `REQ-017` | `AC-017` | `CON-020`, `CON-021`, `CON-022`, `CON-023` |
| `REQ-018` | `AC-018` | `CON-001`, `CON-010`, `CON-023` |
| `REQ-019` | `AC-019` | `CON-001` |

## Human spec approval

- **Decision:** `APPROVED`
- **Reviewer:** Babak Khajavi
- **Date:** 2026-08-13
- **Comments:** Approved for planning. Behavior, states, and acceptance criteria are accepted as specified.

Approval authorizes architecture impact analysis and planning only.
