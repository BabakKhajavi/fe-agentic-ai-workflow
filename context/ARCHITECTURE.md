# Architecture

This document describes both the AI-assisted delivery workflow and the target
architecture of the React application. Mandatory technical constraints are
defined in `TECHNICAL_REQUIREMENTS.md`, while day-to-day development conventions
are defined in `ENGINEERING_STANDARDS.md`.

For the rationale behind this target architecture — a row-by-row comparison
of the legacy `toppers-web-ordering` app against the rules in this
workspace, with the rule ID backing each target-side claim — see
[`toppers-vs-smooth-web-comparison.html`](toppers-vs-smooth-web-comparison.html).
Open it directly in a browser; it is not rendered by this Markdown viewer.

## Architectural principles

- Keep product features independent and easy to locate.
- Separate UI rendering, application behavior, and external integrations.
- Make data flow and side effects explicit.
- Prefer simple local solutions until shared infrastructure is justified.
- Design for automated testing, accessibility, and observable failures.
- Record architectural decisions that materially affect future development.

## React application architecture

> **Build baseline resolved (2026-11-11):** the user decided that
> `smooth-web` builds with **Rspack 2 and Module Federation 2**, using one
> shared build config (see "Build tooling and shared config" below and
> `specs/active/rspack-mf2-migration/`). This replaces both the earlier
> single-app Vite baseline and the current Webpack 5 setup.
>
> - Until that feature's tasks land, the repositories still contain
>   `webpack.config.js`.
> - New work must follow the Rspack baseline.
> - "Source organization" below was written for a single application.
>   Apply it inside each repository, not across the federation.

### Technology baseline

The proof-of-concept application will use:

- React (with TypeScript), MUI, Redux Toolkit, `react-redux`, Formik, and
  Yup are **frozen at fixed, exact versions** as a deliberate company
  policy — see `TECHNICAL_REQUIREMENTS.md` `TECH-148` for the exact
  versions. This is a hard pin, not a "track latest" policy; changing one
  requires an explicit, separately approved decision.
- Rspack 2 (`@rspack/core`, `@rspack/cli`, `@rspack/dev-server`) for local
  development and production builds:
  - TypeScript and JSX are compiled by Rspack's built-in SWC loader;
  - Module Federation 2 comes from `@module-federation/enhanced/rspack`;
  - all six repositories build from the shared config
    `@smooth/web-build-config`;
  - Node 22 (`>=22.12`).
  Decided 2026-11-11.
- React Router `8` with lazy-loaded route modules for client-side
  navigation, **if confirmed compatible** with `smooth-web-app`'s current
  routing and Module Federation setup (see the open question below);
  otherwise the currently installed router version remains the baseline.
  Unlike the libraries above, React Router is not part of the `TECH-148`
  freeze — its version is a separate, explicit decision.
- Redux Toolkit for global client state.
- RTK Query for API access, server-state caching, and cache invalidation.
- Jest and React Testing Library for automated component and behavior
  tests (confirmed 2026-09-11 for `smooth-web`; previously written as
  Vitest. Jest is independent of the bundler, so the Rspack decision does
  not change it).
- MSW (Mock Service Worker) for mocking API responses in tests instead of
  mocking `fetch` or RTK Query internals directly.
- ESLint and TypeScript checking as required quality gates.
- Playwright for critical multi-page or cross-MFE end-to-end flows.

Exact dependency versions are recorded in each `smooth-web-*/package.json`.
Agents must follow the installed versions instead of assuming APIs from
another version.

**Open question, not yet confirmed:** React Router `8` compatibility with
`smooth-web-app`'s current routing and Module Federation configuration has
not been verified against the actual repository. A Planner/Architect Agent
must check the installed router version and confirm `8` is compatible (or
report that it is not) before `IMPACT.md`/`PLAN.md` proposes adopting it —
see `TECH-150`.

### Module federation structure

The target `smooth-web` is six sibling repositories, not one application
(decided 2026-11-08; see `EXECUTION_PLAN.md` Change 1):

```text
smooth-web/
├── smooth-web-app/          # host shell — shell, sidebar, app bar, routing, white-labeling
├── smooth-web-data-mfe/     # remote: smooth_data — all APIs, global state, config
├── smooth-web-design-mfe/   # remote: smooth_ui — all reusable components
├── smooth-web-core-mfe/     # remote: smooth_core — Locations, Menu, Offers, Orders
├── smooth-web-payment-mfe/  # remote: smooth_payment — all payment features
└── smooth-web-auth-mfe/     # remote: smooth_auth — whole authentication flow and its components
```

