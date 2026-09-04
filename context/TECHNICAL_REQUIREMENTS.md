# Technical Requirements

These requirements apply to every feature in the React application. A feature
may add stricter requirements but may not weaken these requirements without an
explicitly approved architectural decision.

## Platform

- `TECH-001` — The application must be implemented with React and TypeScript.
- `TECH-002` — The application must use Vite for local development and
  production bundling and must produce a repeatable production build.
- `TECH-003` — The application must run without browser-delivered secrets.
- `TECH-004` — Dependencies must be declared and locked by the repository's
  package manager.

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

## Type safety and quality

- `TECH-010` — Application code must pass TypeScript checking without errors.
- `TECH-011` — Application code must pass the configured lint rules.
- `TECH-012` — Production builds and automated tests must complete successfully
  before a feature is presented as done.
- `TECH-013` — Suppressed type or lint errors must include a specific,
  reviewable justification.

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
