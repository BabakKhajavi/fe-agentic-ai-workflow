# Developer Report: rspack-mf2-migration / TASK-001 – TASK-005

This is an execution report for the review handoff, not an additional
approved SDD artifact.

## Result

- **Feature:** `rspack-mf2-migration`
- **Jira issue:** none. Jira was skipped by the user's decision on
  2026-11-12; traceability is through this spec.
- **Plan tasks:** `TASK-001` to `TASK-005`, run as one session at the user's
  request.
- **Status:** `READY_FOR_REVIEW`. Deviations are listed below; none blocks
  review.
- **Branch:** `rspack-mf2-migration` in all six repositories, created from
  `development`.
- **New repository:** `smooth-web-build-config`, on branch `development`,
  with no remote yet.
- **Committed:** nothing. All changes are in the working trees.
- **Date:** 2026-11-12. Node v22.22.0 (nvm), `@rspack/core` 2.3.2,
  `@module-federation/enhanced` 2.9.3.

## Traceability delivered

| Requirements | Acceptance criteria | Contracts | Technical requirements |
| --- | --- | --- | --- |
| `REQ-001`, `REQ-002`, `REQ-009` | `AC-001` | `CON-001`, `CON-002`, `CON-007` | `TECH-153`, `TECH-156` |
| `REQ-003`, `REQ-007` | `AC-005` | `CON-001`, `CON-002` | `TECH-154`, `TECH-155` |
| `REQ-004` | `AC-002`, `AC-006` | `CON-003`, `CON-004` | `TECH-149`, `TECH-151` |
| `REQ-005`, `REQ-006` | `AC-002`, `AC-004` | `CON-005`, `CON-006` | `TECH-154` |
| `REQ-008` | `AC-007` | `CON-001`, `CON-002` | `TECH-002` |
| `REQ-010` | `AC-008` | `CON-007` | `TECH-157` |
| `REQ-011` | `AC-009` | — | — (done with the spec) |
| `REQ-012` | `AC-008` (partial; see Deviations) | `CON-008` | — |

## File changes

| Action | Path | Reason |
| --- | --- | --- |
| Created | `smooth-web-build-config/{package.json,.nvmrc,.gitignore,README.md}` | `TASK-001` / `CON-001` |
| Created | `smooth-web-build-config/src/index.js` | `createSmoothRspackConfig`, `getAppEnv`: SWC, React Refresh, HTML, CSS, MF 2 (`manifest`, `shareStrategy: 'loaded-first'`, runtime plugin, `dts`), dev server. Sets `lazyCompilation: false` and `dev: isDev && isServe` (see Deviations). |
| Created | `smooth-web-build-config/src/shared.js` | `SHARED_SINGLETONS`, unchanged values; `import: false` for React in remotes (`CON-003`) |
| Created | `smooth-web-build-config/src/remotes.js` | Remote registry, `resolveRemoteEntry`, `remotesFor` (`CON-004`, `CON-006`) |
| Created | `smooth-web-build-config/runtime/remote-resolver.js` | MF 2 runtime plugin: `beforeRequest` resolution and `errorLoadRemote` logging (`CON-006`) |
| Created | `smooth-web-build-config/test/{remotes,shared,remote-resolver}.test.js` | 16 unit tests (`AC-004`, `AC-005`, `CON-003`) |
| Created | `*/rspack.config.js`, `*/.nvmrc`, `*/.npmrc` (`install-links=true`) | `CON-002`, `CON-007`, `RISK-001` |
| Deleted | `*/webpack.config.js`, `*/.babelrc`, `smooth-web-app/scripts/utils/*`, `smooth-web-{core,auth,payment}-mfe/src/scripts/utils/*` | `IMP-003`, `IMP-004` |
| Modified | `*/package.json`, `*/package-lock.json` | Webpack, Babel, and loaders removed; Rspack 2, MF 2, and the shared config added; `start`/`build`/`typecheck` scripts; `engines` (`IMP-002`) |
| Modified | `*/tsconfig.json` | `isolatedModules: true`; `paths "*": ["./@mf-types/*"]` in consumers (`CON-007`, `CON-008`) |
| Modified | `*/.gitignore` | `@mf-types/`, `@mf-types.zip`, `@mf-types.d.ts`, `.mf/` |
| Modified | `smooth-web-auth-mfe`: `package.json` name; `public/index.html`; `src/bootstrap.tsx`; `src/modules/{Placeholder,layout}`; `src/providers/menu-provider.tsx` → `auth-provider.tsx`; `src/routes/MenuRoutes.tsx` → `AuthRoutes.tsx` | `DEC-002` identity: `smooth_auth`, port 9001, `./AuthApp` (`IMP-007`) |
| Modified | `smooth-web-payment-mfe`: the same pattern, `Settings*` → `Payment*`. Also fixes the pre-existing mismatch where `bootstrap.tsx` mounted on `#dashboard-root` while `index.html` had `#auth-root`; both are now `#payment-root`. | `DEC-002` identity: `smooth_payment`, port 9005, `./PaymentApp` (`IMP-007`) |
| Modified | `smooth-web-app/src/routes/authenticated-routes.tsx` | `ADEC-001`: index → `smooth_core/CoreApp`; `/menu/*` → `smooth_core/CoreApp`; `/settings/*` → `smooth_payment/PaymentApp`; dashboard routes removed; placeholders unchanged |
| Reverted to HEAD | `smooth-web-app/src/routes/unauthenticated-routes.tsx` | Your uncommitted temporary switch to `smooth_menu/MenuApp` is undone, as its own TODO asked ("swap back to `smooth_auth/AuthApp` once the auth MFE exists"). It is now byte-identical to the committed version. |
| Modified | `smooth-web-app/src/types/mfe.d.ts` | Stubs updated to the target remotes and kept temporarily, with an explanatory header (see Deviations) |
| Deleted | `smooth-web-{core,auth,payment}-mfe/src/types/mfe.d.ts` | Replaced by federated types (`CON-008`) |
| Modified | `smooth-web-data-mfe/src/api/auth/auth-api-slice.ts`, `src/api/brand/brand-api-slice.ts` | Added `export` to 6 payload/param interfaces, because declaration emit for federated types failed with TS4023. Types only; no behaviour change. |