**Current state does not match the target yet (inspected 2026-11-08):**

- `smooth-web-core-mfe` was scaffolded locally on 2026-11-08. It has no
  commits, no Bitbucket remote yet, and dev port 9007.
- The old local `smooth-web-menu-mfe`, `-location-mfe`, `-offers-mfe`,
  and `-order-mfe` folders were removed on 2026-11-08. Their scope moves to
  `smooth-web-core-mfe`.
- The local `smooth-web-payment-mfe` and `smooth-web-auth-mfe` folders still
  carry another repository's git remote and federation name. They must
  become `smooth_payment` and `smooth_auth`.
- The host's `webpack.config.js` still declares `smooth_dashboard`,
  `smooth_settings`, and `smooth_menu`, and has no core or payment remote.

Do not plan feature work against `smooth-web-core-mfe` or
`smooth-web-payment-mfe` until the infrastructure steps in
`EXECUTION_PLAN.md` are approved and done.

Each repository is versioned and deployed independently. Remote URLs come
from the remote registry in `@smooth/web-build-config`. At runtime they are
resolved by its Module Federation 2 runtime plugin, which applies the
release manifest's version or falls back to the environment base URL. This
replaces the hand-written `createPromiseRemote`/`getMfeBaseUrl` helpers.
Shared singletons are declared once in the shared config. Check it before
assuming a dependency can float independently in a remote.

