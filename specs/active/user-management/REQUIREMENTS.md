# Requirements: User Management Dashboard

## Document control

| Field | Value |
| --- | --- |
| Feature slug | `user-management` |
| Status | `READY_FOR_SPEC` |
| Intake date | `2026-08-13T13:48:15Z` |
| Prepared by | `Intake Agent` |
| Human intake approval | `APPROVED` |
| Approved by | Babak Khajavi |
| Approval date | 2026-08-13 |

## Sources

| Source ID | Page title | URL or page ID | Version | Last updated | Sections used |
| --- | --- | --- | --- | --- | --- |
| `SRC-001` | `User Management Dashboard: AI-assisted Spec-Driven Development for React Application` | `https://smoothpay.atlassian.net/wiki/spaces/~6104334d3900e60070b6d9bb/pages/3154247681` (page ID `3154247681`) | `Not exposed by the configured Confluence integration` | `Reported by the integration as "14 minutes ago" relative to intake time (2026-08-13T13:48:15Z); page status "current". Absolute last-updated timestamp is not exposed by the tool.` | Sections 1–20 (entire page) |

No child or linked Confluence pages were read. The page links only to external DummyJSON API documentation (`https://dummyjson.com/docs/users`, `https://dummyjson.com/docs`), which is not Confluence source material and is treated as external reference only, not additional approved scope.

## Business objective

Demonstrate that an AI-assisted, agentic coding workflow can take a Confluence
product requirement — a User Management Dashboard — through requirements
extraction, specification, planning, Jira task creation, and implementation in
a React application, producing a working paginated user table with add, edit,
and delete capability. The feature itself is a proof-of-concept vehicle for
demonstrating API integration, forms, validation, state management, caching,
and responsive UI behavior to Chief of Staff and executive stakeholders.
(SRC-001: Sections 2, 3)

## Users and stakeholders

| Actor | Need or responsibility | Source |
| --- | --- | --- |
| Administrator (primary user) | Maintains a directory of users: views, adds, edits, and deletes user records | `SRC-001: Section 4` |
| Executives, technical leads, and stakeholders (secondary audience) | Evaluate the practical value, governance, and transparency of the AI-assisted SDD workflow by observing the demonstration | `SRC-001: Section 4, 18` |

## Scope

### In scope

- View users in a table.
- Navigate through users using frontend pagination.
- Choose how many users are displayed per page.
- Add a user.
- Edit an existing user.
- Delete a user after confirmation.
- See loading, empty, success, validation, and error states.
- Retain added, edited, and deleted records during the current browser session.

(SRC-001: Section 5.1)

### Out of scope

- Authentication or authorization.
- A production backend or database.
- Permanent persistence after a browser refresh.
- Server-side pagination.
- User profile pages.
- Bulk editing or bulk deletion.
- File or avatar uploads.
- Sorting or filtering.
- Production deployment.
- Audit history.
- Internationalization.

(SRC-001: Section 5.2)

## User journeys

### Journey 1 — View and paginate users

1. The administrator opens `/users`.
2. The application retrieves the complete user collection from the mock API and displays the first page (10 records by default).
3. The administrator changes pages or changes the page size; the table updates from the already-retrieved collection without another list request.

Source: `SRC-001: Section 7.1, 7.3`

### Journey 2 — Add a user

1. The administrator selects `Add user` from the page header.
2. The administrator completes the required fields and submits the form, or cancels.
3. On successful submission, the new user is created via the mock API, added to the displayed collection, a success notification appears, the form closes, and the new record is made visible.

Source: `SRC-001: Section 8`

### Journey 3 — Edit a user

1. The administrator selects `Edit` on a table row.
2. The form opens prepopulated with that user's current values, with its primary action labeled `Save changes`.
3. On successful submission, the user is updated via the mock API, the table reflects the new values, a success notification appears, the form closes, and the current page is preserved where possible.

Source: `SRC-001: Section 9`

### Journey 4 — Delete a user

1. The administrator selects `Delete` on a table row.
2. A confirmation dialog naming the user appears; the record is not removed until the administrator confirms.
3. On confirmation, the user is deleted via the mock API, removed from the displayed collection, a success notification appears, the dialog closes, and pagination is recalculated (moving to the previous valid page if the current page becomes empty).

Source: `SRC-001: Section 10`

## Functional requirements

### REQ-001 — View users in a paginated table

- **Statement:** The system must display the User Management Dashboard at route `/users`, with a page header (title `User Management`, supporting text `View and manage users`, primary action `Add user`) and a table with columns Name, Email, Phone, Role, and Actions.
- **Actor:** Administrator
- **Trigger:** Administrator navigates to `/users`.
- **Expected outcome:** The page header and user table render with the specified columns.
- **Rationale:** Core viewing capability for the feature.
- **Priority:** Not specified
- **Source:** `SRC-001: Section 7.1, 7.2, 7.3`
- **Original source ID:** `AC-001`
- **Acceptance evidence:** Given the administrator opens `/users`, when the API request succeeds, then users are displayed in the table with the specified columns.

