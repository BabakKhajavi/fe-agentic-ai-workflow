# Requirements: Rspack and Module Federation 2 migration with a shared build config

## Document control

| Field | Value |
| --- | --- |
| Feature slug | `rspack-mf2-migration` |
| Status | `READY_FOR_SPEC` |
| Intake date | `2026-11-11` |
| Prepared by | `Intake Agent` (run inline by Claude in the user session) |
| Human intake approval | `APPROVED` (2026-11-12) |
| Approved by | babak@smooth.tech |
| Approval date | 2026-11-12 |

## Sources

| Source ID | Source type | Title or summary | Reference | Version | Last updated | Sections or notes used |
| --- | --- | --- | --- | --- | --- | --- |
| `SRC-001` | `Developer-provided` | User decision: move every MFE to Rspack and adopt Module Federation 2 now. | User session, 2026-11-11 | — | 2026-11-11 | "change the whole MF to use Rspack … if Module Federation 2 is something we should move to … do it right now" |
| `SRC-002` | `Developer-provided` | User decision: use raw Rspack, not Rsbuild, for native MF, flexibility, optimization, and customization. Implement it with a shared config. | User session, 2026-11-11 | — | 2026-11-11 | "I am still interested in Rspack …", "add it to plan and let's implement that along with a shared config" |
| `SRC-003` | `Developer-provided` | Web Two execution plan, Changes 1–2: the six-repository target, per-client release pins, and the bootstrap loader (AD-2.7). | `context/EXECUTION_PLAN.md`; Confluence page `3204939777` | draft | 2026-11-08 | Change 1, Change 2 AD-2.7, Phase 1 |

## Business objective

Move all six `smooth-web` repositories from Webpack 5 to Rspack 2 with Module
Federation 2 now, while the repositories are still early scaffolds and before
the Phase 1 staging pipeline is built. This gives:

- faster builds and dev reloads;
- a federation runtime that can load each client's pinned release at
  runtime (AD-2.7);
- one shared, versioned build config instead of six drifting copies.

The open Vite-vs-Webpack baseline conflict is resolved in favour of Rspack
(`SRC-001`, `SRC-002`).

## Users and stakeholders

| Actor | Need or responsibility | Source |
| --- | --- | --- |
| Frontend developers | Fast local builds and HMR, and one consistent config across repositories. | `SRC-002` |
| Release / DevOps owner | Build artifacts that the Phase 1 staging deploy and the Phase 3 release manifests can rely on. | `SRC-003` |
| Technical lead | One place to change the federation, shared-singleton, and build policy. | `SRC-002` |

## Scope

### In scope

- The build tooling of all six repositories: `smooth-web-app`,
  `smooth-web-data-mfe`, `smooth-web-design-mfe`, `smooth-web-core-mfe`,
  `smooth-web-payment-mfe`, and `smooth-web-auth-mfe`.
- A shared build-config module that all six consume.
- Replacing the hand-written `createPromiseRemote` loader with the Module
  Federation 2 runtime.
- Updating the workflow context documents (architecture, standards,
  technical requirements).

### Out of scope

- Runtime white-labeling and slug routing (Phase 2).
- Per-client release manifests and promotion (Phase 3).
- Bitbucket pipelines and AWS (Phase 1a–1d). Only the build commands the
  pipelines call are changed here.
- Jest, ESLint, and Storybook setup. No tests or lint config exist yet;
  they are separate features.
- Feature behaviour changes of any kind.

## Functional requirements

### REQ-001 — Rspack builds every repository
- **Statement:** Every repository must build (`npm run build`) and serve
  (`npm start`) with Rspack 2 (`@rspack/core`, `@rspack/cli`,
  `@rspack/dev-server`) instead of Webpack.
- **Source:** `SRC-001`, `SRC-002`
- **Acceptance evidence:** a successful production build and dev server
  start in all six repositories. No `webpack`, `webpack-cli`,
  `webpack-dev-server`, `babel-loader`, `html-webpack-plugin`,
  `@pmmmwh/react-refresh-webpack-plugin`, `@types/webpack`, `style-loader`,
  or `css-loader` remains in any `package.json`.

### REQ-002 — SWC replaces Babel for compilation
- **Statement:** TS, TSX, and JS must be compiled with Rspack's built-in SWC
  loader, using the React automatic runtime and React Refresh in
  development.
- **Source:** `SRC-002`
- **Acceptance evidence:** no `babel-loader` in the build; HMR keeps
  component state in the dev server.

### REQ-003 — Module Federation 2 plugin, one pinned version
- **Statement:** Every repository must use the `ModuleFederationPlugin` from
  `@module-federation/enhanced/rspack`, at one exact version shared by all
  six repositories.
- **Source:** `SRC-001`
- **Acceptance evidence:** the same exact version appears in all six lock
  files.