React, MUI, Redux Toolkit, RTK Query, `react-redux`, and React Router
(once/if adopted) must all be declared as shared singletons
(`singleton: true` with a pinned `requiredVersion`) in
`@smooth/web-build-config` `SHARED_SINGLETONS` (consumed by every repository's `rspack.config.js`), at the
versions in "Technology baseline" above, so exactly one instance of each
loads across the host and every remote. See `TECH-148` through `TECH-151`.

### Build tooling and shared config

Decided 2026-11-11. Spec: `specs/active/rspack-mf2-migration/`.

- **Bundler:** Rspack 2, with the built-in SWC loader, `HtmlRspackPlugin`,
  `@rspack/plugin-react-refresh`, and built-in CSS. Babel is not used for
  builds.
- **Federation:** Module Federation 2, from
  `@module-federation/enhanced/rspack`, at one exact version shared by all
  repositories.
  - Every remote emits `remoteEntry.js` and `mf-manifest.json`.
  - Consumers reference remotes through `mf-manifest.json`.
- **Shared config:** `@smooth/web-build-config`, in the tooling repository
  `smooth-web-build-config`. It owns:
  - the base Rspack config (`createSmoothRspackConfig`);
  - the shared-singleton map (`SHARED_SINGLETONS`);
  - the remote registry (`REMOTES`: federation names, ports, environment
    base URLs);
  - the remote-resolver runtime plugin.

  Each repository's `rspack.config.js` declares only its own values: kind,
  name, port, entry, exposes, remotes, HTML, and defines.
- **Runtime remote resolution** is the foundation of the per-client release
  loader (Execution Plan AD-2.7). A runtime plugin rewrites remote entries
  from the release manifest before the remotes initialise, so a client can
  be pinned or rolled back without rebuilding.
- **Typing:** SWC does not type-check, so every repository has a `typecheck`
  script (`tsc --noEmit`) with `isolatedModules: true`. Federated type
  generation replaces the `declare module 'smooth_*/…'` stubs, if adopted
  (`DEC-003` in the spec).

See `TECHNICAL_REQUIREMENTS.md` `TECH-153` through `TECH-157`.

### Deployment and per-client releases

Decided 2026-11-12 to 2026-11-16. The full description and onboarding
steps are in `DEPLOYMENT.md`.

- **Build once, share everywhere.** Each repository version is built once
  and shared by all clients. Branding is runtime data
  (`/configs/<slug>-brand.json`), never baked into a build.
- **Each client has its own entry page,** `clients/<slug>/index.html`, in
  each environment's app bucket. The CloudFront Function
  `client-entry-rewrite` serves it for `/<slug>/…`.
  - In **staging**, the entry page is a copy of the latest shell.
  - In **sandbox and production**, it is the bundle's shell HTML with
    `window.__SMOOTH_CLIENT_RELEASE__` inlined, which pins every remote's
    version through the Module Federation runtime resolver.
- **Package pipelines build and publish code; `smooth-web-release` decides
  which client runs which code.**
  - `deploy-staging`, `create-bundle`, `promote-*`, `rollback-*`;
  - client selection from `clients.json`;
  - state committed to `state/<env>/<slug>.json`.
- **Production only receives tested bundles.** A bundle must already be in
  sandbox, the deployment group must approve, and rollback never rebuilds.

See `TECHNICAL_REQUIREMENTS.md` `TECH-158`–`TECH-164`.

### Data and global state ownership (`smooth-web-data-mfe`)

`smooth-web-data-mfe` is the single owner of server-data access and global
client state for the whole federated application:

- RTK Query API endpoint slices live under
  `smooth-web-data-mfe/src/apis/<feature-name>/<feature-name>-api-slice.ts`
  (or equivalent), one feature folder per business capability.
- Global client state — previously implemented as RxJS `Subject`/
  `BehaviorSubject` streams such as `auth-stream.ts` — must be implemented as
  Redux Toolkit slices instead, registered in the shared Redux store
  alongside the RTK Query API slices.
- Both kinds of slices are exposed from `smooth-web-data-mfe`'s
  `rspack.config.js` `exposes` entry (Module Federation 2) so the host
  and other remotes can consume them.
- A consuming repository (`smooth-web-app`, `smooth-web-core-mfe`,
  `smooth-web-payment-mfe`, `smooth-web-auth-mfe`,
  `smooth-web-design-mfe`) must not define its
  own copy of state or API access already owned by `smooth-web-data-mfe`. It
  wraps the imported slice in its own feature-specific custom hook, and
  components consume only that hook — never the imported slice directly.
  This keeps a single source of truth for each piece of shared state.
- The feature name must match between `smooth-web-data-mfe` and the
  consuming repository (for example, `apis/orders` in `smooth-web-data-mfe`
  paired with `src/features/orders/` in the consumer) so a reviewer can
  trace ownership across repositories.

See `TECHNICAL_REQUIREMENTS.md` `TECH-130` through `TECH-135` for the
enforceable rules and `ENGINEERING_STANDARDS.md` for naming conventions.

**Open question, not yet confirmed:** where the single shared Redux store
itself is instantiated (in the host `smooth-web-app`, or in
`smooth-web-data-mfe` and exposed to the host) has not been stated. Module
Federation requires state libraries like `redux`/`react-redux` to be shared
singletons; confirm the store's instantiation point and its `shared` config
entry with a human before a Planner/Architect Agent relies on it.

### Design system ownership (`smooth-web-design-mfe`)

`smooth-web-design-mfe` is the single owner of the shared, reusable
component library for the whole federated application, built with atomic
design:

- It is **pure and domain-free** (`TECH-152`, decided 2026-11-08):
  - No component, folder, or prop is named after a business domain.
  - It contains no business rules.
  - It does not access APIs or global state.

  Domain-specific composition belongs in the owning feature repository
  (`smooth-web-core-mfe`, `smooth-web-payment-mfe`, `smooth-web-auth-mfe`,
  or `smooth-web-app`).

- Every component that may be reusable beyond the feature that first needs
  it must be created in `smooth-web-design-mfe`, classified as an atom,
  molecule, organism, layout, or template, before it is consumed anywhere
  else.
- Every component in this design system has a Storybook (`storybook.js`)
  story.
- These components are exposed from `smooth-web-design-mfe`'s
  `rspack.config.js` `exposes` entry (Module Federation 2) so the host
  shell and every remote can consume them.
- Before creating a new reusable component, check the existing design
  system for an equivalent. Reuse it rather than creating a duplicate.
- If an existing shared component needs to change and that change could
  affect its current consumers, stop and get explicit human approval
  before changing it. Do not silently modify shared component behavior
  that other repositories already depend on.
- The whole design system is built from the shared, global MUI theme
  (colors, typography, spacing, shape, breakpoints, component defaults) —
  a design-system component must not hardcode style values the theme
  already defines.

This is a separate ownership boundary from `smooth-web-data-mfe` (data/
state, above): `smooth-web-design-mfe` owns shared **presentation**
(components), `smooth-web-data-mfe` owns shared **data and state**. A
consuming repository composes the two through its own feature-specific
hooks and containers.

See `TECHNICAL_REQUIREMENTS.md` `TECH-142` through `TECH-147` for the
enforceable rules and `ENGINEERING_STANDARDS.md` for folder and naming
conventions.

### Source organization

The target source structure is:

```text
sample-react-app/src/
├── app/              # Application composition, providers, and routing
├── features/         # Feature-owned UI, behavior, API access, and tests
├── components/       # Reusable application-wide presentation components
├── services/         # Shared external-system and HTTP infrastructure
├── hooks/            # Shared React hooks
├── types/            # Shared TypeScript types
├── constants/        # Shared, application-wide named constants
├── utils/            # Pure shared utilities
├── styles/           # Global styles, tokens, and themes
├── test/             # Shared test setup and helpers
└── main.tsx          # Browser entry point
```

Not every directory must exist initially. Create a directory only when the
application has code that belongs in it.

### Feature boundaries

Each business capability should live under `src/features/<feature-name>/` and
may contain:

```text
<feature-name>/
├── containers/       # Container components: own data/behavior, no markup of their own
├── components/       # Presentational components: typed props in, JSX out
├── context/          # Optional — feature-internal state only, never cross-module state
├── hooks/
├── services/
├── utils/
├── types.ts
├── constants.ts
├── index.ts
└── __tests__/
```

A feature owns its feature-specific components, state, contracts, and tests.
Code moves into a shared directory only after it has a genuine application-wide
use case.

Features must not import internal files from another feature. Cross-feature
usage must go through the feature's intentional public exports.

### Component model

Each feature follows the container/representational pattern:

- A **container** component (`containers/`) owns data-fetching, hook
  orchestration, and derived state, and passes typed props down. It renders
  little or no markup of its own.
- A **presentational** component (`components/`) receives typed props,
  renders JSX, and avoids hidden external state, direct data-fetching, or
  Redux/Context access of its own.
- No component file, container or presentational, may exceed 300 lines.
  Split it into smaller components or extract logic into a hook or pure
  function before it does.
- Prefer composition (children, render props, slots) over passing the same
  prop through several intermediate components. When state must reach
  deeply nested descendants within one feature, use that feature's own
  `context/` provider instead of drilling props through layers that don't
  use the value themselves.
- A component reused only within one repository's own features belongs in
  that repository's `src/components/`; feature-specific components stay
  with their feature. A component that may be reusable across
  repositories belongs in the shared atomic design system owned by
  `smooth-web-design-mfe` instead (see "Design system ownership"
  below) — create it there first, then consume it via Module Federation.
- Business rules should be extracted into testable hooks or pure functions when
  embedding them in JSX would obscure behavior.

### Routing and lazy loading

- Route implementation modules are loaded lazily and form route-level
  Rspack chunks.
- The application shell, Redux provider, MUI theme provider, router, error
  boundary, and route-loading fallback are composed under `src/app/`.
- Lazy route modules must not be eagerly imported by the shell or route table.
- Every lazy route must have an accessible loading experience and a recoverable
  failure experience.

### State management

Use the narrowest state mechanism that satisfies the requirement:

1. Component state for local interaction.
2. Formik 2 state for form values, touched state, errors, and submission state.
3. URL state for navigation, filtering, and shareable views.
4. RTK Query cache for remote server state.
5. A feature's own `context/` provider for internal state shared across that
   feature's containers and components (for example, a modal's open/closed
   state) — never for state another feature or module needs.
