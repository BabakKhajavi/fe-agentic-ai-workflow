# Implementation Plan: User Management Dashboard

## Document control

| Field | Value |
| --- | --- |
| Feature slug | `user-management` |
| Status | `READY_FOR_APPROVAL` |
| Based on | `REQUIREMENTS.md`, `SPEC.md`, `CONTRACTS.md`, `IMPACT.md` (all `APPROVED`) |
| Human plan approval | `APPROVED` |

## Implementation strategy

Add one self-contained feature module, `src/features/user-management/`,
reusing the application shell, theme, store, and shared RTK Query `api` slice
exactly as the bootstrap baseline intended. Build bottom-up: domain types and
the mock API layer first, then the presentation pieces that depend on them
(form, delete dialog, table), then the orchestration hook and page that
compose everything, then route registration, then a final full-suite
verification pass. Each task is independently reviewable and testable, and
every mutation's cache behavior is implemented alongside the endpoint that
needs it rather than deferred.

## Ordered tasks

### TASK-001 — Define user-management domain types and role mapping

- **Outcome:** `RoleLabel`, `RoleValue`, `ROLE_LABEL_TO_VALUE`, `UserFormValues`, `User`, `GetUsersResponse`, and `UpsertUserRequest` exist exactly as specified in `CONTRACTS.md`'s Data shapes section, ready for the API layer and forms to import.
- **Depends on:** `None`
- **Expected files:**
  - `Create: src/features/user-management/types.ts`
- **Implementation notes:** No React or Redux code here — pure type/constant definitions. `ROLE_LABEL_TO_VALUE` is the single source of truth for the `BR-014` mapping; do not duplicate the mapping elsewhere.
- **Requirements:** `REQ-008`, `REQ-010`, `REQ-015`, `BR-005`, `BR-014`
- **Acceptance criteria:** Supports `AC-008`, `AC-010`, `AC-011` (contributing types only; verified through the tasks that consume them)
- **Contracts:** `CONTRACTS.md` Data shapes
- **Verification:** `npm run typecheck` passes; no standalone test file (exercised via `TASK-002`–`TASK-006` tests).
- **Suggested Jira type:** `Task`

### TASK-002 — Add RTK Query endpoints for the Users mock API

- **Outcome:** `getUsers`, `addUser`, `updateUser`, `deleteUser` are injected into the shared `api` slice and behave exactly as `CON-020`–`CON-023` specify, including the manual cache updates (insert-at-beginning for add, in-place replace for update, remove for delete).
- **Depends on:** `TASK-001`
- **Expected files:**
  - `Modify: src/services/api.ts` (add `'User'` to `tagTypes`)
  - `Create: src/features/user-management/services/usersApi.ts`
  - `Create: src/features/user-management/__tests__/usersApi.test.ts`
- **Implementation notes:** Each endpoint's `query` returns a full absolute DummyJSON URL (see `IMPACT.md` *Change summary*); do not change the shared slice's `baseUrl`. `getUsers` sets `providesTags: [{ type: 'User', id: 'LIST' }]`; mutations set `invalidatesTags: []` and instead call `dispatch(api.util.updateQueryData('getUsers', undefined, draft => { ... }))` inside `onQueryStarted`, guarded by the mutation's own success (roll back on failure via the returned patch). Delete must also account for pagination recalculation being the *caller's* responsibility (`TASK-006`), not baked into the endpoint.
- **Requirements:** `REQ-001`, `REQ-003`–`REQ-014`, `REQ-016`, `REQ-017`, `BR-009`, `BR-010`, `BR-013`, `BR-014`
- **Acceptance criteria:** `AC-001`, `AC-003`, `AC-004`, `AC-005`, `AC-006`, `AC-007`, `AC-008`, `AC-009`, `AC-011`, `AC-012`, `AC-013`, `AC-014`, `AC-015`, `AC-016`, `AC-017`
- **Contracts:** `CON-020`, `CON-021`, `CON-022`, `CON-023`
- **Verification:** `usersApi.test.ts` mocks `global.fetch` per endpoint (success and failure) and asserts: correct URL/method/body; cache contains the inserted/updated/removed record in the right position; a failed mutation does not mutate the cache.
- **Suggested Jira type:** `Task`