### REQ-002 — Lazy-loaded route

- **Statement:** The system must load the `/users` route implementation as a separate route-level bundle, not as part of the initial application bundle.
- **Actor:** System
- **Trigger:** Administrator navigates to `/users`.
- **Expected outcome:** The User Management route code is fetched as its own chunk on first navigation to the route.
- **Rationale:** Keeps the initial bundle small; consistent with the application's lazy-route architecture.
- **Priority:** Not specified
- **Source:** `SRC-001: Section 7.1`
- **Original source ID:** `AC-002`
- **Acceptance evidence:** Given the administrator initially loads the application, when they navigate to `/users`, then the User Management route loads as a separate route-level bundle.

### REQ-003 — Frontend-only pagination after initial load

- **Statement:** The system must retrieve the complete user collection once from the mock API and perform all subsequent pagination in the browser without further list requests.
- **Actor:** System
- **Trigger:** Administrator changes the displayed page.
- **Expected outcome:** Correct records for the selected page are shown; no additional users-list API request is made.
- **Rationale:** Explicit product requirement to avoid server-side pagination for this POC.
- **Priority:** Not specified
- **Source:** `SRC-001: Section 7.3`
- **Original source ID:** `AC-003`
- **Acceptance evidence:** Given the user collection has loaded, when the administrator changes pages, then the correct records display without another users-list API request.

### REQ-004 — Page size selection

- **Statement:** The system must let the administrator choose a page size of 5, 10, or 25 records, defaulting to 10, and must display the current record range and total count (e.g., `1–10 of 208`) with next/previous controls.
- **Actor:** Administrator
- **Trigger:** Administrator selects a page size option.
- **Expected outcome:** The table shows the selected number of records per page and returns to the first page.
- **Rationale:** Explicit pagination control requirement.
- **Priority:** Not specified
- **Source:** `SRC-001: Section 7.3`
- **Original source ID:** `AC-004`
- **Acceptance evidence:** Given the table is displayed, when the administrator selects 5, 10, or 25 rows per page, then the table shows that many records and returns to the first page.

### REQ-005 — Loading state

- **Statement:** The system must display a visible, accessible loading indicator while the initial user collection is being retrieved, without showing an empty-table message, while keeping the page title visible.
- **Actor:** System
- **Trigger:** The initial users request is pending.
- **Expected outcome:** Loading indicator is shown; no empty-state message; no duplicate requests from repeated rendering.
- **Rationale:** Explicit loading-state requirement.
- **Priority:** Not specified
- **Source:** `SRC-001: Section 7.4`
- **Original source ID:** `AC-012`
- **Acceptance evidence:** Given the users request is pending, when the page renders, then an accessible loading state is displayed.

### REQ-006 — Empty state

- **Statement:** The system must display `No users found` when the locally displayed collection has no records, while keeping the `Add user` action available and not rendering an empty collection of table rows.
- **Actor:** System
- **Trigger:** The displayed user collection is empty.
- **Expected outcome:** Empty-state message and `Add user` action are shown; no empty row markup.
- **Rationale:** Explicit empty-state requirement.
- **Priority:** Not specified
- **Source:** `SRC-001: Section 7.5`
- **Original source ID:** `AC-015`
- **Acceptance evidence:** Given the locally displayed collection contains no users, when the table area renders, then `No users found` and the Add User action are available.

### REQ-007 — Initial load error state with retry

- **Statement:** The system must display `We couldn't load users. Please try again.` with a `Try again` action if the initial users request fails, without showing stale or misleading success content.
- **Actor:** System / Administrator
- **Trigger:** The initial users request fails.
- **Expected outcome:** Error message and retry action are shown; selecting `Try again` retries the request.
- **Rationale:** Explicit initial-error-state requirement.
- **Priority:** Not specified
- **Source:** `SRC-001: Section 7.6`
- **Original source ID:** `AC-013`
- **Acceptance evidence:** Given the users request fails, when the error state appears and the administrator selects Try Again, then the application retries the request.

### REQ-008 — Add a user

- **Statement:** The system must let the administrator open an accessible Add User form (First name, Last name, Email, Phone, Role, with `Cancel` and `Add user` actions) and, on successful submission, call the mock API add-user endpoint, insert the returned user at the beginning of the displayed collection per `BR-013`, show `User added successfully`, close the form, and reset the table to the first page so the new record is immediately visible.
- **Actor:** Administrator
- **Trigger:** Administrator selects `Add user` and submits valid values.
- **Expected outcome:** New user is created via the mock API, becomes the first record in the collection, and is visible on page 1 during the session.
- **Rationale:** Core create capability.
- **Priority:** Not specified
- **Source:** `SRC-001: Section 8`
- **Original source ID:** `AC-005`
- **Acceptance evidence:** Given valid values are entered, when the administrator submits the Add User form, then the add API is called, the returned user is inserted at the beginning of the collection, and the table shows page 1 with the new record as the first row.