6. Redux Toolkit slices, owned by `smooth-web-data-mfe`, for genuinely
   cross-module or cross-app global client state.
7. Context otherwise only for stable framework concerns that are not
   application state.

A feature may pair its `context/` provider with its own custom hook (for
example `useOrdersContext`) so consumers read and update that state without
calling `useContext` directly. Do not put feature-internal state in the
Redux store, and do not reach for a feature's `context/` provider to share
state that a different feature or repository also needs — that belongs in
`smooth-web-data-mfe` (see "Data and global state ownership" above).

Do not duplicate RTK Query server data in Redux slices, Context, or
component state.

### Data and integrations

- Components must not scatter direct HTTP calls throughout the UI.
- Application API calls must be declared as RTK Query endpoints.
- Request and response shapes must be represented by TypeScript types and
  documented in the feature's `CONTRACTS.md`.
- Query endpoints provide cache tags and mutations invalidate affected tags or
  perform an explicitly documented cache update.
- Endpoint hooks are wrapped by feature custom hooks when orchestration,
  transformation, or feature-specific behavior is required.
- Loading, empty, success, and failure states must be intentionally designed.
- Mock data used for the executive demonstration must be clearly separated from
  production integration code.
- Secrets must never be shipped in the browser bundle.

### Styling

- MUI is the only component and styling system.
- Use an application MUI theme for color, spacing, typography, shapes,
  breakpoints, elevation, and stable component overrides.
- Use MUI components, `sx`, `styled`, and theme overrides for implementation.
- Keep feature-specific styled components near the feature that owns them.
- Do not add Tailwind, CSS Modules, Sass, styled-components, another UI library,
  or direct Emotion usage outside the MUI abstraction.
- Use `CssBaseline` for global normalization.
- Responsive behavior and accessible interaction states are required, not
  optional polish.

