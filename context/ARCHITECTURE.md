# Architecture

This document describes both the AI-assisted delivery workflow and the target
architecture of the React application. Mandatory technical constraints are
defined in `TECHNICAL_REQUIREMENTS.md`, while day-to-day development conventions
are defined in `ENGINEERING_STANDARDS.md`.

## Architectural principles

- Keep product features independent and easy to locate.
- Separate UI rendering, application behavior, and external integrations.
- Make data flow and side effects explicit.
- Prefer simple local solutions until shared infrastructure is justified.
- Design for automated testing, accessibility, and observable failures.
- Record architectural decisions that materially affect future development.

## React application architecture

> **Unresolved conflict (2026-09-02):** everything below in "Technology
> baseline" and "Source organization" describes a single Vite application,
> written before `smooth-web` existed as the implementation target.
> `smooth-web` is actually five separate repositories built on Webpack with
> Module Federation (see "Module federation structure" below), not a single
> Vite app. This is a real architecture disagreement, not a naming issue —
> per this workspace's rule to stop and ask when sources disagree, do not
> silently apply this baseline to `smooth-web`. A human must decide whether
> to rewrite this baseline for the MFE reality or keep it for a future
> single-app scope, before agents plan or implement features against
> `smooth-web`.

### Technology baseline

The proof-of-concept application will use:

- React with TypeScript.
- Vite for local development and production builds.
- React Router with lazy-loaded route modules for client-side navigation.
- Material UI (MUI) as the only component and styling system.
- Formik 2 with Yup for forms and validation.
- Redux Toolkit for global client state.
- RTK Query for API access, server-state caching, and cache invalidation.
- Vitest and React Testing Library for automated component and behavior tests.
- ESLint and TypeScript checking as required quality gates.
- Playwright only when an end-to-end browser flow is required by a feature.

Exact dependency versions are recorded in each `smooth-web-*/package.json`.
Agents must follow the installed versions instead of assuming APIs from
another version.

### Module federation structure

`smooth-web` is five sibling repositories, not one application:

```text
smooth-web/
├── smooth-web-app/           # host shell — Webpack + ModuleFederationPlugin
├── smooth-web-data-mfe/      # remote: smooth_data
├── smooth-web-design-mfe/    # remote: smooth_ui
├── smooth-web-menu-mfe/      # remote: smooth_menu
└── smooth-web-settings-mfe/  # remote: smooth_settings
```

The host's `webpack.config.js` also declares `smooth_auth`
(`smooth-web-auth-mfe`) and `smooth_dashboard` (`smooth-web-dashboard-mfe`) as
remotes, but neither repo was present locally as of 2026-09-02 — confirm
their status with the user rather than assuming they don't exist or building
against them.

Each repo is independently versioned and independently deployed behind
`getMfeBaseUrl(...)`. Shared singletons (React, etc.) are declared in the
host's `shared` config — check it before assuming a dependency can float
independently in a remote.

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
├── components/
├── hooks/
├── services/
├── types.ts
├── index.ts
└── __tests__/
```

A feature owns its feature-specific components, state, contracts, and tests.
Code moves into a shared directory only after it has a genuine application-wide
use case.

Features must not import internal files from another feature. Cross-feature
usage must go through the feature's intentional public exports.

### Component model

- Components should have one clear responsibility.
- Pages and feature containers coordinate data and user flows.
- Presentation components receive typed props and avoid hidden external state.
- Reusable components belong in `src/components/`; feature-specific components
  stay with their feature.
- Business rules should be extracted into testable hooks or pure functions when
  embedding them in JSX would obscure behavior.

### Routing and lazy loading

- Route implementation modules are loaded lazily and form route-level Vite
  chunks.
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
5. Redux Toolkit slices for genuinely global client state.
6. Context only for stable framework concerns that are not application state.

Do not duplicate RTK Query server data in Redux slices or component state.

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

- Pure business logic receives unit tests.
- User-visible component behavior receives React Testing Library tests.
- Critical multi-page or integration flows may receive Playwright tests.
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

`smooth-web/` already exists as five independently versioned repositories
(see "Module federation structure" above), so there is no bootstrap step.
Agents must inspect the specific `smooth-web-*/` repository (or repositories)
named by the task's Jira issue — its current structure, installed
dependencies, and conventions — before planning or implementing a feature.
Do not assume the Vite/single-app technology baseline above applies until the
unresolved conflict noted at the top of this document is resolved by a
human.

### Integration principle

Confluence and Jira access should use configured, authenticated integrations.
Credentials and secrets must never be stored in specifications, prompts, agent
definitions, source code, or commits.
