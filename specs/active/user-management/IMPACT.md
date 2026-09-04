# Impact Assessment: User Management Dashboard

## Document control

| Field | Value |
| --- | --- |
| Feature slug | `user-management` |
| Status | `READY_FOR_APPROVAL` |
| Repository inspected | `sample-react-app` |
| Revision or state | No git repository is initialized in this workspace; working tree as produced by the `/poc-bootstrap-app` session on 2026-08-13, unmodified since (verified by re-inspecting `package.json`, `src/`, and installed `node_modules` versions during this planning pass). |
| Human plan approval | `APPROVED` |

## Repository baseline

| Concern | Observed implementation | Version or evidence | Alignment |
| --- | --- | --- | --- |
| React and TypeScript | React with TypeScript, Vite-bundled | `react@19.2.8`, `typescript@6.0.3` (`package.json`) | Aligned |
| Vite | Dev/build tooling | `vite@8.2.1` (`node_modules/vite/package.json`); `npm run build` produces route-level chunks | Aligned |
| Routing and lazy loading | `src/app/routes.ts` defines a `RouteObject[]` consumed by `src/app/App.tsx`'s `<Routes>`/`<Route Component={...}>`; one lazy route (`/`, `HomePage`) exists via `React.lazy` | `react-router-dom@7.18.2`; `src/app/routes.ts`, `src/app/App.tsx` | Aligned — adding `/users` only requires appending an entry, no shell change |
| MUI theme and styling | `src/styles/theme.ts` exports `createAppTheme(mode)`; `App.tsx` wraps the tree in `ThemeProvider` + `CssBaseline`; theme mode is Redux-backed (`themeModeSlice`, defaults to `light`) | `@mui/material@9.3.1`, `@emotion/react@11.14.0`, `@emotion/styled@11.14.1` | Aligned — light default already matches `Q-002`'s resolution |
| Formik 2 and Yup | Not yet used by any feature; dependencies installed at bootstrap | `formik@2.4.9`, `yup@1.7.1` (`package.json`) | Aligned/Not used yet — available for this feature |
| Redux Toolkit and RTK Query | `src/app/store.ts` configures one store with `themeModeReducer` and the shared `api.reducer`; `src/services/api.ts` defines an empty `createApi` slice (`reducerPath: 'api'`, `tagTypes: []`, `endpoints: () => ({})`) intended for feature endpoints to be added via `api.injectEndpoints` | `@reduxjs/toolkit@2.12.0`, `react-redux@9.3.0` | Aligned — store already wires the shared `api.reducer` and its middleware, so adding endpoints requires no `store.ts` change |
| Tests and quality scripts | `npm run typecheck` (`tsc -b`), `npm run lint` (`eslint .`), `npm run test` (`vitest run`), `npm run build` (`tsc -b && vite build`) all pass on the current baseline; Vitest configured with `jsdom` + `src/test/setup.ts` (`@testing-library/jest-dom`, RTL `cleanup` in `afterEach`) | `vitest@4.1.10`, `@testing-library/react@16.3.2`, `eslint@10.8.1` (`package.json`, `vite.config.ts`) | Aligned |

No mandatory `TECH-*` gap was found in the current repository.

## Change summary

The feature adds one new module, `src/features/user-management/`, following the
established feature-boundary pattern from `ARCHITECTURE.md`
(`components/`, `hooks/`, `services/`, `types.ts`, `index.ts`, `__tests__/`).
It injects four endpoints into the already-configured shared `api` slice, adds
one lazy route entry to the existing route table, and needs one one-line
addition to the shared `api` slice's `tagTypes`. No existing component, the
Redux store, the app shell, or the MUI theme requires modification — the
bootstrap baseline already anticipated exactly this shape of feature addition.
Pagination, dialog-open, and edit-target state are kept in feature-local hook
state (not Redux), per `CONTRACTS.md`'s "Not needed" global client-state
contract; the user collection itself lives only in the RTK Query cache.

Because the shared `api` slice's `fetchBaseQuery` uses a relative `baseUrl: '/'`
(intentionally generic, not coupled to any one backend), this feature's
endpoints supply the full absolute DummyJSON URL (e.g.,
`https://dummyjson.com/users`) in each endpoint's `query` function. RTK
Query's `fetchBaseQuery` treats an absolute URL returned by `query` as
self-contained and does not prefix it with `baseUrl`, so this works without
coupling the shared base query to one external mock service — a decision
worth recording here because it affects how any future feature's endpoints are
written against the same shared slice.