### Forms and validation

- Formik 2 owns form values, touched state, validation state, and submission
  lifecycle.
- Yup provides declarative validation schemas.
- MUI provides form controls and visual states.
- Feature custom hooks own submission and non-trivial workflow logic.
- Presentation components render the form contract exposed by those hooks.

### Logic placement

- Pages compose features and route-level layout.
- Components render typed inputs and user-visible states.
- Feature custom hooks coordinate Redux selectors, RTK Query endpoint hooks,
  Formik behavior, navigation, and user workflows.
- Pure domain rules remain framework-independent functions.
- Hooks should not be created for logic that does not use React behavior.

### Error handling

- Expected failures must produce clear user-facing states.
- Unexpected rendering failures should be contained by an application-level
  error boundary where appropriate.
- Technical diagnostics must not expose credentials or sensitive data.
- Agents must not hide errors solely to make the demonstration appear successful.

### Testing architecture

- Pure business logic receives Jest unit tests.
- Every component receives Jest + React Testing Library test scenarios,
  created in the same change that creates the component and updated in the
  same change that changes its behavior — not deferred to a later task.
- Critical multi-page or cross-MFE integration flows receive Playwright
  tests.
- API calls are mocked with MSW handlers in tests, not by mocking `fetch`
  or RTK Query internals directly.
- Tests should verify observable behavior instead of component internals.
- Each feature plan must map acceptance criteria to verification evidence.

### Dependency direction

```text
app
 ├── features
 │    ├── shared components
 │    ├── shared services
 │    └── shared utilities/types
 └── shared components/services/utilities/types
```

Shared code must not depend on a specific feature. Infrastructure must not
import page or presentation components.

### Architecture decisions

Material decisions that change this baseline must be documented in the relevant
feature's `IMPACT.md` and approved in `PLAN.md`. If the decision becomes a stable
application-wide convention, update this document after approval.

---

## Agentic workflow architecture

### System boundary

The proof of concept consists of:

- Confluence, Jira, or a developer-provided requirement description as the
  upstream product-requirement source, depending on how the feature was
  intaken.
- `ai-workflow/` as the local SDD and agent-orchestration workspace.
- Jira as the approved-work tracking system.
- `smooth-web/` (five repositories — see "Module federation structure" above)
  as the implementation target.
- Git as the change history and review boundary, per `smooth-web` repository.

### High-level flow

```text
Product requirement source
(Confluence, Jira ticket, or developer prompt)
             |
             v
        Intake agent
             |
             v
         Spec agent
  REQUIREMENTS / SPEC / CONTRACTS
             |
             v
    Planner / architect agent
         IMPACT / PLAN
             |
             v
       Human approval gate
             |
             v
          Jira agent
       Epic and work items
             |
             v
       Developer agent
             |
             v
       React implementation
             |
             v
        Review agent
             |
             v
        Human review gate
             |
             v
            Done
```

### Artifact lifecycle

Each feature receives a stable identifier such as `customer-search`. Its local
artifacts live at:

```text
ai-workflow/specs/active/customer-search/
```

The literal `<feature>` directory currently acts as a scaffold and will later be
replaced or copied for a real feature identifier.

Each requirement should receive an identifier such as `REQ-001`. The same
identifier must be referenced by acceptance criteria, plan steps, Jira work,
tests, and review evidence where applicable.

### Control boundaries

- Intake and specification agents may write SDD documents but not application
  code.
- The planner may inspect application code but only writes `IMPACT.md` and
  `PLAN.md`.
- The Jira agent may create Jira work only after explicit human approval.
- The developer may change application code only for an approved Jira task.
- The Developer Agent reports verification evidence and unresolved findings; a
  human performs the final POC review.
- Production deployment is outside this proof of concept.

### Human approvals

The first approval gate validates product intent, technical impact, contracts,
scope, risk, and implementation order. The second approval gate validates the
working feature and the review evidence.

Approval must be explicit and recorded. The absence of feedback is not approval.

### React application lifecycle

`smooth-web/` already exists as six independently versioned repositories
(see "Module federation structure" above), so there is no bootstrap step.
Agents must inspect the specific `smooth-web-*/` repository (or repositories)
named by the task's Jira issue — its current structure, installed
dependencies, and conventions — before planning or implementing a feature.
Apply the Rspack + Module Federation 2 baseline above, and read "Source
organization" per repository.

### Integration principle

Confluence and Jira access should use configured, authenticated integrations.
Credentials and secrets must never be stored in specifications, prompts, agent
definitions, source code, or commits.