### TASK-003 — Build the Add/Edit User form and submission hook

- **Outcome:** `useUserForm` and `UserFormDialog` implement `CON-010`: Formik 2 + Yup validation per `BR-001`–`BR-006`, role label→value mapping on submit, `Save changes` label when editing, submission blocked while invalid or in flight, errors preserve entered values and are retryable.
- **Depends on:** `TASK-001`, `TASK-002`
- **Expected files:**
  - `Create: src/features/user-management/hooks/useUserForm.ts`
  - `Create: src/features/user-management/components/UserFormDialog.tsx`
  - `Create: src/features/user-management/__tests__/UserFormDialog.test.tsx`
- **Implementation notes:** `useUserForm` owns the Formik config, Yup schema, and mutation dispatch (`addUser` when no target user, `updateUser` when editing); the dialog component only renders the form contract the hook exposes (`TECH-120`–`TECH-122`). Field errors use MUI `error`/`helperText` for `aria-describedby` association (`REQ-018`). Dialog focus management relies on MUI `Dialog`'s built-in trap/return-focus behavior.
- **Requirements:** `REQ-008`, `REQ-009`, `REQ-010`, `REQ-011`, `REQ-015`, `REQ-016`, `REQ-018`, `BR-001`–`BR-006`, `BR-014`
- **Acceptance criteria:** `AC-008`, `AC-009`, `AC-010`, `AC-011`, `AC-012`, `AC-016`, `AC-018` (form-scoped part)
- **Contracts:** `CON-010`, `CON-031`
- **Verification:** `UserFormDialog.test.tsx` — required/format/length errors per field after touch and on submit attempt; submit disabled while invalid/in-flight; valid Add and valid Edit each call the correct mutation with mapped `role`; a mocked mutation failure keeps the dialog open with entered values and shows a retryable message; `Cancel` calls neither mutation.
- **Suggested Jira type:** `Task`

### TASK-004 — Build the delete confirmation dialog

- **Outcome:** `DeleteUserDialog` implements `BR-011`: names the target user, requires explicit confirmation before calling `deleteUser`, and surfaces a retryable error on failure without closing prematurely.
- **Depends on:** `TASK-002`
- **Expected files:**
  - `Create: src/features/user-management/components/DeleteUserDialog.tsx`
  - `Create: src/features/user-management/__tests__/DeleteUserDialog.test.tsx`
- **Implementation notes:** Dialog text is `Delete <first name> <last name>?` plus the session-scope explanatory sentence from `REQUIREMENTS.md` Section 10. `Cancel` closes without any API call. On confirm, call `deleteUser`; on success close and let the caller (`TASK-006`) handle pagination recalculation and the success notification; on failure, keep the dialog open with a retryable error (`REQ-016`).
- **Requirements:** `REQ-012`, `REQ-013`, `REQ-014`, `REQ-016`, `REQ-018`, `BR-009`, `BR-011`
- **Acceptance criteria:** `AC-013`, `AC-014`, `AC-016`, `AC-018` (dialog-scoped part)
- **Contracts:** `CON-023`
- **Verification:** `DeleteUserDialog.test.tsx` — dialog names the correct user; `Cancel` triggers no mutation call and no deletion; `Delete` calls `deleteUser` and closes on success; a mocked failure keeps the dialog open with a retryable message.
- **Suggested Jira type:** `Task`

### TASK-005 — Build the Users table and pagination

- **Outcome:** `UserTable` renders Name/Email/Phone/Role/Actions via MUI `Table` + `TablePagination`, performs client-side-only pagination (`BR-007`, `BR-008`), shows the empty state (`REQ-006`), and meets the accessibility/responsive requirements for the table itself (`REQ-018`, `REQ-019`, `BR-012`).
- **Depends on:** `TASK-001`
- **Expected files:**
  - `Create: src/features/user-management/components/UserTable.tsx`
  - `Create: src/features/user-management/__tests__/UserTable.test.tsx`
