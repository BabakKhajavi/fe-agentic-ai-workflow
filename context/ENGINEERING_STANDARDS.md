# Engineering Standards

## General

- Use TypeScript for application code.
- Prefer clear, direct implementations over speculative abstractions.
- Keep changes small enough to review against an approved Jira task.
- Do not mix unrelated refactoring with feature implementation.
- Remove unused code rather than commenting it out.

## Naming and organization

- Use `PascalCase` for React components and their files.
- Use `camelCase` for functions, hooks, variables, and non-component modules.
- Prefix custom hooks with `use`.
- Use descriptive domain names instead of generic names such as `data`, `item`,
  or `handler` when the domain meaning is known.
- Keep feature-specific code under `src/features/<feature-name>/`.

### Constants

- Do not hardcode a reused, user-facing, or domain-meaningful string or
  number inline; declare it as a named constant (`UPPER_SNAKE_CASE` for a
  primitive, or a typed `const` object/enum for a related group) and import
  it wherever it is used.
- Put feature-specific constants in `src/features/<feature-name>/constants.ts`.
- Put constants shared across features or across a repository's public
  surface in `src/constants/`.
- Cache tag names, RTK Query endpoint/base-query option keys, storage keys,
  and route paths follow this rule the same as user-facing copy.
- A literal that is genuinely unique to one call site and carries no
  reusable or domain meaning (for example, a one-off array index or a CSS
  pixel value) does not need to be extracted.

### Utility functions

- Do not define a utility function inside a component file.
- Put a utility function used only by one feature in
  `src/features/<feature-name>/utils/`.
- Put a utility function reused across features in that repository's
  shared `src/utils/`.
- A utility function should be a pure function with a typed signature; keep
  React-specific logic in a hook instead.

## React

- Use function components and hooks.
- Keep render logic free of unexpected side effects.
- Use effects only for synchronization with external systems.
- Do not store values in state when they can be derived during rendering.
- Use stable identifiers for list keys; do not use array indexes when ordering can
  change.
- Make loading, empty, error, and success states explicit.
- Load route modules lazily and provide accessible loading and error states.
- Keep orchestration and feature logic in feature custom hooks rather than page
  or presentation component bodies.

### Container/representational components

- Put a feature's container components in `containers/`: they call hooks,
  own data-fetching and derived state, and pass typed props down. They
  should render little or no markup directly.
- Put a feature's presentational components in `components/`: they take
  typed props and render JSX only — no data-fetching, no direct Redux or
  Context access, no orchestration.
- Keep every component file, container or presentational, at or under 300
  lines. When one grows past that, split it into smaller components or move
  logic into a hook or pure function.
- Reach for composition (children, render props, slots) before adding a new
  prop that only passes through several layers. If a value genuinely needs
  to reach a deeply nested descendant within one feature, expose it through
  that feature's `context/` provider and a matching hook instead of drilling
  it through props.