## Affected areas

| ID | Action | Existing/proposed path | Change | Traceability |
| --- | --- | --- | --- | --- |
| `IMP-001` | `Modify` | `src/services/api.ts` | Add `'User'` to `tagTypes` (currently `[]`) so injected endpoints can type `providesTags`/`invalidatesTags`. No other change; `baseQuery`, `reducerPath`, and `endpoints: () => ({})` stay as-is. | `CON-020`, `TECH-115`, `TECH-116` |
| `IMP-002` | `Modify` | `src/app/routes.ts` | Append one `RouteObject` for `path: '/users'` with `Component` set to the lazily-imported `UserManagementPage`. No change to `App.tsx`, which already renders whatever is in the `routes` array. | `REQ-002`, `AC-002`, `CON-001` |
| `IMP-003` | `Create` | `src/features/user-management/types.ts` | `RoleLabel`, `RoleValue`, `ROLE_LABEL_TO_VALUE`, `UserFormValues`, `User`, `GetUsersResponse`, `UpsertUserRequest` per `CONTRACTS.md` Data shapes. | `REQ-008`, `REQ-010`, `REQ-015`, `BR-005`, `BR-014` |
| `IMP-004` | `Create` | `src/features/user-management/services/usersApi.ts` | `api.injectEndpoints` defining `getUsers`, `addUser`, `updateUser`, `deleteUser` against the DummyJSON Users API, including the documented manual cache updates (insert-at-beginning for add, in-place replace for update, remove for delete). | `REQ-001`, `REQ-003`–`REQ-014`, `REQ-016`, `REQ-017`; `BR-009`, `BR-010`, `BR-013`, `BR-014`; `CON-020`–`CON-023` |
| `IMP-005` | `Create` | `src/features/user-management/hooks/useUserForm.ts` | Formik/Yup wiring for the Add/Edit form: schema per `BR-001`–`BR-006`, role label→value mapping (`BR-014`) on submit, dispatch of `addUser`/`updateUser`, submission-in-flight guard (`BR-006`). | `REQ-008`–`REQ-011`, `REQ-015`, `REQ-016`; `CON-010`, `CON-031` |
| `IMP-006` | `Create` | `src/features/user-management/hooks/useUserListPage.ts` | Orchestrates `getUsers`, pagination (page/page-size, reset-on-page-size-change per `BR-008`, reset-on-add per `BR-013`, recalculation-on-delete per `BR-009`), dialog open/edit-target state, and success/error notification state. | `REQ-001`, `REQ-003`–`REQ-007`, `REQ-014`, `REQ-017`; `CON-020`, `CON-030` |
| `IMP-007` | `Create` | `src/features/user-management/components/UserFormDialog.tsx` | Accessible MUI `Dialog` rendering the Add/Edit form contract (`CON-010`); labeled fields with associated errors; primary action label switches to `Save changes` when editing; disabled while invalid or submitting. | `REQ-008`–`REQ-011`, `REQ-015`, `REQ-018`; `BR-001`–`BR-006` |
| `IMP-008` | `Create` | `src/features/user-management/components/DeleteUserDialog.tsx` | Accessible MUI `Dialog` naming the target user (`BR-011`), `Cancel`/`Delete` actions, focus returned to the triggering row action on close. | `REQ-012`–`REQ-014`, `REQ-018`; `BR-009`, `BR-011` |
| `IMP-009` | `Create` | `src/features/user-management/components/UserTable.tsx` | MUI `Table` + `TablePagination` rendering Name/Email/Phone/Role/Actions; accessible `Edit <name>`/`Delete <name>` action names (`BR-012`); empty-state row replaced by the `No users found` message (`REQ-006`); desktop shows all columns, small screens scope horizontal scroll to the table container only (`REQ-019`, `NFR-002`). | `REQ-001`, `REQ-003`, `REQ-004`, `REQ-006`, `REQ-018`, `REQ-019`; `BR-007`, `BR-008`, `BR-012` |
| `IMP-010` | `Create` | `src/features/user-management/components/UserManagementPage.tsx` | Page composition: header (`User Management` / `View and manage users` / `Add user`), session-only notice (`REQ-017`), loading/empty/error states (`REQ-005`–`REQ-007`), and the table/dialogs above. This is the component the lazy route imports. | `REQ-001`, `REQ-002`, `REQ-005`–`REQ-007`, `REQ-017`, `REQ-019` |
| `IMP-011` | `Create` | `src/features/user-management/index.ts` | Feature's public export (the page component) consumed by the route table, per `ARCHITECTURE.md`'s cross-feature-import boundary. | `REQ-002` |
| `IMP-012` | `Create` | `src/features/user-management/__tests__/usersApi.test.ts` | RTK Query endpoint tests: request URLs/params, response typing, and the manual cache-update behavior for add/update/delete against a mocked `fetch`. | `REQ-001`, `REQ-003`, `REQ-008`, `REQ-010`, `REQ-014`, `REQ-016`; `BR-009`, `BR-013`, `BR-014` |
| `IMP-013` | `Create` | `src/features/user-management/__tests__/UserFormDialog.test.tsx` | Field validation messages, submit-blocked-while-invalid, valid Add/Edit submission (mocked mutation success and failure), Cancel behavior. | `REQ-008`, `REQ-009`, `REQ-011`, `REQ-015`, `REQ-016`; `BR-001`–`BR-006` |
| `IMP-014` | `Create` | `src/features/user-management/__tests__/DeleteUserDialog.test.tsx` | Confirmation naming, Cancel makes no call, Confirm calls delete and surfaces success/error. | `REQ-012`–`REQ-014`, `REQ-016`; `BR-011` |
| `IMP-015` | `Create` | `src/features/user-management/__tests__/UserTable.test.tsx` | Column rendering, client-side pagination/page-size behavior, empty state, accessible row-action names. | `REQ-001`, `REQ-003`, `REQ-004`, `REQ-006`, `REQ-018`; `BR-007`, `BR-008`, `BR-012` |
| `IMP-016` | `Create` | `src/features/user-management/__tests__/UserManagementPage.test.tsx` | Integration test: loading → success render, empty state, initial error + retry, session-only notice text, and one end-to-end add/edit/delete happy path through the composed page. | `REQ-001`, `REQ-002`, `REQ-005`–`REQ-007`, `REQ-008`, `REQ-010`, `REQ-014`, `REQ-017` |

