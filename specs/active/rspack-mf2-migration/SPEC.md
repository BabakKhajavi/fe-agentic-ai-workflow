# Specification: Rspack and Module Federation 2 migration with a shared build config

## Document control

| Field | Value |
| --- | --- |
| Feature slug | `rspack-mf2-migration` |
| Status | `READY_FOR_PLANNING` |
| Requirements baseline | `REQUIREMENTS.md` 2026-11-11 |
| Human spec approval | `APPROVED` (2026-11-12) |

## Objective

All six repositories build and serve with Rspack 2 and Module Federation 2,
from one shared build config. The running application behaves exactly as it
did under Webpack. Remote URLs are resolved by a Module Federation runtime
plugin, which Phase 3's per-client release loader will build on.

## Scope

### In scope

The same as `REQUIREMENTS.md`: `REQ-001` to `REQ-011`, plus `REQ-012` if
`DEC-003` is accepted.

### Out of scope

- White-labeling.
- Per-client manifests.
- Pipelines and AWS.
- Jest, ESLint, and Storybook.
- Feature behaviour.

## Developer flow

1. A developer runs `nvm use` (Node 22), `npm install`, then `npm start` in
   `smooth-web-app` and the remotes it needs.
2. Each dev server starts on its fixed port:

   | Repository | Port |
   | --- | --- |
   | app | 9000 |
   | auth | 9001 |
   | design | 9002 |
   | data | 9003 |
   | payment | 9005 |
   | core | 9007 |

3. The host loads each remote through the MF 2 runtime from
   `http://localhost:<port>/mf-manifest.json`, and the application renders.
4. Editing a component hot-reloads it and keeps its state.
5. `npm run build` produces `dist/` with content-hashed files,
   `remoteEntry.js`, and `mf-manifest.json` (remotes), or `index.html` (the
   host and any standalone remote).
6. `npm run typecheck` runs `tsc --noEmit`.

## Behavioural states

| State | Expected behaviour | Related requirements |
| --- | --- | --- |
| Remote available | Its modules load and render as before. | `REQ-004`, `REQ-006` |
| Release manifest present | The runtime plugin rewrites the remote entry to `<origin>/<repo>/<version>/mf-manifest.json?v=<build>`. | `REQ-006` |
| No manifest version | The environment base URL from the remote registry is used. | `REQ-006` |
| Remote fails to load | The runtime plugin logs `Failed to load remote <name> from <url>` and rethrows, so the existing error boundary and route fallback behave as before. A branded fallback is Phase 2/3 work. | `REQ-006` |
| Release build with a bad `APP_VERSION` | The build fails with the existing error message. | `REQ-008` |

## Acceptance criteria

### AC-001 — Every repository builds with Rspack
- **Given:** Node 22 and a clean install
- **When:** `npm run build` runs in each of the six repositories
- **Then:**
  - it exits with code 0;
  - `dist/` contains content-hashed assets;
  - each remote has `remoteEntry.js` and `mf-manifest.json`;
  - no Webpack or Babel build dependency remains.
- **Covers:** `REQ-001`, `REQ-002`, `REQ-005`, `REQ-009`
- **Verification:** Automated (build command); developer report evidence

### AC-002 — Local federation works end to end
- **Given:** the host and its remotes are running in dev mode
- **When:** the host opens at `http://localhost:9000`
- **Then:**
  - every declared remote loads through the MF 2 runtime;
  - there are no federation errors in the console;
  - only one React instance is loaded (singleton preserved).
- **Covers:** `REQ-004`, `REQ-006`, `REQ-008`
- **Verification:** Manual POC demonstration, with a console check of
  `__FEDERATION__` share scopes

### AC-003 — HMR keeps component state
- **Given:** the dev server is running
- **When:** a component file in the host or a remote is edited
- **Then:** the change appears without a full reload, and component state is
  kept
- **Covers:** `REQ-002`
- **Verification:** Manual

### AC-004 — Manifest override changes the remote URL
- **Given:** `window.__SMOOTH_RELEASE_MANIFEST__ = { data: '1.2.3' }` is set
  before bootstrap
- **When:** the host loads `smooth_data`
- **Then:** the requested URL is
  `<origin>/smooth-web-data-mfe/1.2.3/mf-manifest.json?v=<build>`
- **Covers:** `REQ-006`
- **Verification:** Automated unit test of the resolver function in the
  shared config (`node --test`)

### AC-005 — Shared config is the single source of truth
- **Given:** the six repository configs
- **When:** they are reviewed
- **Then:**
  - each `rspack.config.js` contains only repository-specific values;
  - the shared-singleton map and the remote registry exist only in the
    shared config;
  - every repository pins the same shared-config version.
- **Covers:** `REQ-003`, `REQ-007`
- **Verification:** Review plus a script check

### AC-006 — Contracts preserved
- **Given:** the pre-migration Webpack configs
- **When:** they are compared with the effective Rspack configs
- **Then:** the federation names, `remoteEntry.js`, the exposes keys,
  shared-singleton versions, and `strictVersion`/`import: false` settings
  are identical, apart from the remote-set change accepted under `DEC-002`