- **Implementation notes:** Pagination (current page, page size, slicing) is owned by the caller (`TASK-006`); `UserTable` is a presentation component receiving the already-paginated slice, total count, and handlers as props — it does not call RTK Query directly (`TECH-120`). Row actions use `aria-label` of `Edit ${firstName} ${lastName}` / `Delete ${firstName} ${lastName}`. The table container, not the page, owns any horizontal scroll (`NFR-002`).
- **Requirements:** `REQ-001`, `REQ-003`, `REQ-004`, `REQ-006`, `REQ-018`, `REQ-019`, `BR-007`, `BR-008`, `BR-012`
- **Acceptance criteria:** `AC-001`, `AC-003`, `AC-004`, `AC-006`, `AC-018` (table-scoped part), `AC-019` (table-scoped part)
- **Contracts:** `CON-001` (rendering only), consumes `CON-020`
- **Verification:** `UserTable.test.tsx` — renders the five columns; changing page/page-size calls the provided handlers with no network access (table has no RTK Query dependency to assert against, confirming isolation); page-size change handler contract matches `BR-008`'s reset-to-page-1 expectation (verified at the hook level in `TASK-006`); empty collection renders `No users found` with no row markup; row actions expose the expected accessible names.
- **Suggested Jira type:** `Task`

### TASK-006 — Build the list orchestration hook and page composition

- **Outcome:** `useUserListPage` and `UserManagementPage` tie `getUsers`, pagination state, dialog/edit-target state, and notifications together, implementing the loading/empty/error states, the session-only notice, and the add/delete pagination side effects (`BR-013`, `BR-009`).
- **Depends on:** `TASK-002`, `TASK-003`, `TASK-004`, `TASK-005`
- **Expected files:**
  - `Create: src/features/user-management/hooks/useUserListPage.ts`
  - `Create: src/features/user-management/components/UserManagementPage.tsx`
  - `Create: src/features/user-management/index.ts`
  - `Create: src/features/user-management/__tests__/UserManagementPage.test.tsx`
- **Implementation notes:** `useUserListPage` calls `getUsers`, derives the current page's slice from `page`/`pageSize` state, resets `page` to `1` on page-size change (`BR-008`) and after a successful add (`BR-013`), and moves `page` to the nearest valid page after a delete empties the current page (`BR-009`). `UserManagementPage` renders the header, the session-only notice (`Demo mode: Changes are retained for this browser session only.`), and switches between loading/empty/error/success per `getUsers`' query state, delegating to `UserTable`, `UserFormDialog`, and `DeleteUserDialog`. `index.ts` exports only `UserManagementPage` for the route table to import.
- **Requirements:** `REQ-001`, `REQ-002`, `REQ-005`, `REQ-006`, `REQ-007`, `REQ-014`, `REQ-017`, `REQ-019`
- **Acceptance criteria:** `AC-001`, `AC-002`, `AC-005`, `AC-006`, `AC-007`, `AC-008` (page-reset part), `AC-015`, `AC-017`, `AC-019` (page-scoped part)
- **Contracts:** `CON-001`, `CON-030`
- **Verification:** `UserManagementPage.test.tsx` — pending query shows the loading indicator without an empty-table message; empty collection shows `No users found`; failed initial load shows the retry banner and `Try again` re-triggers the query; the session-only notice text is present; one end-to-end path exercises add → new row first on page 1, edit → updated row visible, delete → row removed and pagination recalculated.
- **Suggested Jira type:** `Task`

### TASK-007 — Register the `/users` lazy route

- **Outcome:** `/users` is reachable, lazily loads `UserManagementPage` as its own bundle, and is covered by the application shell's existing Suspense fallback and route error boundary with no shell change.
- **Depends on:** `TASK-006`
- **Expected files:**
  - `Modify: src/app/routes.ts`