### REQ-004 — Federation contracts are preserved
- **Statement:** The following must stay as they are, except where `DEC-002`
  changes the remote set:
  - federation names;
  - the `remoteEntry.js` filename;
  - every existing `exposes` key;
  - every shared-singleton entry, including React and react-dom
    `strictVersion` `19.2.x` and `import: false` in remotes.
- **Source:** `SRC-003`, `TECH-148`–`TECH-151`
- **Acceptance evidence:** a diff of the exposes and shared maps before and
  after migration.

### REQ-005 — Each build emits a federation manifest
- **Statement:** Every remote build must emit `mf-manifest.json` next to
  `remoteEntry.js`.
- **Source:** `SRC-001`, `SRC-003` (AD-2.7)
- **Acceptance evidence:** the file is present in every remote's `dist/`.

### REQ-006 — The MF runtime resolves remote URLs
- **Statement:** The host and every remote that consumes other remotes must
  resolve remote entry URLs through a Module Federation runtime plugin, not
  through `createPromiseRemote`. Resolution order stays the same:
  1. a version in `window.__SMOOTH_RELEASE_MANIFEST__` gives
     `<origin>/<repo>/<version>/`;
  2. otherwise, the environment base URL is used;
  3. the `?v=<__SMOOTH_BUILD_VERSION__>` cache-busting query is kept.
- **Source:** `SRC-001`, `SRC-003` (AD-2.7)
- **Acceptance evidence:**
  - `createPromiseRemote.js` is deleted from every repository.
  - The host loads its remotes locally.
  - A manifest override, simulated locally, changes the requested URL.

### REQ-007 — One shared build config
- **Statement:** All six repositories must build from one shared config
  module. It provides:
  - the base Rspack config;
  - the shared-singleton map;
  - the remote registry (names, ports, and environment base URLs);
  - the remote-resolver runtime plugin.

  Each repository's `rspack.config.js` declares only its repository-specific
  values: name, kind, port, entry, exposes, remotes, HTML, and defines.
- **Source:** `SRC-002`
- **Acceptance evidence:** each repository config is ≤ 40 lines, and the
  shared-singleton map exists in exactly one place.

### REQ-008 — Dev and production behaviour parity
- **Statement:** The following must behave as they do under Webpack:
  - dev ports;
  - CORS headers on remotes;
  - `historyApiFallback`;
  - output `publicPath` rules (host `/smooth-web-app/` or
    `/smooth-web-app/<APP_VERSION>/`; remotes `auto`);
  - content-hashed filenames;
  - source maps;
  - every `DefinePlugin` variable;
  - the host HTML template parameters;
  - the semantic-version guard on release builds.
- **Source:** `SRC-003`
- **Acceptance evidence:**
  - The host HTML output contains the same injected `window.__SMOOTH_*`
    values.
  - A release build with an invalid `APP_VERSION` fails.

### REQ-009 — Node 22 baseline
- **Statement:** Every repository must declare Node `>=22.12` (Rspack 2
  requires `^20.19 || >=22.12`) in `engines` and in `.nvmrc`.
- **Source:** dependency constraint (`DEP-001`)
- **Acceptance evidence:** `.nvmrc` and `engines` are present. Builds run on
  Node 22 locally and in the `node:22` pipeline image.

### REQ-010 — Type checking stays a separate gate
- **Statement:** Every repository must have a `typecheck` script
  (`tsc --noEmit`) and `isolatedModules: true` in `tsconfig.json`, because
  SWC does not type-check.
- **Source:** `TECH-*` quality gates
- **Acceptance evidence:** `npm run typecheck` passes in all six
  repositories. The baseline on 2026-11-11 is 0 errors with or without
  `isolatedModules`.

### REQ-011 — Workflow documents reflect the new baseline
- **Statement:** `ARCHITECTURE.md`, `ENGINEERING_STANDARDS.md`,
  `TECHNICAL_REQUIREMENTS.md`, the agent docs, and the templates must name
  Rspack + Module Federation 2 and the shared config, and must resolve the
  Vite-vs-Webpack conflict.
