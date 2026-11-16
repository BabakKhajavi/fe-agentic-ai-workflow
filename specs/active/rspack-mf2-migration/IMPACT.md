# Impact Assessment: Rspack and Module Federation 2 migration with a shared build config

## Document control

| Field | Value |
| --- | --- |
| Feature slug | `rspack-mf2-migration` |
| Status | `READY_FOR_APPROVAL` |
| Repository inspected | `smooth-web-app`, `smooth-web-data-mfe`, `smooth-web-design-mfe`, `smooth-web-core-mfe`, `smooth-web-auth-mfe`, `smooth-web-payment-mfe` |
| Revision or state | Working trees on 2026-11-11. `smooth-web-core-mfe` is uncommitted (scaffolded 2026-11-08). The others are at their last commits ("updated package-lock"). |
| Human plan approval | `APPROVED` (2026-11-12) |

## Repository baseline

| Concern | Observed implementation | Evidence | Alignment |
| --- | --- | --- | --- |
| React and TypeScript | React 19.2.3, TS ^5.9.3. Every repository has `"type": "module"`. `tsc --noEmit` gives 0 errors in all six, with or without `isolatedModules`. | `package.json`, `tsconfig.json`, tsc run 2026-11-11 | Aligned |
| Bundler / Module Federation | Webpack ^5.104.1 + `webpack.container.ModuleFederationPlugin` (MF 1), babel-loader, html-webpack-plugin, `@pmmmwh/react-refresh-webpack-plugin`. Remotes load through `createPromiseRemote.js`, an injected `<script>` tag with `window` globals, a hardcoded old repository map, and a `Date.now()` cache-buster. | `*/webpack.config.js`, `*/scripts/utils/createPromiseRemote.js` | Gap: this feature |
| Shared singletons | The same map is duplicated in six configs. React and react-dom are `strictVersion 19.2.x`, with `import: false` in remotes. | each `webpack.config.js` | Aligned. It is centralised by this feature. |
| Host remotes | `smooth_auth`, `smooth_ui`, `smooth_data`, `smooth_dashboard`, `smooth_settings`, `smooth_menu`. Routes import `smooth_menu/MenuApp`, `smooth_dashboard/DashboardApp`, and `smooth_settings/SettingsApp`. Three of these repositories no longer exist, so the host cannot run end to end today. | `smooth-web-app/webpack.config.js`, `src/routes/*-routes.tsx` | Gap: `DEC-002` |
| Remote identities | `smooth-web-auth-mfe` is still `smooth_menu` (port 9006, `./MenuApp`). `smooth-web-payment-mfe` is still `smooth_settings` (port 9005, `./SettingsApp`). | their `webpack.config.js` | Gap: `DEC-002` |
| Remote typing | `src/types/mfe.d.ts` contains bare `declare module 'smooth_*/…'` stubs, so every remote import is `any`. | `smooth-web-app/src/types/mfe.d.ts` | Gap: `DEC-003` |
| Host HTML | The 511-line `public/index.html` uses `<%= APP_VERSION %>`, `GIT_SHA`, `API_URL`, and `BRAND_ASSET_BASE_URL`. An inline script loads the release manifest and runtime config before bootstrap. | `smooth-web-app/public/index.html` | Must render identically under `HtmlRspackPlugin` |
| Tests and quality scripts | No test, lint, or typecheck scripts; `test` echoes an error. | `package.json` | Gap. `typecheck` is added here; lint and test are out of scope. |
| Node | Local default is v16.19.0. v22.22.0 is available through nvm. Pipelines use `node:22`. | `node -v`, `bitbucket-pipelines.yml` | Gap: `.nvmrc` |

## Change summary

Each repository's Webpack config is replaced by a short `rspack.config.js`
that calls `createSmoothRspackConfig` from a new shared package. The shared
package owns:

- SWC compilation and React Refresh;
- HTML generation;
- MF 2 with `SHARED_SINGLETONS` and `manifest: true`;
- the remote registry;
- a runtime plugin that replaces `createPromiseRemote`. It keeps today's
  resolution order (release-manifest version, then environment base URL,
  then `?v=` build version) and drops `Date.now()`.

Runtime behaviour is otherwise unchanged.

## Affected areas