- **Implementation notes:** Add `{ path: '/users', Component: lazy(() => import('../features/user-management')) }` following the existing `HomePage` entry's pattern exactly. Do not touch `src/app/App.tsx` — it already renders whatever `routes` contains.
- **Requirements:** `REQ-002`
- **Acceptance criteria:** `AC-002`
- **Contracts:** `CON-001`
- **Verification:** `npm run build` output lists a dedicated chunk for the feature (matching the existing `HomePage-*.js` pattern); manual navigation to `/users` in `npm run dev` confirms the route renders.
- **Suggested Jira type:** `Task`

### TASK-008 — Final integration verification

- **Outcome:** The complete feature passes every required automated gate and is demonstrable end-to-end against the live DummyJSON API.
- **Depends on:** `TASK-001`–`TASK-007`
- **Expected files:** `None` (verification only)
- **Implementation notes:** Run the checks below in order; fix forward rather than skip any of them.
- **Requirements:** `TECH-010`–`TECH-013`, all `REQ-001`–`REQ-019`
- **Acceptance criteria:** All `AC-001`–`AC-019`
- **Contracts:** All `CON-001`, `CON-010`, `CON-020`–`CON-023`, `CON-030`, `CON-031`
- **Verification:** `npm run typecheck`, `npm run lint`, `npm run test`, `npm run build` all pass; manual POC walkthrough in `npm run dev` (or `npm run preview` against the build) covering view/paginate, add, edit, delete, an intentionally invalid form submission, and a simulated network failure (e.g., offline dev tools) to confirm the error/retry states — recorded as `Manual POC demonstration` for `AC-018`/`AC-019` per `SPEC.md`.
- **Suggested Jira type:** `Task`

## Final integration verification

1. Type checking — `npm run typecheck` (`tsc -b`).
2. Linting — `npm run lint` (`eslint .`).
3. Automated tests — `npm run test` (`vitest run`), including `TASK-002`–`TASK-006`'s new test files.
4. Vite production build — `npm run build`, confirming `/users` code-splits into its own chunk.
5. POC user-flow demonstration — manual walkthrough per `TASK-008`.

`AC-018` (accessibility) and `AC-019` (responsive layout) are each partly
automated (accessible-name/role assertions in the component tests) and partly
`Manual POC demonstration`, because this POC's test stack has no
visual-regression or real-viewport testing tool, and adding one is out of
scope for this feature (see `IMPACT.md` `RISK-004` for the related test-tooling
shortcut).

## Traceability matrix