### REQ-009 — Cancel add without creating

- **Statement:** The system must close the Add User form without creating a user when the administrator selects `Cancel`.
- **Actor:** Administrator
- **Trigger:** Administrator selects `Cancel` in the Add User form.
- **Expected outcome:** Form closes; no user is created.
- **Rationale:** Explicit cancel-safety requirement.
- **Priority:** Not specified
- **Source:** `SRC-001: Section 8`
- **Original source ID:** `None`
- **Acceptance evidence:** Given the Add User form is open, when Cancel is selected, then the form closes and no user is created.

### REQ-010 — Edit a user

- **Statement:** The system must let the administrator open the user form prepopulated with a selected user's current values (primary action labeled `Save changes`) and, on successful submission, call the mock API update-user endpoint, update the user in the displayed collection, show `User updated successfully`, close the form, and preserve the current table page where possible.
- **Actor:** Administrator
- **Trigger:** Administrator selects `Edit` on a row and submits valid changes.
- **Expected outcome:** User is updated via the mock API and reflected in the table during the session.
- **Rationale:** Core update capability.
- **Priority:** Not specified
- **Source:** `SRC-001: Section 9`
- **Original source ID:** `AC-007`
- **Acceptance evidence:** Given an existing user is selected, when valid changes are submitted, then the update API is called and the updated values appear in the table during the current session.

### REQ-011 — Cancel edit without changes

- **Statement:** The system must close the Edit User form without changing the user when the administrator selects `Cancel`.
- **Actor:** Administrator
- **Trigger:** Administrator selects `Cancel` in the Edit User form.
- **Expected outcome:** Form closes; the user record is unchanged.
- **Rationale:** Explicit cancel-safety requirement.
- **Priority:** Not specified
- **Source:** `SRC-001: Section 9`
- **Original source ID:** `AC-008`
- **Acceptance evidence:** Given the Edit User form is open, when Cancel is selected, then the dialog closes and the user remains unchanged.

### REQ-012 — Delete requires confirmation

- **Statement:** The system must open a confirmation dialog naming the user (`Delete <first name> <last name>?` plus explanatory text) with `Cancel` and `Delete` actions when the administrator selects `Delete` on a row, and must not remove the record until the administrator confirms.
- **Actor:** Administrator
- **Trigger:** Administrator selects `Delete` on a row.
- **Expected outcome:** Confirmation dialog appears; record is not removed until confirmed.
- **Rationale:** Destructive-action safety requirement.
- **Priority:** Not specified
- **Source:** `SRC-001: Section 10`
- **Original source ID:** `AC-009`
- **Acceptance evidence:** Given Delete is selected, when the confirmation dialog appears, then the record is not removed until the administrator confirms the action.

### REQ-013 — Cancel delete without removing

- **Statement:** The system must close the delete confirmation dialog without deleting the record when the administrator selects `Cancel`.
- **Actor:** Administrator
- **Trigger:** Administrator selects `Cancel` in the delete confirmation dialog.
- **Expected outcome:** Dialog closes; record is not deleted.
- **Rationale:** Explicit cancel-safety requirement.
- **Priority:** Not specified
- **Source:** `SRC-001: Section 10`
- **Original source ID:** `None`
- **Acceptance evidence:** Given the delete confirmation dialog is open, when Cancel is selected, then the dialog closes and the record is not deleted.

### REQ-014 — Confirmed delete removes the record and recalculates pagination

- **Statement:** On confirmed deletion, the system must call the mock API delete-user endpoint, remove the user from the displayed collection, show `User deleted successfully`, close the dialog, recalculate pagination, and move to the previous valid page if the current page becomes empty.
- **Actor:** System
- **Trigger:** Administrator confirms deletion.
- **Expected outcome:** User is removed from the table during the session; pagination state remains valid.
- **Rationale:** Core delete capability with correct pagination bookkeeping.
- **Priority:** Not specified
- **Source:** `SRC-001: Section 10`
- **Original source ID:** `AC-010`, `AC-011`
- **Acceptance evidence:** Given deletion is confirmed, when the mock API succeeds, then the user is removed from the table during the current session, and given deletion removes the final record on the current page, when the cache is updated, then the table navigates to the nearest valid page.

### REQ-015 — Field-level form validation blocks invalid submission