## State and data impact

### Form state

Formik 2 owns `UserFormValues` (`firstName`, `lastName`, `email`, `phone`,
`role`); Yup validates per `BR-001`–`BR-006`. `useUserForm` (`IMP-005`) is the
single owner of submission logic, including the `RoleLabel` → `RoleValue`
mapping (`BR-014`); no Formik value or error is duplicated in component state
(`TECH-102`).

### Global client state

Not needed. Pagination selection, which dialog is open, which user is being
edited/deleted, and transient success/error notification content are
feature-local hook state (`useUserListPage`), consistent with
`CONTRACTS.md`'s global client-state contract and `ARCHITECTURE.md`'s
narrowest-state-mechanism guidance. No new Redux slice is created, and
`src/app/store.ts` requires no change.

### Server state and API

RTK Query, `MOCK` mode, against the public DummyJSON Users API
(`https://dummyjson.com`), per `CON-020`–`CON-023`:

- `getUsers` — `providesTags: [{ type: 'User', id: 'LIST' }]`.
- `addUser`, `updateUser`, `deleteUser` — `invalidatesTags: []`; each performs
  a targeted `updateQueryData` cache mutation on the `getUsers` cache entry
  instead of invalidating and refetching (documented cache-update strategy per
  `TECH-116`), because refetching would both violate `REQ-003` (no second list
  request on ordinary pagination) and lose control over insertion position
  (`BR-013`).
- No data is duplicated into a Redux slice or component state (`TECH-114`).

## Quality impact

- **Accessibility:** Semantic `<table>` markup (`UserTable`); `Edit <name>` /
  `Delete <name>` accessible names (`BR-012`); MUI `Dialog` provides focus
  trapping and Escape-to-close, with focus explicitly returned to the
  triggering row action on close; Formik/Yup errors wired to MUI `TextField`/
  `Select` via `error` + `helperText` (`aria-describedby`); loading and
  success/error messaging exposed via `role="status"`/`role="alert"`
  (reusing the existing `LoadingFallback` pattern for the loading state).
  Checked via RTL accessible-name/role queries plus one manual POC pass at
  supported viewports (`REQ-018`, `TECH-020`–`TECH-024`).
