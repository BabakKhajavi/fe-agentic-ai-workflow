# Technical Requirements

These requirements apply to every feature in the React application. A feature
may add stricter requirements but may not weaken these requirements without an
explicitly approved architectural decision.

## Platform

- `TECH-001` — The application must be implemented with React and
  TypeScript, both frozen at the exact version in `TECH-148`.
- `TECH-002` — The application must use Vite for local development and
  production bundling and must produce a repeatable production build.
- `TECH-003` — The application must run without browser-delivered secrets.
- `TECH-004` — Dependencies must be declared and locked by the repository's
  package manager.
- `TECH-148` — TypeScript, React, `react-dom`, MUI (`@mui/material`,
  `@mui/icons-material`, `@mui/system`), Redux Toolkit (`@reduxjs/toolkit`,
  which includes RTK Query), `react-redux`, Formik, and Yup are **frozen**
  at the following exact versions, decided 2026-09-14. This is a fixed
  pin, not a "track latest" policy — do not bump any of these to a newer
  release without an explicit, separately approved decision, even when a
  newer stable version is published:
  - React / `react-dom` — `19.3.0`
  - TypeScript — `7.0.2`
  - `@reduxjs/toolkit` — `2.12.0`
  - `react-redux` — `9.3.0`
  - `@mui/material` / `@mui/icons-material` / `@mui/system` — `9.4.0`
  - Formik — `2.4.9`
  - Yup — `1.7.1`

  Every `smooth-web-*` repository must use exactly these versions. MUI "X"
  products (`@mui/x-data-grid`, `@mui/x-charts`, etc.) are frozen at the
  version compatible with the pinned `@mui/material` version at the time
  this list is set, which may not share the same major number as core
  MUI.
- `TECH-149` — React, MUI, Redux Toolkit, RTK Query, `react-redux`, and
  React Router (once adopted) must be declared as shared singletons
  (`singleton: true` with a pinned `requiredVersion`) in every
  repository's `webpack.config.js` `ModuleFederationPlugin` `shared`
  configuration, so exactly one instance of each loads across the host and
  all remotes.
- `TECH-150` — React Router `8` must not be adopted until its
  compatibility with `smooth-web-app`'s current routing and Module
  Federation configuration is verified and a human confirms the adoption.
  Until then, the currently installed router version is the baseline. This
  is a separate decision from `TECH-148`.
- `TECH-151` — A repository must not install a different version of a
  singleton-shared or `TECH-148` library than the frozen version used by
  every other repository. Changing a `TECH-148` version at all is a
  deliberate exception to the freeze: it requires explicit human approval,
  a coordinated change across every repository, and verification of
  breaking changes and toolchain compatibility (lint config, test runner,
  build config) before it lands — reported in the task's `IMPACT.md` or
  Developer Report.

## Routing and code splitting

- `TECH-005` — Application routes must be lazy-loaded so route implementation
  code is split from the initial application bundle.
- `TECH-006` — Lazy route elements must render through an intentional Suspense
  fallback that is accessible and visually consistent with the application.
- `TECH-007` — Route-loading failures must produce a recoverable error state
  rather than a blank screen.
- `TECH-008` — Shared application shells and providers may remain in the initial
  bundle when they are required by every route.

## UI and styling

- `TECH-090` — Material UI (MUI) must be the application's only UI component and
  styling system.
- `TECH-091` — Colors, typography, spacing, shapes, breakpoints, and component
  defaults must be defined through the MUI theme when they are application-wide.
- `TECH-092` — Application styling must use MUI components and supported MUI
  styling APIs such as `sx`, `styled`, and theme overrides.
- `TECH-093` — Tailwind, CSS Modules, Sass, styled-components, Emotion used
  directly outside MUI, and additional component libraries must not be added.
- `TECH-094` — `CssBaseline` must provide the application-wide browser reset.

## Design system

- `TECH-142` — A component that may be reusable across repositories must
  be created first in `smooth-web-settings-mfe`, classified as an atom,
  molecule, organism, layout, or template, before it is consumed by any
  remote or the host shell.
- `TECH-143` — Every reusable component in `smooth-web-settings-mfe`'s
  design system must have a Storybook story.
- `TECH-144` — `smooth-web-settings-mfe` must expose its design-system
  components through its `webpack.config.js` `ModuleFederationPlugin`
  `exposes` configuration so the host and other remotes can consume them.
- `TECH-145` — Before creating a new reusable component, the existing
  design system must be checked for an equivalent; an existing component
  must be reused, not duplicated.
- `TECH-146` — Changing an existing shared component in a way that could
  affect its current consumers requires explicit human approval before the
  change is made.
- `TECH-147` — Design-system components must be implemented using the
  shared, global MUI theme rather than component-local hardcoded style
  values.

