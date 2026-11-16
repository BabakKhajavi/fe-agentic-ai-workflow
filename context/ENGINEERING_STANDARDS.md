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
  `rspack.config.js` `exposes` entry so other repositories can consume
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

### Design system (`smooth-web-design-mfe`)

- Keep `smooth-web-design-mfe` pure: it holds generic, domain-free
  atomic-design components only (`TECH-152`).
  - Name components by what they look like or do (`MediaCard`,
    `ConfirmModal`, `StatBar`), not by business concept (`ProductCard`,
    `OfferBanner`, `OrderStatusStepper`).
  - Components take all data through props. They never call APIs, read
    `smooth-web-data-mfe` state, or contain business rules.
  - A domain-specific component built from design-system pieces lives in
    its owning feature repository, under
    `src/features/<domain>/components/`.
- Follow the atomic-design structure that already exists in
  `smooth-web-design-mfe` (confirmed 2026-11-08):
  - categories at the root of `src/`: `src/<atoms|molecules|organisms|templates>/`;
  - components grouped in subfolders by kind, for example
    `src/atoms/buttons/contained-button.tsx` or
    `src/molecules/carousel/`;
  - each level exported through its `index.ts`.

  Add new components into the matching existing category and subfolder,
  in the same style. Do not introduce a parallel `design-system/` tree.
  The existing structure and components are accepted as they are.
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

### Build tooling (Rspack + Module Federation 2)

Decided 2026-11-11 (`TECH-153`–`TECH-157`).

- **Use Node 22.** Run `nvm use` (`.nvmrc`) before installing or building.
- **Scripts in every repository:**

  | Script | Command |
  | --- | --- |
  | `start` | `rspack serve` |
  | `build` | `rspack build` |
  | `typecheck` | `tsc --noEmit` |

  Run `typecheck` and `build` before handing work over for review.
- **Keep `rspack.config.js` short.** It calls `createSmoothRspackConfig`
  from `@smooth/web-build-config` and passes only repository-specific
  values: kind, federation name, port, entry, exposes, remotes, HTML
  template, and defines. Example:

  ```js
  import { createSmoothRspackConfig } from '@smooth/web-build-config';
  export default createSmoothRspackConfig({
    dirname: import.meta.dirname,
    kind: 'remote',
    name: 'smooth_core',
    port: 9007,
    exposes: { './CoreApp': './src/routes/CoreRoutes.tsx' },
    remotes: ['design', 'data'],
    html: { template: './public/index.html' },
  });
  ```
- **Change shared build behaviour in `smooth-web-build-config`, not in a
  repository.** This covers loaders, plugins, shared singletons, remote URLs,
  ports, and runtime plugins. Release a new version of the package and
  update every repository to it in the same change set.
- **Add a new remote in two places:**
  1. register it in the shared config's `REMOTES` (federation name,
     repository, port, environment base URLs);
  2. list it in the consumer's `remotes`.

  Never hardcode a remote URL in a repository.
- **Dev ports are fixed:**

  | Repository | Port |
  | --- | --- |
  | app | 9000 |
  | auth | 9001 |
  | design | 9002 |
  | data | 9003 |
  | payment | 9005 |
  | core | 9007 |
- **Name exposes after the app or module**, for example `./CoreApp`,
  `./PaymentApp`, `./AuthApp`, `./Store`, `./atoms`. Changing or removing an
  exposes key is a contract change and must be recorded in the feature's
  `CONTRACTS.md`.
- **Know the dev-mode limits.** They were found during the 2026-11-12
  migration.
  - A remote does not render standalone on its own port. Remotes never
    bundle React (`import: false`); run them through the host.
  - Hot updates work inside the host and inside each dev server, but not
    across federation boundaries.
  - `typecheck` in a consumer needs `@mf-types/`. Run `npm start` or
    `npm run build` first; they fetch the remote types.
  - Keep production-only `optimization` (module concatenation, a single
    runtime chunk) out of development, or React Refresh falls back to full
    reloads.
- **Do not add Babel, Webpack, or webpack-only plugins.** Use Rspack's
  built-in equivalents (SWC loader, `HtmlRspackPlugin`, built-in CSS).

### Shared singleton versions

- React, MUI (`@mui/material`, `@mui/icons-material`, `@mui/system`),
  Redux Toolkit, RTK Query, and `react-redux` are pinned, shared-singleton
  dependencies across every `smooth-web-*` repository. Match the version
  in `@smooth/web-build-config` `SHARED_SINGLETONS` and the host's
  `package.json`. Do not install a different version in a remote.
- Declare a shared singleton only in `SHARED_SINGLETONS`, never in a
  repository's own `rspack.config.js`.
- Before adding or bumping one of these libraries, check the shared config
  first. A version change must be a coordinated release of
  `@smooth/web-build-config`, adopted by every repository and approved by a
  human, not a local change in one remote.
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

## Releases and deployment

The full model and the client-onboarding checklist are in
`DEPLOYMENT.md` (`TECH-158`–`TECH-164`).

- **Don't put client logic or client values in app code or builds.** A new
  client is a registry entry plus a brand file, never a code change.
- **Branch flow:** merge to `staging` to reach staging. Cut a version with
  `prepare-release` (bump, tag, push); the `v*` tag publishes the build to
  sandbox. Never re-tag or overwrite a version.
- **Releasing to clients:** use the `smooth-web-release` pipelines from
  Bitbucket. Start with `DRY_RUN=true`, and with `CLIENT=ring:canary`, before
  widening to `ring:general` or `all`. Promote to sandbox before production.
- **Clients:**
  - add, rename, suspend, or activate them only in
    `smooth-web-release/clients.json`;
  - then run `node scripts/sync-pipeline-clients.js`, so the `CLIENT`
    dropdown matches;
  - `validate` fails otherwise.
- **Slugs** are fixed at onboarding: lowercase, digits, and hyphens.
  Never rename a slug; onboard a new one instead.
- **Brand files and brand assets:**
  - brand files follow the `Brand` / `BrandColors` v2 types;
  - upload brand JSON with `Cache-Control: public, max-age=0,
    must-revalidate, s-maxage=3600` and invalidate its path;
  - give brand assets a version in the file name and cache them
    immutably;
  - host assets in `brand-assets/clients/<slug>/`, not on third-party hosts.
- **Pipeline files:** change the package pipelines through the shared
  template, so all six stay identical. Never hand-edit one repository's
  pipeline.
- **Rollback** by running `rollback-*`. Never edit `clients/<slug>/index.html`
  or `state/` by hand.

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