- **Statement:** The system must validate all Add/Edit User form fields per the rules in `BR-001` through `BR-006`, display field-level validation messages, and prevent submission while the form is invalid or while another submission is in progress.
- **Actor:** System
- **Trigger:** Administrator touches a field or attempts to submit the form.
- **Expected outcome:** Invalid submissions are blocked and understandable, field-associated validation messages are shown.
- **Rationale:** Explicit validation requirement.
- **Priority:** Not specified
- **Source:** `SRC-001: Section 11`
- **Original source ID:** `AC-006`
- **Acceptance evidence:** Given one or more form values are invalid, when the administrator attempts submission, then submission is prevented and field-level validation messages are displayed.

### REQ-016 — Mutation failure handling preserves input

- **Statement:** If an add, update, or delete request fails, the system must keep the relevant form or record available, display an understandable, retryable error message, and must not silently discard entered form values.
- **Actor:** System
- **Trigger:** An add, update, or delete mock API request fails.
- **Expected outcome:** User input/record is preserved; a retryable error is shown.
- **Rationale:** Explicit mutation-failure requirement.
- **Priority:** Not specified
- **Source:** `SRC-001: Section 7.6`
- **Original source ID:** `AC-014`
- **Acceptance evidence:** Given an add, edit, or delete request fails, when the error is returned, then the application preserves the relevant user input or record and provides a retryable error message.

### REQ-017 — Session-only persistence notice

- **Statement:** The system must display a notice (`Demo mode: Changes are retained for this browser session only.`) communicating that added, edited, and deleted records persist only for the current browser session because the mock API does not permanently change its server-side dataset.
- **Actor:** System
- **Trigger:** The User Management page is displayed.
- **Expected outcome:** Session-only notice is visible.
- **Rationale:** Explicit requirement to avoid implying permanent storage.
- **Priority:** Not specified
- **Source:** `SRC-001: Section 13`
- **Original source ID:** `AC-016`
- **Acceptance evidence:** Given the application is using DummyJSON, when the User Management page is displayed, then it communicates that changes persist only for the current browser session.

### REQ-018 — Accessible interaction

- **Statement:** The system must use semantic table markup; give Add, Edit, and Delete controls accessible names that identify the related user (per `BR-012`); ensure forms are keyboard accessible; manage dialog focus correctly (including returning focus after a dialog closes); associate validation messages with their fields; expose loading and success/error messages to assistive technology; require explicit confirmation for destructive actions; and not rely on color alone to convey information.
- **Actor:** System
- **Trigger:** Any user interaction with the table, forms, or dialogs.
- **Expected outcome:** All stated accessibility behaviors are observable via keyboard and assistive technology.
- **Rationale:** Explicit accessibility requirements beyond the general WCAG 2.2 AA target (see `TECH-020`–`TECH-024`).
- **Priority:** Not specified
- **Source:** `SRC-001: Section 15`
- **Original source ID:** `None`
- **Acceptance evidence:** Manual and automated accessibility checks confirm semantic markup, accessible names (e.g., `Edit Emily Johnson`, `Delete Emily Johnson`), keyboard operability, dialog focus management, and field/error association.

### REQ-019 — Responsive layout

- **Statement:** On desktop, the system must display all table columns with Edit/Delete actions in the final column. On small screens, the table must remain usable without losing access to actions; horizontal scrolling is acceptable inside the table container but the page itself must not have unintended horizontal scrolling; dialogs and forms must fit within the viewport; primary actions must remain easily accessible.
- **Actor:** System
- **Trigger:** The page is viewed at a desktop or small-screen viewport.
- **Expected outcome:** Table, dialogs, and forms remain usable and correctly scoped at supported viewport widths.
- **Rationale:** Explicit responsive-behavior requirement.
- **Priority:** Not specified
- **Source:** `SRC-001: Section 16`
- **Original source ID:** `None`
- **Acceptance evidence:** Manual verification at desktop and small-screen viewport widths confirms the stated layout behavior; see also `TECH-030`, `TECH-031`.

## Business rules

### BR-001 — First name validation

- **Rule:** First name is required (leading/trailing whitespace ignored when checking emptiness) and must be no more than 50 characters. Messages: `First name is required`, `First name must be 50 characters or fewer`.
- **Applies to:** `REQ-015`, `REQ-008`, `REQ-010`
- **Source:** `SRC-001: Section 11`
- **Verification evidence:** Submitting an empty or >50-character first name shows the corresponding message and blocks submission.

### BR-002 — Last name validation

- **Rule:** Last name is required (leading/trailing whitespace ignored when checking emptiness) and must be no more than 50 characters. Messages: `Last name is required`, `Last name must be 50 characters or fewer`.
- **Applies to:** `REQ-015`, `REQ-008`, `REQ-010`
- **Source:** `SRC-001: Section 11`
- **Verification evidence:** Submitting an empty or >50-character last name shows the corresponding message and blocks submission.

### BR-003 — Email validation