- **Source:** user request, 2026-11-11 ("also update Standards and
  Architecture")
- **Acceptance evidence:** no workflow document still prescribes Vite or
  Webpack for `smooth-web`.

### REQ-012 — Typed remote modules (conditional on `DEC-003`)
- **Statement:** Remotes generate federated type declarations, and
  consumers use them in place of the `declare module 'smooth_*/…'` stubs
  (currently `any`).
- **Source:** `SRC-001` (MF 2 benefits)
- **Acceptance evidence:** the host's `tsc` resolves real types for
  `smooth_data/*` and `smooth_ui/*` imports.

## Dependencies and constraints

| ID | Type | Description | Source | Affected requirements |
| --- | --- | --- | --- | --- |
| `DEP-001` | Constraint | `@rspack/core` 2.3.2 and `@rspack/dev-server` 2.2.2 require Node `^20.19.0 \|\| >=22.12.0`. Local default Node is 16.19.0; Node 22.22.0 is available through nvm. | npm metadata, 2026-11-11 | `REQ-001`, `REQ-009` |
| `DEP-002` | Constraint | `@rspack/core` 2 is ESM-only. All six repositories are already `"type": "module"`. | npm package inspection | `REQ-001` |
| `DEP-003` | Dependency | Versions verified 2026-11-11: `@rspack/core` 2.3.2, `@rspack/cli` 2.3.2, `@rspack/dev-server` 2.2.2, `@rspack/plugin-react-refresh` 2.0.2, `@module-federation/enhanced` 2.9.3. | npm registry | `REQ-001`, `REQ-003` |
| `DEP-004` | Constraint | The installed library versions (React 19.2.3, MUI 7.3.6, RTK 2.11.2, react-redux 9.2.0, TS ^5.9.3) and the shared `requiredVersion` ranges must not change as part of this migration. | repository `package.json` files | `REQ-004` |

## Assumptions

| ID | Assumption | Reason it is not confirmed | Affected requirements | Resolution needed |
| --- | --- | --- | --- | --- |
| `ASM-001` | Staging, release, and production base URLs for the new remotes (`smooth-web-core-mfe`, `smooth-web-payment-mfe`) follow the existing `https://<env-host>/<repo>` pattern until the Web Two hostnames are provided. | Hostnames are pending (EXECUTION_PLAN Q2.6b). | `REQ-006`, `REQ-007` | No. They are config values, replaced in Phase 1c. |
| `ASM-002` | Remote entries point at `mf-manifest.json`, not `remoteEntry.js`. Both are emitted. | Recommended for MF 2 snapshot and preload; not separately confirmed. | `REQ-005`, `REQ-006` | No |

## Ambiguities, conflicts, and missing information

| ID | Classification | Finding | Sources | Affected requirements | Impact |
| --- | --- | --- | --- | --- | --- |
| `GAP-001` | Conflict (pre-existing) | `TECH-148` freezes React 19.3.0, TypeScript 7.0.2, MUI 9.4.0, RTK 2.12.0, and react-redux 9.3.0. The repositories run React 19.2.3, TS ^5.9.3, MUI 7.3.6, RTK 2.11.2, and react-redux 9.2.0, and their shared config pins `19.2.x` / `7.x.x`. | `TECHNICAL_REQUIREMENTS.md` `TECH-148`; repository `package.json` files | `REQ-004` | Not resolved by this feature. The migration keeps the installed versions unchanged, so it neither widens nor closes the gap. A human must decide separately whether the repositories move to `TECH-148` or `TECH-148` is corrected. |

## Clarification questions

See `SPEC.md` `DEC-001`–`DEC-003`. They are raised as decisions so that
approval can be given in one review.

## Applicable global technical requirements

| Technical requirement | Relevance |
| --- | --- |
| `TECH-149`, `TECH-151` | The shared singletons must be preserved exactly. `TECH-148` versions: see `GAP-001`. |
| `TECH-153`–`TECH-157` (new) | The Rspack, MF 2, shared-config, Node, and typecheck rules this feature introduces. |

## Traceability summary

| Local ID | Source | Verification evidence | Open questions |
| --- | --- | --- | --- |
| `REQ-001` | `SRC-001`, `SRC-002` | Builds and dev servers run in six repositories. | None |
| `REQ-002` | `SRC-002` | HMR check. | None |
| `REQ-003` | `SRC-001` | Lock files. | None |
| `REQ-004` | `SRC-003` | Exposes/shared diff. | `DEC-002` |
| `REQ-005` | `SRC-001` | `dist/mf-manifest.json`. | None |
| `REQ-006` | `SRC-001`, `SRC-003` | Host loads remotes; manifest override. | None |
| `REQ-007` | `SRC-002` | Config size; single shared map. | `DEC-001` |
| `REQ-008` | `SRC-003` | HTML and release-guard checks. | None |
| `REQ-009` | `DEP-001` | `.nvmrc` and `engines`. | None |
| `REQ-010` | `TECH-*` | `npm run typecheck`. | None |
| `REQ-011` | User request | Document grep. | None |
| `REQ-012` | `SRC-001` | Host `tsc` with real types. | `DEC-003` |

## Human intake approval

- **Decision:** `APPROVED`
- **Reviewer:** babak@smooth.tech
- **Date:** 2026-11-12
- **Comments:** All five documents were approved together, with DEC-001 A, DEC-002 Yes, ADEC-001 proposal, and DEC-003 Yes. Jira was skipped for this tooling feature at the user's request; traceability is through this spec and the developer reports.