## Forms and validation

- `TECH-100` — Forms must use Formik version 2.
- `TECH-101` — Declarative form validation schemas must use Yup.
- `TECH-102` — Form fields must integrate Formik state and validation with MUI
  controls without duplicating form state in React component state.
- `TECH-103` — Validation errors must be associated with their controls,
  understandable to users, and exposed to assistive technology.
- `TECH-104` — Submission logic and non-trivial form behavior must be handled by
  the feature's custom hooks rather than embedded in presentation components.

## Application state and API access

- `TECH-110` — Global client state must use Redux Toolkit (RTK).
- `TECH-111` — Server data and API requests must use RTK Query.
- `TECH-112` — The application must provide one configured Redux store through
  the application root and register RTK Query reducers and middleware.
- `TECH-113` — API endpoints must define TypeScript request and response types
  that agree with the feature's `CONTRACTS.md`.
- `TECH-114` — RTK Query caching must be used instead of copying server data into
  ordinary Redux slices or component state.
- `TECH-115` — Query endpoints must use tag types and `providesTags` where their
  cached data can be affected by mutations.
- `TECH-116` — Mutations must use `invalidatesTags` or an explicitly documented
  cache update strategy so affected queries become current.
- `TECH-117` — Cache lifetime, refetch behavior, polling, optimistic updates, and
  manual cache updates must be configured only when required by the user flow
  and documented in `SPEC.md` or `CONTRACTS.md`.
- `TECH-118` — Duplicate API clients or ad hoc `fetch` calls must not bypass RTK
  Query for application API access.
- `TECH-119` — Authentication tokens, when required, must be attached through a
  centralized RTK Query base-query configuration and must not be persisted in an
  unsafe browser location.
- `TECH-130` — All RTK Query API endpoint slices must be defined in
  `smooth-web-data-mfe`, organized by feature under `apis/<feature-name>/`,
  and named `<feature-name>-api-slice`.
- `TECH-131` — All global (non-server) client state must be defined as
  Redux Toolkit slices in `smooth-web-data-mfe` and registered in the
  shared Redux store. It must not be implemented as an RxJS
  `Subject`/`BehaviorSubject` stream (the legacy pattern, e.g.
  `auth-stream.ts`); an existing stream must be migrated to a Redux Toolkit
  slice when an approved task touches it.
- `TECH-132` — Every API slice and global-state slice defined in
  `smooth-web-data-mfe` must be exposed through its `webpack.config.js`
  `ModuleFederationPlugin` `exposes` configuration so other `smooth-web-*`
  repositories can consume it.
- `TECH-133` — A consuming repository must not define its own duplicate API
  slice or duplicate global-state slice for data owned by
  `smooth-web-data-mfe`; it must consume the slice exposed by
  `smooth-web-data-mfe` instead.
- `TECH-134` — A consuming repository must access `smooth-web-data-mfe`
  state and API slices only through its own feature-specific custom
  hook(s); components must not import an exposed slice, its selectors, or
  its endpoint hooks directly.
- `TECH-135` — The feature name used under
  `smooth-web-data-mfe/apis/<feature-name>/` (and its global-state slice)
  must match the feature name used in the consuming repository's
  `src/features/<feature-name>/` so ownership stays traceable across
  repositories.
- `TECH-136` — An endpoint must use `transformResponse` (or
  `transformErrorResponse`) whenever the raw API response needs
  normalization, reshaping, or trimming before it is cached, instead of
  repeating that transformation in a consuming hook or component. The
  transformed shape must be documented in `CONTRACTS.md`.
- `TECH-137` — Redux Toolkit slices (owned by `smooth-web-data-mfe`) must be
  used only for state that is genuinely cross-module or cross-app.
  Feature-internal state must not be added to the Redux store; use the
  feature's own Context provider instead (see `TECH-140`).

## Logic and hooks

- `TECH-120` — Pages and presentation components must focus on composition and
  rendering; workflow, state-selection, form-submission, and orchestration logic
  must live in respective custom hooks.
- `TECH-121` — Feature-specific hooks must live with their feature; only hooks
  used across multiple features belong in `src/hooks/`.
- `TECH-122` — Custom hooks must expose the smallest clear interface required by
  their consumers and must not hide unexpected side effects.
- `TECH-123` — Pure calculations and domain rules that do not require React must
  remain pure functions rather than being placed in hooks.
- `TECH-124` — A utility function must not be defined inside a component
  file. It must live in a `utils/` folder: under the owning feature
  (`src/features/<feature-name>/utils/`) when it is only useful to that
  feature, or under the repository's shared `src/utils/` when it is
  reusable across features in that repository.

## Component structure