- **Rule:** Email is required, must be a valid email format, and must be no more than 100 characters. Messages: `Email is required`, `Enter a valid email address`, `Email must be 100 characters or fewer`.
- **Applies to:** `REQ-015`, `REQ-008`, `REQ-010`
- **Source:** `SRC-001: Section 11`
- **Verification evidence:** Submitting an empty, malformed, or >100-character email shows the corresponding message and blocks submission.

### BR-004 — Phone validation

- **Rule:** Phone is required and must be 7–25 characters, containing only numbers, spaces, parentheses, hyphens, and an optional leading plus sign. Messages: `Phone number is required`, `Enter a valid phone number`.
- **Applies to:** `REQ-015`, `REQ-008`, `REQ-010`
- **Source:** `SRC-001: Section 11`
- **Verification evidence:** Submitting an empty or out-of-pattern phone value shows the corresponding message and blocks submission.

### BR-005 — Role validation

- **Rule:** Role is required and must be one of Admin, Moderator, or User (displayed labels; see `BR-014` for the label-to-value mapping used in submitted payloads). Message: `Role is required`.
- **Applies to:** `REQ-015`, `REQ-008`, `REQ-010`
- **Source:** `SRC-001: Section 6, 11`
- **Verification evidence:** Submitting without a selected role shows the message and blocks submission.

### BR-006 — Validation timing and submission guard

- **Rule:** Field validation runs after a field has been touched and again on submit; the form must not submit while invalid or while a submission is already in progress.
- **Applies to:** `REQ-015`
- **Source:** `SRC-001: Section 11`
- **Verification evidence:** Untouched invalid fields show no premature error; submit is blocked during an in-flight submission or while any field is invalid.

### BR-007 — Default and available page sizes

- **Rule:** The table's default page size is 10 records; the only available page sizes are 5, 10, and 25.
- **Applies to:** `REQ-004`
- **Source:** `SRC-001: Section 7.3`
- **Verification evidence:** Page size control offers exactly 5/10/25, initially set to 10.

### BR-008 — Page size change resets to first page

- **Rule:** Changing the page size returns the table to the first page.
- **Applies to:** `REQ-004`
- **Source:** `SRC-001: Section 7.3`
- **Verification evidence:** Selecting a new page size while on page 3, for example, returns the view to page 1.

### BR-009 — Pagination recalculation after deletion

- **Rule:** If deleting a record empties the current page, the table moves to the nearest valid (previous) page.
- **Applies to:** `REQ-014`
- **Source:** `SRC-001: Section 7.3, 10`
- **Verification evidence:** Deleting the sole remaining record on the last page moves the view to the new last valid page.

### BR-010 — Mutations are session-scoped only

- **Rule:** Because the mock API does not permanently change its server-side dataset, all successful add/edit/delete mutations must be reflected via the application's RTK Query cache and remain correct while navigating within the application for the current session; a browser refresh may restore the original dataset, and the application must not imply permanent storage.
- **Applies to:** `REQ-008`, `REQ-010`, `REQ-014`, `REQ-017`
- **Source:** `SRC-001: Section 13`
- **Verification evidence:** Adding, editing, or deleting a user persists correctly across in-app navigation without a page refresh; the session-only notice is visible.

### BR-011 — Delete confirmation must name the affected user

- **Rule:** The delete confirmation dialog must display `Delete <first name> <last name>?` plus explanatory text, and must not delete until the administrator confirms.
- **Applies to:** `REQ-012`
- **Source:** `SRC-001: Section 10`
- **Verification evidence:** Dialog text includes the specific user's first and last name.

### BR-012 — Accessible name pattern for row actions

- **Rule:** Icon-only Edit/Delete row actions must have accessible names that identify the related user, following the pattern `Edit <first name> <last name>` / `Delete <first name> <last name>`.
- **Applies to:** `REQ-018`
- **Source:** `SRC-001: Section 15`
- **Verification evidence:** Accessible-name queries (e.g., ARIA/accessibility tree) return `Edit Emily Johnson`, `Delete Emily Johnson`, etc., for the corresponding row.

### BR-013 — New record insertion position and visibility

- **Rule:** On successful add, the new user must be inserted at the beginning of the locally cached collection, and the table must reset to the first page so the new record is immediately visible.
- **Applies to:** `REQ-008`
- **Source:** `SRC-001: Section 8`, resolved by human decision on `Q-003` (2026-08-13T13:51:37Z)
- **Verification evidence:** After adding a user, the table shows page 1 with the new record as the first row.

### BR-014 — Role label-to-value mapping

- **Rule:** The UI must display role labels in Title Case (`Admin`, `Moderator`, `User`), while the Add/Edit User form submission and the mock API request/response contract use lowercase values (`admin`, `moderator`, `user`), mapped one to one: `Admin` → `admin`, `Moderator` → `moderator`, `User` → `user`.
- **Applies to:** `REQ-008`, `REQ-010`, `REQ-015`
- **Source:** `SRC-001: Section 6, 12`, resolved by human decision on `Q-005` (2026-08-13T13:51:37Z)
- **Verification evidence:** The Role select displays Title Case options; submitted request payloads and rendered table values are consistent with the stated mapping.