| Requirement | Acceptance criteria | Contracts | Impact | Tasks | Verification |
| --- | --- | --- | --- | --- | --- |
| `REQ-001` | `AC-001` | `CON-001`, `CON-020` | `IMP-004`, `IMP-009`, `IMP-010` | `TASK-002`, `TASK-005`, `TASK-006` | Automated — `usersApi.test.ts`, `UserTable.test.tsx`, `UserManagementPage.test.tsx` |
| `REQ-002` | `AC-002` | `CON-001` | `IMP-002`, `IMP-010`, `IMP-011` | `TASK-006`, `TASK-007` | Automated build-chunk check + manual navigation |
| `REQ-003` | `AC-003` | `CON-020` | `IMP-004`, `IMP-009` | `TASK-002`, `TASK-005` | Automated — `usersApi.test.ts`, `UserTable.test.tsx` |
| `REQ-004` | `AC-004` | `CON-020` | `IMP-004`, `IMP-009` | `TASK-002`, `TASK-005` | Automated — `usersApi.test.ts`, `UserTable.test.tsx` |
| `REQ-005` | `AC-005` | `CON-020` | `IMP-004`, `IMP-010` | `TASK-002`, `TASK-006` | Automated — `UserManagementPage.test.tsx` |
| `REQ-006` | `AC-006` | `CON-020` | `IMP-004`, `IMP-009`, `IMP-010` | `TASK-002`, `TASK-005`, `TASK-006` | Automated — `UserTable.test.tsx`, `UserManagementPage.test.tsx` |
| `REQ-007` | `AC-007` | `CON-020` | `IMP-004`, `IMP-010` | `TASK-002`, `TASK-006` | Automated — `UserManagementPage.test.tsx` |
| `REQ-008` | `AC-008` | `CON-010`, `CON-021` | `IMP-003`, `IMP-004`, `IMP-005`, `IMP-007`, `IMP-010` | `TASK-001`, `TASK-002`, `TASK-003`, `TASK-006` | Automated — `usersApi.test.ts`, `UserFormDialog.test.tsx`, `UserManagementPage.test.tsx` |
| `REQ-009` | `AC-009` | `CON-010` | `IMP-005`, `IMP-007` | `TASK-003` | Automated — `UserFormDialog.test.tsx` |
| `REQ-010` | `AC-011` | `CON-010`, `CON-022` | `IMP-003`, `IMP-004`, `IMP-005`, `IMP-007` | `TASK-001`, `TASK-002`, `TASK-003` | Automated — `usersApi.test.ts`, `UserFormDialog.test.tsx` |
| `REQ-011` | `AC-012` | `CON-010` | `IMP-005`, `IMP-007` | `TASK-003` | Automated — `UserFormDialog.test.tsx` |
| `REQ-012` | `AC-013` | `CON-023` | `IMP-004`, `IMP-008` | `TASK-002`, `TASK-004` | Automated — `DeleteUserDialog.test.tsx` |
| `REQ-013` | `AC-014` | `CON-023` | `IMP-008` | `TASK-004` | Automated — `DeleteUserDialog.test.tsx` |
| `REQ-014` | `AC-015` | `CON-023` | `IMP-004`, `IMP-008`, `IMP-010` | `TASK-002`, `TASK-004`, `TASK-006` | Automated — `usersApi.test.ts`, `DeleteUserDialog.test.tsx`, `UserManagementPage.test.tsx` |
| `REQ-015` | `AC-010` | `CON-010` | `IMP-003`, `IMP-005`, `IMP-007` | `TASK-001`, `TASK-003` | Automated — `UserFormDialog.test.tsx` |
| `REQ-016` | `AC-016` | `CON-021`, `CON-022`, `CON-023` | `IMP-004`, `IMP-005`, `IMP-007`, `IMP-008` | `TASK-002`, `TASK-003`, `TASK-004` | Automated — `usersApi.test.ts`, `UserFormDialog.test.tsx`, `DeleteUserDialog.test.tsx` |
| `REQ-017` | `AC-017` | `CON-020`–`CON-023` | `IMP-004`, `IMP-010` | `TASK-002`, `TASK-006` | Automated — `usersApi.test.ts`, `UserManagementPage.test.tsx` |
| `REQ-018` | `AC-018` | `CON-001`, `CON-010`, `CON-023` | `IMP-007`, `IMP-008`, `IMP-009` | `TASK-003`, `TASK-004`, `TASK-005` | Automated accessible-name/role assertions + `Manual POC demonstration` |
| `REQ-019` | `AC-019` | `CON-001` | `IMP-009`, `IMP-010` | `TASK-005`, `TASK-006` | Automated (jsdom-level where meaningful) + `Manual POC demonstration` |

## Recommended Jira breakdown

- **Epic:** User Management Dashboard — paginated user directory with add/edit/delete against the DummyJSON mock API.
- **Child work:** One Jira issue per `TASK-001`–`TASK-008`. None of the eight tasks need to be combined — each has an independent, reviewable outcome, a bounded file set, and its own verification. All eight default to issue type `Task`, per `JIRA.md`'s observed project configuration (only `Task` and `Bug` issue types are confirmed available in `SDM`/`Web`, and none of these tasks describes a defect correction). The Jira Agent must still inspect current project metadata before creation in case configuration has changed.

Jira creation remains prohibited until this plan and `IMPACT.md` are approved.

## Human plan approval

- **Decision:** `APPROVED`
- **Reviewer:** Babak Khajavi
- **Date:** 2026-08-13
- **Comments:** Approved for Jira creation and POC implementation. Manual RTK Query cache updates are approved for DummyJSON session-only mutations, while query tag definitions must remain in place for traceability and future invalidation.

Approval authorizes Jira creation and implementation according to this plan. It
does not authorize unplanned scope or production deployment.