## Tests

| Test | Coverage | Result |
| --- | --- | --- |
| `smooth-web-build-config`: `node --test` (16 tests) | Registry and ports; environment URLs; manifest override (`AC-004`); no `Date.now()`; unknown remote or environment; `remotesFor`; singletons unchanged (`CON-003`); remotes `import: false`; plugin `beforeRequest` override; resolve-once; `errorLoadRemote` for both hook shapes | Passed, 16/16 |
| Browser, host `localhost:9000` | `AC-002`: host renders the auth remote; `loadRemote` succeeds for core, payment, data/Store, ui/atoms and ui/molecules; one React (`host@19.2.3`) across all 6 federation instances; no console errors | Passed |
| Browser, override through the release manifest | `AC-004` end to end: a temporary `/smooth-web-app/manifest.json` with `{"core":"9.9.9"}` made the resolver point core at `<origin>/smooth-web-core-mfe/9.9.9/mf-manifest.json` while the others kept their default URLs; the expected 404 was logged by `errorLoadRemote`. The temporary file was removed afterwards. | Passed |
| Browser, HMR | `AC-003`: editing the host component `async-boundary.tsx` applied a hot update ("Updated modules … App is up to date") and the page state survived (marker kept) | Passed, after a fix (see Deviations) |
| Negative federated-type check | `AC-008` / `CON-008`: with real types, assigning `store.getState` to `number` fails with TS2322 | Passed (the types are real) |

## Verification

All on Node 22.22.0. Final run on 2026-11-12.

| Command | Result | Evidence or failure |
| --- | --- | --- |
| `npm run typecheck` (×6) | Passed | 0 errors in all six. The host still uses stubs (see Deviations). |
| lint | Unavailable | No ESLint is configured in any repository (known gap, out of scope). |
| `node --test` (build-config) | Passed | 16/16 |
| `NODE_ENV=staging rspack build` (×6) | Passed | Exit 0 in 1–10 s each. Every repository emits `mf-manifest.json`; remotes emit `remoteEntry.js`; design emits a CSS chunk linked to its exposes in the manifest. |
| `NODE_ENV=development rspack build` (×6) | Passed | Exit 0 in 0–9 s (it hung before the `dev` fix) |
| `NODE_ENV=release APP_VERSION=1.0.0 GIT_SHA=abc123` host build | Passed | `publicPath` `/smooth-web-app/1.0.0/`; `index.html` has `__SMOOTH_BUILD_VERSION__="1.0.0"`, `__SMOOTH_GIT_SHA__="abc123"`, `apiUrl:""`, `brandAssetBaseUrl:"https://release.app.smooth.tech"` (`AC-007`) |
| `NODE_ENV=release APP_VERSION=abc` host build | Passed (fails as intended) | Exit 1: "APP_VERSION must be a semantic version for release builds" |
| Exposes and shared diff against `git show HEAD:webpack.config.js` | Passed | data's 7 exposes and design's 4 exposes are identical; the singleton map is identical (unit-tested) (`AC-006`) |
| Webpack and Babel leftovers | Passed | 0 leftover files; no Webpack, Babel, or loader packages in any `package.json` |

## POC shortcuts preserved

- `POC-001` / `RISK-001`: the shared config is consumed through
  `file:../smooth-web-build-config` with `install-links=true`. CI cannot
  install it until the Bitbucket repository exists (Phase 1a); then switch
  to a `git+ssh` tag.