## Feature-specific non-functional requirements

Do not duplicate the global `TECH-*` requirements here.

### NFR-001 — No duplicate list requests from re-rendering

- **Statement:** Repeated rendering of the User Management page must not trigger duplicate users-list API requests.
- **Measure:** Exactly one `GET` users-list request is observed per page load or explicit retry.
- **Source:** `SRC-001: Section 7.4`

### NFR-002 — No unintended page-level horizontal scrolling

- **Statement:** At small-screen viewport widths, the page itself must not scroll horizontally; any necessary horizontal scrolling is scoped to the table container.
- **Measure:** At supported small-screen breakpoints, `document`/page-level horizontal overflow is absent; the table container may independently scroll.
- **Source:** `SRC-001: Section 16`

## Dependencies and constraints

| ID | Type | Description | Source | Affected requirements |
| --- | --- | --- | --- | --- |
| `DEP-001` | External dependency | The feature depends on the public DummyJSON Users API (`https://dummyjson.com`) as its mock backend for list, add, update, and delete operations, with the specific endpoints and payload shapes documented in Section 12. | `SRC-001: Section 12` | `REQ-001`, `REQ-008`, `REQ-010`, `REQ-014` |
| `DEP-002` | Constraint | DummyJSON simulates mutations but does not persist them server-side; the feature must not rely on server-side persistence beyond the current session. | `SRC-001: Section 13` | `REQ-008`, `REQ-010`, `REQ-014`, `REQ-017` |
| `DEP-003` | Constraint | The implementation must follow the local SDD workspace's existing technical baseline (React + TypeScript, Vite, React Router lazy routes, MUI-only styling with MUI Table/TablePagination, Formik 2 + Yup, Redux Toolkit + RTK Query, Vitest + React Testing Library, WCAG 2.2 AA) already defined in `TECHNICAL_REQUIREMENTS.md` and `ARCHITECTURE.md`. | `SRC-001: Section 14` | All `REQ-*` |

## Assumptions

| ID | Assumption | Reason it is not confirmed | Affected requirements | Resolution needed |
| --- | --- | --- | --- | --- |
| `ASM-001` | The Document information table's unfilled fields (Product owner, Technical owner, Last updated) do not block intake and are administrative metadata only. | The source page literally contains placeholder text (`<Add name>`, `<Add date>`) rather than filled-in values. | None (process only) | No |
| `ASM-002` | ~~A default MUI theme mode (light or dark) can be selected by the team without changing scope, behavior, or contracts.~~ **Resolved:** Light mode is the confirmed default; the existing theme toggle may remain available. | Previously unconfirmed — see `Q-002` answer. | `REQ-001` | No — resolved by human decision on `Q-002` (2026-08-13T13:51:37Z) |
| `ASM-003` | ~~Newly added users are made visible either by inserting them at a known position in the locally cached collection or by another visibility mechanism.~~ **Resolved:** New users are inserted at the beginning of the collection and the table resets to the first page (see `BR-013`). | Previously unconfirmed — see `Q-003` answer. | `REQ-008` | No — resolved by human decision on `Q-003` (2026-08-13T13:51:37Z) |

## Ambiguities, conflicts, and missing information

| ID | Classification | Finding | Sources | Affected requirements | Impact |
| --- | --- | --- | --- | --- | --- |
| `GAP-001` | Ambiguity — **Resolved** | The source did not specify whether new users are inserted at the beginning or end of the cached collection. **Resolved:** insert at the beginning and reset to the first page (see `BR-013`, `Q-003`). | `SRC-001: Section 8, 19` | `REQ-008` | Add-mutation cache update strategy and pagination navigation logic are now determined for `CONTRACTS.md`. |
| `GAP-002` | Ambiguity — **Resolved** | UI-facing role values are Title Case (`Admin`, `Moderator`, `User`) while Mock API payloads use lowercase values (`admin`, `moderator`, `user`), with no stated mapping. **Resolved:** explicit 1:1 mapping recorded in `BR-014` (see `Q-005`). | `SRC-001: Section 6, 12` | `REQ-008`, `REQ-010`, `REQ-015` | Role field option values and API payload shape are now determined for `CONTRACTS.md`. |
| `GAP-003` | Missing — **Resolved** | The source did not name who provides human approval at each SDD checkpoint. **Resolved:** the requesting human is the approver at every checkpoint for this POC (see `Q-001`). | `SRC-001: Section 19` | None (process only) | Governance/process clarity for the demonstration; does not change application behavior. |
| `GAP-004` | Missing — **Resolved** | The source did not confirm whether it is acceptable, for the live demonstration, that a manual browser refresh resets mock data. **Resolved:** accepted for this POC (see `Q-004`). | `SRC-001: Section 13, 19` | `REQ-017` | Confirms the demo can be presented with this known, accepted limitation. |
| `GAP-005` | Conflict — **Resolved** | Section 13 states the refresh-reset behavior is "an intentional POC constraint, not a defect," while Section 19 separately still listed it as needing confirmation. **Resolved:** Section 13 is authoritative; the Section 19 item is treated as resolved (human decision). | `SRC-001: Section 13, 19` | `REQ-017` | No further action needed; `BR-010` already reflects the Section 13 behavior. |