- **Covers:** `REQ-004`
- **Verification:** Review (diff in the developer report)

### AC-007 — Production output parity
- **Given:** `NODE_ENV=release APP_VERSION=1.0.0` (host) and
  `NODE_ENV=staging`
- **When:** the host is built
- **Then:**
  - `publicPath` is `/smooth-web-app/1.0.0/` (release) or `/smooth-web-app/`
    (staging);
  - `index.html` contains the same `window.__SMOOTH_*` template
    substitutions;
  - `APP_VERSION=abc` fails the release build.
- **Covers:** `REQ-008`
- **Verification:** Automated (build plus grep)

### AC-008 — Type checking passes
- **Given:** `isolatedModules: true`
- **When:** `npm run typecheck` runs in every repository
- **Then:** 0 errors
- **Covers:** `REQ-010`, `REQ-012` (if accepted)
- **Verification:** Automated

### AC-009 — Documents updated
- **Given:** the workflow context documents
- **When:** they are searched for Vite or Webpack prescriptions for
  `smooth-web`
- **Then:** none remain, apart from historical notes, and Rspack + MF 2 plus
  the shared config are documented
- **Covers:** `REQ-011`
- **Verification:** grep

## POC implementation assumptions

| ID | Assumption or shortcut | Reason | Demo effect | Production follow-up |
| --- | --- | --- | --- | --- |
| `POC-001` | The shared config is consumed through a local `file:` path until its Bitbucket repository exists (`DEC-001` A). | The Bitbucket repository is created in Phase 1a. | None locally; CI cannot install it yet. | Switch to a `git+ssh` tag pin in Phase 1a. |
| `POC-002` | Environment base URLs reuse the existing `*.app.smooth.tech` hosts. | Web Two hostnames are pending. | None locally. | Replace them in Phase 1c. |
| `POC-003` | A remote load failure rethrows with no branded fallback. | Fallback UX belongs to Phases 2–3. | Same as today. | Add a fallback runtime plugin later. |

## Applicable technical requirements

- `TECH-149`, `TECH-151`: the shared singletons are preserved. The installed versions stay unchanged; the `TECH-148` mismatch is `GAP-001` and is out of scope.
- `TECH-153`–`TECH-157` (new): Rspack 2, MF 2, the shared config, Node 22,
  and the typecheck gate.

## Open decisions

| ID | Decision needed | Options | Recommendation | Status |
| --- | --- | --- | --- | --- |
| `DEC-001` | Where the shared build config lives. | **A:** a new tooling repository `smooth-web-build-config` (package `@smooth/web-build-config`), consumed as a dependency pinned to a tag. **B:** an identical vendored copy in every repository, plus a drift check. | **A**: real single source, versioned upgrades. Locally `file:../smooth-web-build-config` until the Bitbucket repository exists. | `RESOLVED 2026-11-12: A` |
| `DEC-002` | Whether the new configs declare only the six target remotes. | **Yes:** the host gets `smooth_auth`, `smooth_ui`, `smooth_data`, `smooth_core`, `smooth_payment`; dashboard, settings, and menu are removed; auth and payment get their own identity and ports (`./AuthApp`, `./PaymentApp`); the host routes are updated to match. **No:** keep the current remote set and leave the cleanup for later. | **Yes**: the configs are rewritten anyway, and the host cannot run today because three of its remotes don't exist. This takes over the "step 0" cleanup the user had planned to do separately. | `RESOLVED 2026-11-12: Yes` |
| `DEC-003` | Whether to include MF 2 type generation and consumption (`REQ-012`) now. | **Yes** (last task) or **later**. | **Yes**: it replaces the `any`-typed `declare module` stubs, so remote APIs get real types. | `RESOLVED 2026-11-12: Yes, as TASK-005` |

## Traceability

| Requirement | Acceptance criteria | Contract references |
| --- | --- | --- |
| `REQ-001` | `AC-001` | `CON-001`, `CON-002` |
| `REQ-002` | `AC-001`, `AC-003` | `CON-001` |
| `REQ-003` | `AC-005` | `CON-001` |
| `REQ-004` | `AC-002`, `AC-006` | `CON-003`, `CON-004` |
| `REQ-005` | `AC-001` | `CON-005` |
| `REQ-006` | `AC-002`, `AC-004` | `CON-005`, `CON-006` |
| `REQ-007` | `AC-005` | `CON-001`, `CON-002` |
| `REQ-008` | `AC-002`, `AC-007` | `CON-001`, `CON-002` |
| `REQ-009` | `AC-001` | `CON-007` |
| `REQ-010` | `AC-008` | `CON-007` |
| `REQ-011` | `AC-009` | — |
| `REQ-012` | `AC-008` | `CON-008` |

## Human spec approval

- **Decision:** `APPROVED`
- **Reviewer:** babak@smooth.tech
- **Date:** 2026-11-12
- **Comments:** All five documents were approved together, with DEC-001 A, DEC-002 Yes, ADEC-001 proposal, and DEC-003 Yes. Jira was skipped for this tooling feature at the user's request; traceability is through this spec and the developer reports.

Approval authorizes architecture impact analysis and planning only.