| ID | Action | Path | Change | Traceability |
| --- | --- | --- | --- | --- |
| `IMP-001` | Create | `../smooth-web/smooth-web-build-config/` (new repository, `DEC-001` A) | `package.json` (`@smooth/web-build-config`, ESM, pinned dependencies), `src/index.js` (`createSmoothRspackConfig`, `getAppEnv`), `src/shared.js` (`SHARED_SINGLETONS`), `src/remotes.js` (`REMOTES`, `resolveRemoteEntry`), `runtime/remote-resolver.js`, `test/*.test.js` (`node --test`), `README.md`. | `REQ-003`, `REQ-006`, `REQ-007`; `CON-001`, `CON-003`, `CON-004`, `CON-006` |
| `IMP-002` | Modify | each repository's `package.json` | Remove the Webpack and Babel dev dependencies. Add `@rspack/core`, `@rspack/cli`, `@rspack/dev-server`, `@rspack/plugin-react-refresh`, `@module-federation/enhanced`, and `@smooth/web-build-config`. Scripts: `rspack serve` / `rspack build` / `typecheck`. Add `engines`. | `REQ-001`, `REQ-003`, `REQ-009`, `REQ-010`; `CON-002`, `CON-007` |
| `IMP-003` | Create / Delete | `*/rspack.config.js`; delete `*/webpack.config.js`, `*/.babelrc` | The repository-specific config only (`CON-002`). | `REQ-001`, `REQ-007` |
| `IMP-004` | Delete | `smooth-web-app/scripts/utils/{createPromiseRemote,getMfeBaseUrl}.js`; `smooth-web-{core,auth,payment}-mfe/src/scripts/utils/*` | Replaced by the shared registry and runtime plugin. | `REQ-006` |
| `IMP-005` | Create / Modify | `*/.nvmrc`, `*/tsconfig.json` | Node 22; `isolatedModules: true`. | `REQ-009`, `REQ-010` |
| `IMP-006` | Modify (`DEC-002`) | `smooth-web-app/src/routes/authenticated-routes.tsx`, `unauthenticated-routes.tsx`, `src/types/mfe.d.ts` | Routes import `smooth_auth/AuthApp`, `smooth_core/CoreApp`, and `smooth_payment/PaymentApp`. The dashboard, settings, and menu imports are removed. The route paths these map to must be confirmed during the task, without inventing new routes (see `ADEC-001`). | `REQ-004`; `CON-004` |
| `IMP-007` | Modify (`DEC-002`) | `smooth-web-auth-mfe` (`MenuRoutes` → `AuthRoutes`, `menu-provider` → `auth-provider`, `#menu-root`, `package.json` name, port 9001); `smooth-web-payment-mfe` (`SettingsRoutes` → `PaymentRoutes`, `settings-provider` → `payment-provider`, `package.json` name, port 9005) | Give both placeholders their own identity. Mechanical renames only. | `REQ-004`; `CON-004` |
| `IMP-008` | Modify (`DEC-003`) | the remotes' `rspack.config.js` (`dts`), consumers' `src/types/mfe.d.ts`, `.gitignore` (`@mf-types`) | Federated types. | `REQ-012`; `CON-008` |
| `IMP-009` | Modify | `smooth-web-app/bitbucket-pipelines.yml` and the remotes' pipelines | Only the build invocation changes (`npm run build` stays the same name, so there is likely no change). The manifest repository list (`DEC-002`) is left to Phase 1d/3. | `REQ-001` |
| `IMP-010` | Modify | workflow `context/*`, `agents/*`, `templates/*`, `CLAUDE.md` | Rspack + MF 2 baseline, the shared config, and `TECH-153`–`TECH-157`. Done as part of this spec so reviewers see the target rules. | `REQ-011` |

## State and data impact

- **Form state:** Not needed.
- **Global client state:** Not needed. The store stays in `smooth-web-data-mfe`.
- **Server state and API:** Not needed.

## Quality impact

- **Accessibility:** none.
- **Security and privacy:**
  - Dropping the `Date.now()` cache-buster does not change trust. Remote
    URLs still come only from the registry or a same-origin release
    manifest.
  - The new dependencies come from the npm registry at exact versions, with
    lock files committed.
- **Performance and bundle:**
  - Faster builds and HMR are expected.
  - Bundles stay content-hashed, and shared singletons stay unchanged.
  - The host's `runtimeChunk: 'single'` and `splitChunks` combined with MF 2
    are checked in `RISK-002`.
- **Testing:**
  - `node --test` unit tests cover `resolveRemoteEntry` (`AC-004`) and the
    singleton map.
  - Builds, typecheck, and a manual local federation demo cover the rest
    (`AC-001`–`AC-003`, `AC-007`).

## Dependencies