## Clarification questions

### Q-001 — Who provides human approval at each SDD checkpoint for this feature?

- **Impact:** `Non-blocking`
- **Why it matters:** `ai-workflow/CLAUDE.md` already defines a generic human-approval-gate process; a specific named approver is a process detail that does not change scope, behavior, or contracts.
- **Affected records:** `GAP-003`
- **Owner:** Product owner / Technical owner (both unnamed in the source)
- **Answer:** `RESOLVED (2026-08-13T13:51:37Z)` — "For this POC, I am the human approver at every checkpoint."

### Q-002 — Should the executive demonstration use a light or dark MUI theme by default?

- **Impact:** `Non-blocking`
- **Why it matters:** A sensible default can be chosen and changed later without affecting scope, behavior, or contracts.
- **Affected records:** `ASM-002`, `REQ-001`
- **Owner:** Product owner (unnamed in the source)
- **Answer:** `RESOLVED (2026-08-13T13:51:37Z)` — "Use light mode as the default. The existing theme toggle may remain available."

### Q-003 — Should newly added users appear at the beginning or end of the locally cached collection, and how should the new record be made visible (navigate to its page, or another mechanism)?

- **Impact:** `Blocking`
- **Why it matters:** Changes `REQ-008`'s observable behavior, the add-mutation cache update strategy that `CONTRACTS.md` must define, and the pagination navigation logic implemented for the add flow.
- **Affected records:** `REQ-008`, `GAP-001`, `ASM-003`, `BR-013`
- **Owner:** Product owner (unnamed in the source)
- **Answer:** `RESOLVED (2026-08-13T13:51:37Z)` — "New users must be inserted at the beginning of the locally cached collection. After a user is added successfully, reset the table to the first page so the new record is immediately visible."

### Q-004 — Is it acceptable for the live demonstration that a manual browser refresh resets all added/edited/deleted records back to the original DummyJSON dataset?

- **Impact:** `Non-blocking`
- **Why it matters:** Section 13 already documents this as an intentional POC constraint; confirming it primarily affects how the demo is presented, not required application behavior.
- **Affected records:** `GAP-004`, `GAP-005`, `REQ-017`
- **Owner:** Product owner (unnamed in the source)
- **Answer:** `RESOLVED (2026-08-13T13:51:37Z)` — "Browser refresh resetting mock data is accepted for this POC. Section 13 is authoritative. Treat the Section 19 item as resolved."

### Q-005 — What normalization/mapping rule, if any, should apply between the Title Case role values shown in the UI (Admin/Moderator/User) and the lowercase role values used in the Mock API payloads (admin/moderator/user)?

- **Impact:** `Blocking`
- **Why it matters:** Affects the Role form field's option values, the submitted API request payload shape, and the request/response types `CONTRACTS.md` must define.
- **Affected records:** `REQ-008`, `REQ-010`, `REQ-015`, `GAP-002`, `BR-014`
- **Owner:** Technical owner (unnamed in the source)
- **Answer:** `RESOLVED (2026-08-13T13:51:37Z)` — "UI displays role labels in Title Case (Admin, Moderator, User); the form and API contract use lowercase values (admin, moderator, user). Mapping: Admin → admin, Moderator → moderator, User → user."

## Applicable global technical requirements

