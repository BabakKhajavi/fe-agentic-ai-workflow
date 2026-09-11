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

## Forms

- Use Formik 2 for form state and submission lifecycle.
- Use Yup schemas for declarative validation.
- Use MUI controls and expose validation through `error`, helper text, and
  appropriate accessibility attributes.
- Do not mirror Formik values or errors in separate component state.
- Put non-trivial submission and form workflow logic in a feature custom hook.

## Redux and API access

- Configure the Redux Toolkit store under `src/app/`.
- Use RTK Query endpoints for application API access and remote server state.
- Define typed requests and responses at every endpoint boundary.
- Use cache tags consistently: queries provide tags and relevant mutations
  invalidate those tags.
- Prefer tag invalidation for normal server synchronization; use optimistic or
  manual cache updates only when the interaction requires them.
- Do not copy RTK Query results into ordinary slices.
- Keep transient local UI state out of Redux unless multiple distant consumers
  genuinely require it.

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

## Dependencies

- Use the platform and existing dependencies before adding a package.
- Document why a new dependency is needed in `IMPACT.md`.
- Do not add overlapping libraries for the same concern without approval.
- Commit the dependency lockfile when dependencies change.

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