- **Security/privacy:** No new secrets or credentials; DummyJSON is a public
  API called from the client, consistent with the existing bootstrap baseline
  (`TECH-050`–`TECH-053`). No sensitive data is logged.
- **Performance/bundle:** `/users` becomes its own lazy route-level chunk
  (verified by inspecting the `npm run build` output, matching the existing
  `HomePage` chunk pattern from bootstrap). The full user collection is
  fetched once and paginated client-side, avoiding repeated network calls
  (`REQ-003`, `NFR-001`). No large new dependency is introduced.
- **Testing:** RTL/Vitest component and integration tests per `IMP-012`–`IMP-016`
  cover all `REQ-*` marked `Automated` in `SPEC.md`. `AC-018` and `AC-019` are
  partly automated (accessible-name/role assertions, and jsdom-level layout
  assertions where meaningful) and partly `Manual POC demonstration` — real
  visual/responsive verification across viewport sizes is not meaningfully
  automatable within this POC's test stack (no visual-regression tooling is
  installed, and adding one is out of scope for this feature).

## Dependencies

| Dependency | Existing/new | Purpose | Required by | Risk |
| --- | --- | --- | --- | --- |
| `None` | — | The feature is fully covered by dependencies already installed during `/poc-bootstrap-app` (`react-router-dom`, `@mui/material`, `formik`, `yup`, `@reduxjs/toolkit`, `react-redux`, `vitest`, `@testing-library/react`). | — | — |

## POC shortcuts and production gaps

| ID | Shortcut or gap | POC rationale | Demo impact | Production follow-up |
| --- | --- | --- | --- | --- |
| `RISK-001` | The public DummyJSON Users API is the entire backend. | Explicitly mandated by the approved source and `SPEC.md` `POC-001`. | List/add/edit/delete calls succeed and look real but nothing is stored on a real server. | Replace with a real backend service before any production use. |
| `RISK-002` | Add/edit/delete are reflected only in the client-side RTK Query cache for the session; a browser refresh may restore DummyJSON's original dataset. | DummyJSON does not persist mutations server-side (`BR-010`); accepted for this POC (`Q-004`, `SPEC.md` `POC-002`). | The live demo must avoid refreshing mid-demonstration. | Implement real persistence via a production backend. |
| `RISK-003` | No authentication or authorization. | Explicitly out of scope. | Acceptable for an internal/executive demo only. | Add authentication/authorization before wider release. |
| `RISK-004` | Endpoint tests mock the global `fetch` directly rather than using a network-interception library (e.g., MSW). | No such dependency exists in the bootstrap baseline; adding one is not justified for four endpoints in one POC feature. | None for this feature's own test suite. | If the application grows to many API-backed features, adopt MSW (or equivalent) workspace-wide for consistency and to reduce per-test mocking boilerplate. |
| `RISK-005` | Add/update/delete use manual RTK Query cache updates (`updateQueryData`) instead of tag invalidation and refetch. | Refetching after every mutation would violate `REQ-003` (no extra list request) and cannot express the required insertion position for `BR-013`; DummyJSON's lack of server-side persistence also makes refetch-based consistency meaningless here. | None — this is the approved `CONTRACTS.md` strategy. | Re-evaluate the cache strategy against a real backend's actual consistency and conflict-handling guarantees. |

## Open decisions

No decision remains open beyond what `SPEC.md`/`CONTRACTS.md` already resolved.
The absolute-URL base-query pattern described in *Change summary* is a
technical design choice made during this planning pass, not a product or
contract decision, and does not require a separate approval gate.

| ID | Decision | Affected records | Impact | Status |
| --- | --- | --- | --- | --- |
| — | None remaining | — | — | `CLOSED` |

## Human plan approval

- **Decision:** `APPROVED`
- **Reviewer:** Babak Khajavi
- **Date:** 2026-08-13
- **Comments:** Approved for Jira creation and POC implementation. Manual RTK Query cache updates are approved for DummyJSON session-only mutations, while query tag definitions must remain in place for traceability and future invalidation.