| Technical requirement | Relevance to this feature |
| --- | --- |
| `TECH-001` | Feature must be implemented in React + TypeScript. |
| `TECH-002` | Feature must build via Vite with a repeatable production build. |
| `TECH-005`–`TECH-008` | The `/users` route must be lazy-loaded with an accessible Suspense fallback and a recoverable route-loading error state (`REQ-002`, `REQ-005`, `REQ-007`). |
| `TECH-090`–`TECH-094` | UI must use MUI exclusively, including MUI Table/TablePagination and `CssBaseline`, per Section 14. |
| `TECH-100`–`TECH-104` | Add/Edit User forms must use Formik 2 + Yup, with validation errors properly associated and accessible (`REQ-015`, `BR-001`–`BR-006`). |
| `TECH-110`–`TECH-119` | User list/add/update/delete must use RTK Query with typed requests/responses, tag-based cache invalidation or documented manual cache updates (`REQ-001`, `REQ-008`, `REQ-010`, `REQ-014`, `BR-010`). |
| `TECH-120`–`TECH-123` | Orchestration, submission, and workflow logic must live in feature custom hooks, not in page/presentation components. |
| `TECH-010`–`TECH-013` | Feature must pass type checking, linting, tests, and production build before being presented as done (corresponds to source `AC-017`). |
| `TECH-020`–`TECH-024` | Feature must target WCAG 2.2 AA; underlies the feature-specific accessibility detail in `REQ-018`. |
| `TECH-030`–`TECH-031` | Feature must remain usable and free of unintended horizontal scrolling on mobile and desktop (`REQ-019`, `NFR-002`). |
| `TECH-040`–`TECH-043` | Route-level code splitting for `/users` must be preserved in the production build (`REQ-002`). |
| `TECH-060`–`TECH-062` | Loading, empty, success, and failure states must be intentionally designed and must not silently discard user input (`REQ-005`–`REQ-007`, `REQ-016`). |
| `TECH-070`–`TECH-072` | Each accepted requirement must map to automated evidence or documented manual verification. |
| `TECH-080`–`TECH-082` | This document's `REQ-*`/`BR-*`/`NFR-*` identifiers must be referenced by downstream `SPEC.md`, `PLAN.md`, Jira work, tests, and review evidence. |

## Traceability summary

| Local ID | Source | Acceptance or verification evidence | Open questions |
| --- | --- | --- | --- |
| `REQ-001` | `SRC-001: Section 7.1, 7.2, 7.3` | `AC-001` | `None` |
| `REQ-002` | `SRC-001: Section 7.1` | `AC-002` | `None` |
| `REQ-003` | `SRC-001: Section 7.3` | `AC-003` | `None` |
| `REQ-004` | `SRC-001: Section 7.3` | `AC-004` | `None` |
| `REQ-005` | `SRC-001: Section 7.4` | `AC-012` | `None` |
| `REQ-006` | `SRC-001: Section 7.5` | `AC-015` | `None` |
| `REQ-007` | `SRC-001: Section 7.6` | `AC-013` | `None` |
| `REQ-008` | `SRC-001: Section 8` | `AC-005` | `None` (`Q-003` resolved) |
| `REQ-009` | `SRC-001: Section 8` | Manual/automated test of Cancel action | `None` |
| `REQ-010` | `SRC-001: Section 9` | `AC-007` | `None` (`Q-005` resolved) |
| `REQ-011` | `SRC-001: Section 9` | `AC-008` | `None` |
| `REQ-012` | `SRC-001: Section 10` | `AC-009` | `None` |
| `REQ-013` | `SRC-001: Section 10` | Manual/automated test of Cancel action | `None` |
| `REQ-014` | `SRC-001: Section 10` | `AC-010`, `AC-011` | `None` |
| `REQ-015` | `SRC-001: Section 11` | `AC-006` | `None` (`Q-005` resolved) |
| `REQ-016` | `SRC-001: Section 7.6` | `AC-014` | `None` |
| `REQ-017` | `SRC-001: Section 13` | `AC-016` | `None` (`Q-004` resolved) |
| `REQ-018` | `SRC-001: Section 15` | Manual/automated accessibility verification | `None` |
| `REQ-019` | `SRC-001: Section 16` | Manual verification at supported viewports | `None` |
| `BR-001`–`BR-006` | `SRC-001: Section 11` | Field-level validation test coverage | `None` (`Q-005` resolved for Role, via `BR-005`) |
| `BR-007`, `BR-008` | `SRC-001: Section 7.3` | Pagination control test coverage | `None` |
| `BR-009` | `SRC-001: Section 7.3, 10` | Deletion pagination-recalculation test coverage | `None` |
| `BR-010` | `SRC-001: Section 13` | Session-persistence test coverage | `None` (`Q-004` resolved) |
| `BR-011` | `SRC-001: Section 10` | Delete confirmation dialog test coverage | `None` |
| `BR-012` | `SRC-001: Section 15` | Accessible-name assertion test coverage | `None` |
| `BR-013` | `SRC-001: Section 8` | New-record insertion/page-reset test coverage | `None` (`Q-003` resolved) |
| `BR-014` | `SRC-001: Section 6, 12` | Role label/value mapping test coverage | `None` (`Q-005` resolved) |
| `NFR-001` | `SRC-001: Section 7.4` | Network-call assertion in tests | `None` |
| `NFR-002` | `SRC-001: Section 16` | Manual/automated viewport overflow check | `None` |

## Human intake approval

- **Decision:** `APPROVED`
- **Reviewer:** Babak Khajavi
- **Date:** 2026-08-13
- **Comments:** Approved for the POC specification stage. The resolved clarification decisions are accepted.

Approval authorizes the Spec Agent to begin. It does not authorize Jira creation
or application implementation.