- A feature's `context/` provider holds only that feature's own internal UI
  state (for example, a modal's open/closed state). It is not a place to
  share state with another feature or repository — cross-module state goes
  through `smooth-web-data-mfe`'s Redux store instead.
- Add a `context/` provider only when a feature actually needs it; most
  features will not.

## TypeScript

- Define types at system boundaries and component interfaces.
- Prefer precise domain types over `any`.
- Narrow `unknown` values before use.
- Avoid non-null assertions unless the invariant is documented and enforced.
- Keep API contracts aligned with the feature's `CONTRACTS.md`.

## Testing

- Write tests from acceptance criteria and observable behavior.
- Prefer accessible queries such as role, label, and visible text.
- Avoid tests coupled to private component structure.
- Keep test data small and intention-revealing.
- Run type checking, linting, tests, and the production build before completion.
- Create a component's Jest + React Testing Library test scenarios in the
  same change that creates the component, not as a follow-up task.
- Update a component's test scenarios in the same change that changes its
  behavior; do not leave a test asserting the old behavior.
- Use Jest and React Testing Library for component and hook tests; use
  Playwright only for the task's critical multi-page or cross-MFE flow.
- Mock API responses with MSW request handlers rather than mocking
  `fetch`, RTK Query internals, or a custom API client directly.
- Keep MSW handlers close to the tests that use them (for example, a
  feature's `__tests__/mocks/` folder) and keep their response shapes
  aligned with `CONTRACTS.md`.

## Forms

- Use Formik 2 for form state and submission lifecycle.
- Use Yup schemas for declarative validation.
- Use MUI controls and expose validation through `error`, helper text, and
  appropriate accessibility attributes.
- Do not mirror Formik values or errors in separate component state.
- Put non-trivial submission and form workflow logic in a feature custom hook.

## Redux and API access

- Configure the Redux Toolkit store under `src/app/` of the repository that
  instantiates it (see the open question in `ARCHITECTURE.md` on whether
  that is `smooth-web-app` or `smooth-web-data-mfe`).
- Use RTK Query endpoints for application API access and remote server state.
- Define typed requests and responses at every endpoint boundary.
- Use cache tags consistently: every query that can be affected by a
  mutation provides tags, and the relevant mutation invalidates those tags.
- Prefer tag invalidation for normal server synchronization; use optimistic or
  manual cache updates only when the interaction requires them.
- Use `transformResponse` (or `transformErrorResponse`) on an endpoint when
  the raw response should be normalized, reshaped, or trimmed before it
  reaches the cache and its consumers, instead of reshaping it again in a
  hook or component.
- Do not copy RTK Query results into ordinary slices.
- Keep transient local UI state out of Redux unless multiple distant consumers
  genuinely require it.

### Data ownership across repositories (`smooth-web-data-mfe`)

- Author every RTK Query API slice in `smooth-web-data-mfe` under
  `apis/<feature-name>/`, named `<feature-name>-api-slice`. Do not define an
  API slice in a consuming repository.
- Author every global client-state slice in `smooth-web-data-mfe` as a
  Redux Toolkit slice, not an RxJS `Subject`/`BehaviorSubject` stream.
  Migrate a legacy stream (e.g. `auth-stream.ts`) to a slice when the task
  you are doing touches it.
- Expose new or changed slices from `smooth-web-data-mfe`'s
  `webpack.config.js` `exposes` entry so other repositories can consume
  them.
- In a consuming repository, wrap each imported slice in a
  feature-specific custom hook and let components use only that hook —
  never the imported slice, its selectors, or its endpoint hooks directly.
- Keep the feature name identical between `smooth-web-data-mfe`'s
  `apis/<feature-name>/` and the consuming repository's
  `src/features/<feature-name>/`.

## Accessibility

- Start with semantic HTML.
- Associate inputs with labels and errors.
- Preserve logical focus order and visible keyboard focus.
- Verify keyboard interaction for custom interactive behavior.
- Include accessible names for icon-only controls.

## Styling

- Use MUI as the only component and styling system.
- Use the shared MUI theme rather than repeating arbitrary values.
- Prefer MUI components and `sx`; use MUI `styled` for reusable styled elements
  and theme overrides for stable application-wide customization.
- Do not introduce CSS Modules, Sass, Tailwind, styled-components, direct Emotion
  usage, or another UI component framework.
- Design mobile-first and verify larger layouts.
- Keep visual states for hover, focus, active, disabled, loading, and error
  consistent.
- Avoid inline style values when a reusable token or class communicates intent.

### Design system (`smooth-web-settings-mfe`)

- Organize the shared component library by atomic-design category:
  `smooth-web-settings-mfe/src/design-system/<atoms|molecules|organisms|
  layouts|templates>/<ComponentName>/`, each folder containing
  `<ComponentName>.tsx`, `<ComponentName>.stories.tsx`, `index.ts`, and
  `__tests__/`.
- Before adding a new component that might be reusable, search the design
  system's existing categories for an equivalent before writing a new one.
- Write a Storybook story for every design-system component, covering its
  meaningful prop variations and states (default, loading, error, disabled,
  etc. as applicable).
- Build every design-system component from the shared MUI theme
  (`sx`, `styled`, theme tokens) — never a hardcoded color, spacing, or
  typography value the theme already defines.
- If a change to an existing shared component could affect its current
  consumers (a changed prop type, removed prop, or changed default
  visual behavior), stop and ask the human developer before making the
  change, even mid-task. A purely additive, backward-compatible change
  (a new optional prop with a safe default) does not require asking.

## Dependencies

- Use the platform and existing dependencies before adding a package.
- Document why a new dependency is needed in `IMPACT.md`.
- Do not add overlapping libraries for the same concern without approval.
- Commit the dependency lockfile when dependencies change.

### Shared singleton versions

- React, MUI (`@mui/material`, `@mui/icons-material`, `@mui/system`),
  Redux Toolkit, RTK Query, and `react-redux` are pinned, shared-singleton
  dependencies across every `smooth-web-*` repository. Match the version
  declared in the host's `webpack.config.js` `shared` config and
  `package.json` — do not install a different version in a remote.
- Declare each as `singleton: true` with a pinned `requiredVersion` in the
  repository's own `ModuleFederationPlugin` `shared` configuration.
- Before adding or bumping one of these libraries, check the host's
  `shared` config first; a version change must be coordinated across every
  repository and approved by a human, not made locally in one remote.
- Do not adopt React Router `8` until compatibility with the current
  routing/Module Federation setup is verified and a human confirms it; use
  the currently installed router version until then.

### Frozen version policy

- TypeScript, React, `react-dom`, MUI, Redux Toolkit, `react-redux`,
  Formik, and Yup are **frozen at the exact versions in
  `TECHNICAL_REQUIREMENTS.md` `TECH-148`** by company decision. This is a
  fixed pin, not "always track latest" — install exactly that version,
  even when a newer stable release exists.
- Do not bump one of these libraries to a newer release just because it's
  available. If a task seems to need a newer version (a bug fix, a needed
  API only in a later release), stop and ask the human developer for
  explicit approval before changing it — do not treat "a newer version
  exists" as sufficient reason on its own.
- If a human approves a version change, verify the toolchain around it
  still works (linting, the test runner, the build) before it lands, and
  update the frozen version recorded in `TECH-148` and here as part of
  that same approved change — not as a silent side effect of an unrelated
  task.
- Every repository moves to an approved new version together — never one
  remote ahead of the others (`TECH-149`/`TECH-151`).

## Comments and documentation

- Comments should explain constraints or reasons, not restate the code.
- Update SDD contracts when an approved interface changes.
- Update architecture context only for stable, application-wide decisions.

## Git and review

- Keep commits and diffs focused on approved work.
- Never commit secrets or local credentials.
- Review generated code with the same standards as human-written code.
- The Developer Report must disclose unresolved findings rather than quietly
  accepting them.

### Branching

- The main long-lived branches are `development`, `staging`, and
  `production`.
- Before creating a task branch, always ask the developer which branch is
  the source for this task. Never assume `development` or any other
  default.
- Name every task branch `<jira-issue-key>-<short-kebab-case-title>`, where
  the task number comes first and the short name is a meaningful,
  kebab-case summary derived from the Jira issue title, for example
  `SDM-123-customer-search-filter`.

## Required verification commands

The exact commands will be filled in after React initialization. The application
must provide scripts for:

```text
typecheck
lint
test
build
```