- `POC-002`: environment base URLs are still the Web One
  `*.app.smooth.tech` hosts.
- `POC-003`: a failed remote load is logged and rethrown; there is no
  branded fallback.

## Deviations and remaining risks

1. **The contract changed: `shareStrategy` is `loaded-first`, not
   `version-first` (`CON-003`, amended in `CONTRACTS.md`).**
   - With `version-first`, the MF 2 runtime loads every remote's entry
     eagerly, before the release manifest is fetched. That would break
     per-client pins and download all remotes at startup.
   - The resolver hook moved from `beforeInit` to `beforeRequest` for the
     same reason.
2. **`lazyCompilation: false` was added to the shared config.** Rspack 2
   lazy-compiles dynamic imports in dev by default, which left the
   `import('./bootstrap')` boundary unrendered and broke hot updates.
3. **`dev: isDev && isServe` was added.** The MF dev features start a
   long-lived worker. Without this, `NODE_ENV=development rspack build`
   never exited; it was observed hanging for over 10 minutes.
4. **The host applies its production `optimization` block only outside
   development.** Under Webpack it applied in dev too. `concatenateModules`,
   deterministic IDs, and the single runtime chunk made React Refresh fall
   back to full reloads ("Aborted because … is not accepted"). Production
   output is unchanged.
5. **The `PLAN.md` `TASK-003` verification "each standalone page renders"
   was wrong.**
   - Remotes keep `import: false` for React (`REQ-004`), so a remote opened
     on its own port has no React. It shows MF `RUNTIME-012` and a blank
     page.
   - The Webpack configs had the same setting, so this is not a regression.
     Remotes are verified through the host instead.
   - If standalone remote pages are wanted, that is a separate decision:
     for example, letting a remote bundle React only when served on its
     own.
6. **Cross-federation hot updates are not supported.** Editing a remote's
   file does not update the host page. The host itself hot-updates, and so
   does each remote within its own dev server. The same was true under
   Webpack MF 1.
7. **`REQ-012` (federated types) is only partly delivered.**
   - Done:
     - all 5 remotes generate types;
     - core, auth, and payment type-check against the real data and design
       types;
     - the host receives all 5 type sets.
   - Not done: switching the host to the real types surfaced **9
     pre-existing type mismatches** that the `any` stubs had hidden:
     - `use-brand.ts`: the host's own `Brand` type differs from data-mfe's
       `Brand` (×5);
     - `use-auth.ts`: no matching overload (×1);
     - `ErrorBoundary.tsx`: `ReactNode` → `string` (×1);
     - `appbar.tsx` / `sidebar.tsx`: `null` → `undefined` (×2).
   - Fixing them changes host application code outside this tooling
     feature. So the host keeps its stubs for now, with an explanatory
     header in `src/types/mfe.d.ts`.
   - **Follow-up:** fix the 9 errors and delete that file.
8. **CI type availability is a new risk (`RISK-006`).** Consumers fetch
   remote types during build from the remote URLs. In CI those are the
   staging hosts, which don't serve the new remotes yet, so a fresh-clone
   `typecheck` in core, auth, or payment will fail with "Cannot find module
   'smooth_data/…'". This must be solved in Phase 1e. Options:
   - publish types with the remote builds;
   - build the remotes first in the pipeline;
   - point `remoteTypeUrls` at the artifact bucket.
9. **`webpack` is installed only as an auto-installed peer of
   `@module-federation/enhanced` (`RISK-003`).** It is never imported.
10. **There is a local port clash on 9002.** An unrelated webpack-dev-server
    from `~/Documents/02-regular-portal/sc-merchant-portal-react`, running
    since 2026-09-30, holds `127.0.0.1:9002`. `localhost` resolves to `::1`
    and reaches design-mfe, but anything that uses `127.0.0.1:9002` gets the
    portal. It was left running.
11. **One of my own errors was caught and fixed in-session.** A
    `git checkout` used to revert a test edit briefly restored auth's
    placeholder text to "Menu MFE placeholder". It was re-applied and
    verified.

## Review focus

- `smooth-web-build-config/src/index.js` and `runtime/remote-resolver.js`:
  this is the policy every repository now inherits. Check:
  - `shareStrategy: 'loaded-first'`, and the resolve-once-on-first-request
    behaviour;
  - `lazyCompilation: false`;
  - `dev: isDev && isServe`.
- The host route mapping (`ADEC-001`):
  - `/` and `/menu/*` → core;
  - `/settings/*` → payment;
  - `/dashboard` removed. The auth MFE's `use-auth.ts` still navigates to
    `/dashboard` after login, and that path no longer has a host route.
    This needs a decision.
- Whether to accept the temporary host type stubs (Deviation 7) and the CI
  types risk (Deviation 8) for Phase 1e.