- `TECH-138` — Each feature must separate container components
  (`containers/` — own data-fetching, hook orchestration, and derived
  state, with no independent markup) from presentational components
  (`components/` — typed props in, JSX out, no data-fetching or store/
  Context access of their own).
- `TECH-139` — No component file, container or presentational, may exceed
  300 lines. Split it into smaller components or extract logic into a hook
  or pure function before it does.
- `TECH-140` — When state must reach deeply nested descendants within one
  feature, use that feature's own Context provider (`context/`) instead of
  passing the value as a prop through intermediate components that do not
  use it themselves.
- `TECH-141` — A feature's Context provider must hold only that feature's
  internal state. State needed by another feature or repository must be
  owned by `smooth-web-data-mfe` instead (see `TECH-130` through
  `TECH-135`), not shared through a feature's Context provider.

## Type safety and quality

- `TECH-010` — Application code must pass TypeScript checking without errors.
- `TECH-011` — Application code must pass the configured lint rules.
- `TECH-012` — Production builds and automated tests must complete successfully
  before a feature is presented as done.
- `TECH-013` — Suppressed type or lint errors must include a specific,
  reviewable justification.
- `TECH-014` — A string or numeric literal that is reused, user-facing, or
  carries domain meaning (UI copy, error/status messages, storage or cache
  keys, RTK Query tag names, route paths, event or channel names, HTTP
  headers, etc.) must be declared as a named constant and imported wherever
  it is used, not repeated as an inline literal. A literal that is
  genuinely unique to one call site and carries no reusable meaning is not
  required to be extracted.

## Accessibility

- `TECH-020` — User-facing functionality must target WCAG 2.2 Level AA.
- `TECH-021` — All interactive functionality must be keyboard operable.
- `TECH-022` — Controls must have accessible names and visible focus states.
- `TECH-023` — Status, validation, and error information must not rely only on
  color.
- `TECH-024` — Semantic HTML must be preferred over simulated controls.

## Responsive design

- `TECH-030` — Core user flows must remain usable on mobile and desktop
  viewports.
- `TECH-031` — Content must not require unintended horizontal scrolling at
  supported viewport widths.

## Performance

- `TECH-040` — Features must avoid unnecessary rendering and oversized client
  dependencies.
- `TECH-041` — Images and other large assets must be appropriately sized and
  optimized.
- `TECH-042` — A feature that materially changes bundle size or loading behavior
  must document the impact in `IMPACT.md`.
- `TECH-043` — The Vite production build must preserve route-level code splitting
  and must not eagerly import lazy route implementation modules elsewhere.

## Security and privacy

- `TECH-050` — Credentials, tokens, and private configuration must not be stored
  in source code, SDD documents, prompts, or commits.
- `TECH-051` — Untrusted content must not be rendered as executable HTML.
- `TECH-052` — Sensitive values must not be written to browser logs.
- `TECH-053` — New dependencies must have a clear purpose and be reviewed before
  adoption.

## Reliability and user experience

- `TECH-060` — Asynchronous experiences must define loading, empty, success, and
  failure behavior where applicable.
- `TECH-061` — User-facing errors must explain what happened and, when possible,
  how to recover.
- `TECH-062` — The application must not silently discard user input.

## Testing

- `TECH-070` — Each accepted feature requirement must map to automated evidence
  or an explicitly documented manual verification step.
- `TECH-071` — Tests must focus on user-visible behavior and business rules.
- `TECH-072` — Defect fixes must include a regression test when reasonably
  possible.
- `TECH-073` — Every component must have Jest + React Testing Library test
  scenarios created in the same change that creates the component. A
  component must not be presented as done without them.
- `TECH-074` — Test scenarios must be updated in the same change that
  changes the behavior they cover. A test left asserting removed or
  changed behavior is a defect in that change, not a pre-existing
  condition to fix later.
- `TECH-075` — Component and hook tests must use Jest and React Testing
  Library. Critical multi-page or cross-MFE flows must use Playwright.
- `TECH-076` — Tests must mock API calls with MSW handlers rather than
  mocking `fetch`, RTK Query internals, or a custom API client directly.

## Traceability

- `TECH-080` — Feature requirements must use stable identifiers such as
  `REQ-001`.
- `TECH-081` — Plans, Jira work items, tests, and review evidence must reference
  relevant requirement identifiers.
- `TECH-082` — Deviations from approved requirements or architecture must stop
  implementation until a human approves the updated SDD artifacts.

## Items to confirm before executive demonstration

- Supported browsers and minimum versions.
- Hosting and deployment environment.
- Whether the demonstration uses mock or live backend services.
- Authentication and authorization needs.
- Data classification and privacy constraints.
- Executive demonstration device and target screen sizes.