| Dependency | Existing/new | Purpose | Required by | Risk |
| --- | --- | --- | --- | --- |
| `@rspack/core` 2.3.2 | New | Bundler | `IMP-001`, `IMP-002` | Major version 2. Node ≥ 22.12; ESM-only (`DEP-001`/`DEP-002`). |
| `@rspack/cli` 2.3.2, `@rspack/dev-server` 2.2.2 | New | `rspack build` / `rspack serve` | `IMP-002` | Low |
| `@rspack/plugin-react-refresh` 2.0.2 | New | HMR | `IMP-001` | Low |
| `@module-federation/enhanced` 2.9.3 | New | MF 2 plugin, runtime, manifest, dts | `IMP-001` | Its declared peer is `webpack ^5`. The Rspack entry point does not need webpack at runtime; to be verified at first install (`RISK-003`). |
| `@smooth/web-build-config` 1.0.0 | New (internal) | Shared config | All repositories | `POC-001`: `file:` path until the Bitbucket repository exists. |
| Webpack, Babel, html-webpack-plugin, react-refresh-webpack-plugin, style/css-loader, `@types/webpack` | Removed | — | — | — |

## POC shortcuts and production gaps

| ID | Shortcut or gap | POC rationale | Demo impact | Production follow-up |
| --- | --- | --- | --- | --- |
| `RISK-001` | The shared config is consumed through `file:../smooth-web-build-config`. | Its Bitbucket repository does not exist yet. | Local only; CI cannot install it. | Phase 1a: create the repository, tag `v1.0.0`, switch to `git+ssh://git@bitbucket.org/smoothpay/smooth-web-build-config.git#v1.0.0`. |
| `RISK-002` | The host's `runtimeChunk: 'single'` and vendor `splitChunks` may conflict with MF 2 shared-module initialisation. | Not verifiable before the first build. | It may require removing `runtimeChunk` and `splitChunks` from the host. | Decided during `TASK-003`, with the evidence recorded in the developer report. |
| `RISK-003` | `@module-federation/enhanced` declares a `webpack` peer. | npm may warn. | None if it is only a warning. | If npm errors, add `legacy-peer-deps` only for that package, or install `webpack` as a type-only dev dependency. The choice is recorded. |
| `RISK-004` | `HtmlRspackPlugin` template syntax differs slightly from html-webpack-plugin (lodash templates). | The host template uses only `<%= VAR %>`, which is supported. | None expected. | `AC-007` grep check. |
| `RISK-005` | The new remotes' staging, release, and production URLs are assumed (`ASM-001`). | Hostnames are pending. | None locally. | Phase 1c. |

### Risks found during implementation (2026-11-12)

| ID | Risk | Impact | Follow-up |
| --- | --- | --- | --- |
| `RISK-006` | Consumers fetch federated types from the remote URLs at build time. In CI those are the staging hosts, which don't serve the new remotes yet. | A fresh-clone `typecheck` in core, auth, or payment fails until types are available. | Phase 1e: publish types with the remote artifacts, build the remotes first, or set `remoteTypeUrls`. |
| `RISK-007` | Host real types reveal 9 pre-existing type errors (`use-brand`, `use-auth`, `ErrorBoundary`, `appbar`, `sidebar`). | The host keeps `any` stubs until they are fixed. | A follow-up task fixes them and deletes `smooth-web-app/src/types/mfe.d.ts`. |
| `RISK-008` | The auth MFE navigates to `/dashboard` after login; that host route was removed under `ADEC-001`. | Post-login navigation has no matching host route. | A human decides the post-login landing route. |

## Open decisions

| ID | Decision | Affected records | Impact | Status |
| --- | --- | --- | --- | --- |
| `ADEC-001` | If `DEC-002` is accepted: which host routes do the auth, core, and payment apps mount on? Today, menu is mounted for both the unauthenticated landing and an authenticated route. Proposal: unauthenticated → `smooth_auth/AuthApp`; authenticated `/*` → `smooth_core/CoreApp`; payment mounted at the path where settings is mounted today. Dashboard is removed. | `IMP-006` | Without an answer, the host routes cannot be finalised. | `RESOLVED 2026-11-12: proposal accepted` |
| `DEC-001`–`DEC-003` | See `SPEC.md`. | — | — | `RESOLVED 2026-11-12` |

## Human plan approval

- **Decision:** `APPROVED`
- **Reviewer:** babak@smooth.tech
- **Date:** 2026-11-12
- **Comments:** All five documents were approved together, with DEC-001 A, DEC-002 Yes, ADEC-001 proposal, and DEC-003 Yes. Jira was skipped for this tooling feature at the user's request; traceability is through this spec and the developer reports.
