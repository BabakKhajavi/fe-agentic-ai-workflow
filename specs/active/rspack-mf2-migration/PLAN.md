# Implementation Plan: Rspack and Module Federation 2 migration with a shared build config

## Document control

| Field | Value |
| --- | --- |
| Feature slug | `rspack-mf2-migration` |
| Status | `READY_FOR_APPROVAL` |
| Based on | `REQUIREMENTS.md`, `SPEC.md`, `CONTRACTS.md`, `IMPACT.md` |
| Human plan approval | `APPROVED` (2026-11-12) |

## Implementation strategy

1. Build and unit-test the shared config package first.
2. Migrate the two pure remotes that everything depends on (data, design).
3. Migrate the three app remotes (core, auth, payment), including their
   identity fixes if `DEC-002` is accepted.
4. Migrate the host last and run the whole application locally.
5. Optionally add federated types.

Each task leaves its repository building, so the work can be reviewed one
task at a time. All commands run on Node 22 (`nvm use`).

The Execution Plan places this work in **Phase 1, step 1b "Build
tooling"**, before OIDC (1c), AWS (1d), and the staging deploy (1e).

## Ordered tasks

### TASK-001 — Shared build-config package
- **Outcome:** `smooth-web-build-config` exists locally, as a git repository
  with no remote. It exports `createSmoothRspackConfig`,
  `SHARED_SINGLETONS`, `REMOTES`, `resolveRemoteEntry`, `getAppEnv`, and the
  `runtime/remote-resolver` plugin. Its unit tests pass.
- **Depends on:** None
- **Expected files:** `Create: ../smooth-web/smooth-web-build-config/{package.json,.nvmrc,README.md,src/index.js,src/shared.js,src/remotes.js,runtime/remote-resolver.js,test/remotes.test.js,test/shared.test.js}`
- **Implementation notes:**
  - ESM. Pinned exact versions as peer dependencies.
  - The resolver is a pure function, so it can be unit-tested.
  - No `Date.now()`.
- **Requirements:** `REQ-003`, `REQ-006`, `REQ-007`, `TECH-153`–`TECH-157`
- **Acceptance criteria:** `AC-004`, `AC-005`
- **Contracts:** `CON-001`, `CON-003`, `CON-004`, `CON-006`
- **Verification:** `node --test` passes.
- **Suggested Jira type:** Task

### TASK-002 — Migrate `smooth-web-data-mfe` and `smooth-web-design-mfe`
- **Outcome:** Both build and serve with Rspack and emit
  `mf-manifest.json`. Their exposes and shared settings are unchanged.
- **Depends on:** `TASK-001`
- **Expected files:**
  - `Create: rspack.config.js, .nvmrc`
  - `Modify: package.json, package-lock.json, tsconfig.json`
  - `Delete: webpack.config.js, .babelrc`
- **Implementation notes:**
  - design-mfe uses `css: true`.
  - Keep the existing defines: data `process.env.API_URL`; design
    `process.env.TENANT_CONFIG_BASE_URL`.
- **Requirements:** `REQ-001`, `REQ-002`, `REQ-004`, `REQ-005`, `REQ-009`,
  `REQ-010`
- **Acceptance criteria:** `AC-001`, `AC-006`, `AC-008`
- **Contracts:** `CON-002`, `CON-003`, `CON-005`, `CON-007`
- **Verification:**
  - `npm run build` and `npm run typecheck` pass.
  - `dist/mf-manifest.json` is present.
  - The exposes diff is attached to the developer report.
- **Suggested Jira type:** Task

### TASK-003 — Migrate `smooth-web-core-mfe`, `smooth-web-auth-mfe`, `smooth-web-payment-mfe`
- **Outcome:** All three build and serve standalone with Rspack. They
  consume design and data through the MF 2 runtime.
- **Depends on:** `TASK-002`
- **Expected files:** the same pattern as `TASK-002`, plus `Delete:
  src/scripts/utils/*`. If `DEC-002` is accepted, the identity renames in
  `IMP-007` (auth: `smooth_auth`, port 9001, `./AuthApp`; payment:
  `smooth_payment`, port 9005, `./PaymentApp`).
- **Requirements:** `REQ-001`–`REQ-006`, `REQ-009`, `REQ-010`
- **Acceptance criteria:** `AC-001`, `AC-002` (standalone), `AC-006`, `AC-008`
- **Contracts:** `CON-002`, `CON-004`, `CON-005`, `CON-006`, `CON-007`
- **Verification:**
  - Builds and typecheck pass.
  - Each standalone page renders its placeholder with design and data
    loaded.
- **Suggested Jira type:** Task

### TASK-004 — Migrate the host `smooth-web-app`
- **Outcome:**
  - The host builds and serves with Rspack and loads all remotes through
    the runtime plugin.
  - The `index.html` output matches the current template.
  - The release-version guard and `publicPath` rules are kept.
  - Routes are aligned (if `DEC-002` and `ADEC-001` are accepted).
- **Depends on:** `TASK-003`
- **Expected files:**
  - `Create: rspack.config.js, .nvmrc`
  - `Modify: package.json, package-lock.json, tsconfig.json, src/routes/authenticated-routes.tsx, src/routes/unauthenticated-routes.tsx, src/types/mfe.d.ts`
  - `Delete: webpack.config.js, scripts/utils/createPromiseRemote.js, scripts/utils/getMfeBaseUrl.js`
- **Implementation notes:**
  - Keep the host's `optimization` settings unless `RISK-002` shows a
    conflict; record the outcome.
  - Keep the `BRAND_ASSET_BASE_URL` resolution logic in the host config.
- **Requirements:** `REQ-001`–`REQ-008`, `REQ-010`
- **Acceptance criteria:** `AC-001`, `AC-002`, `AC-003`, `AC-006`, `AC-007`,
  `AC-008`
- **Contracts:** `CON-001`–`CON-007`
- **Verification:**
  - Builds for `NODE_ENV=staging` and for `NODE_ENV=release
    APP_VERSION=1.0.0`, plus a negative test with `APP_VERSION=abc`.
  - A grep of `dist/index.html` for the `window.__SMOOTH_*` values.
  - Local run with all five remotes: no console federation errors, and one
    React instance.
- **Suggested Jira type:** Story

### TASK-005 — Federated types (only if `DEC-003` is accepted)
- **Outcome:** Remotes generate types, and consumers resolve real types for
  `smooth_data/*` and `smooth_ui/*`.
- **Depends on:** `TASK-004`
- **Expected files:**
  - `Modify: shared config (dts options), consumers' src/types/mfe.d.ts, .gitignore`
- **Requirements:** `REQ-012`
- **Acceptance criteria:** `AC-008`
- **Contracts:** `CON-008`
- **Verification:**
  - `npm run typecheck` passes in the host using generated types.
  - A deliberate wrong-type usage fails, to prove the types are real.
- **Suggested Jira type:** Task

## Final integration verification

1. Type checking: `npm run typecheck` in all six repositories.
2. Linting: not configured. This is recorded as a gap; ESLint is a separate
   feature.
3. Automated tests: `node --test` in `smooth-web-build-config`. The
   repositories have no test runner yet.
4. Production build: `npm run build` (Rspack) in all six repositories.
5. Demo: the host plus five remotes running locally, with HMR and the
   manifest-override check.

`AC-002` and `AC-003` are manual. Federation loading and HMR state need a
browser, and a Playwright setup does not exist yet.

## Traceability matrix

| Requirement | Acceptance criteria | Contracts | Impact | Tasks | Verification |
| --- | --- | --- | --- | --- | --- |
| `REQ-001` | `AC-001` | `CON-001`, `CON-002` | `IMP-002`, `IMP-003` | `TASK-002`–`TASK-004` | builds |
| `REQ-002` | `AC-001`, `AC-003` | `CON-001` | `IMP-001` | `TASK-001`–`TASK-004` | build + HMR |
| `REQ-003` | `AC-005` | `CON-001` | `IMP-001`, `IMP-002` | `TASK-001` | lock files |
| `REQ-004` | `AC-002`, `AC-006` | `CON-003`, `CON-004` | `IMP-006`, `IMP-007` | `TASK-002`–`TASK-004` | diff + demo |
| `REQ-005` | `AC-001` | `CON-005` | `IMP-001` | `TASK-002`, `TASK-003` | `dist/mf-manifest.json` |
| `REQ-006` | `AC-002`, `AC-004` | `CON-005`, `CON-006` | `IMP-001`, `IMP-004` | `TASK-001`, `TASK-003`, `TASK-004` | unit test + demo |
| `REQ-007` | `AC-005` | `CON-001`, `CON-002` | `IMP-001`, `IMP-003` | `TASK-001`–`TASK-004` | review |
| `REQ-008` | `AC-002`, `AC-007` | `CON-001`, `CON-002` | `IMP-003` | `TASK-004` | release build + grep |
| `REQ-009` | `AC-001` | `CON-007` | `IMP-005` | `TASK-002`–`TASK-004` | `.nvmrc` / `engines` |
| `REQ-010` | `AC-008` | `CON-007` | `IMP-005` | `TASK-002`–`TASK-004` | typecheck |
| `REQ-011` | `AC-009` | — | `IMP-010` | done with the spec | grep |
| `REQ-012` | `AC-008` | `CON-008` | `IMP-008` | `TASK-005` | typecheck with types |

## Recommended Jira breakdown

- **Epic:** Web Two: Rspack + Module Federation 2 build tooling
- **Child work:** one item per `TASK-*`.

Jira creation remains prohibited until this plan and `IMPACT.md` are
approved.

## Post-approval amendments (2026-11-12, during implementation)

Recorded as this workspace requires. Details and evidence are in
`DEVELOPER_REPORT.md`.

- **`TASK-003` verification:** "each standalone page renders its
  placeholder" is withdrawn.
  - Remotes keep `import: false` for React (`REQ-004`, unchanged from
    Webpack), so a remote opened on its own has no React (MF
    `RUNTIME-012`).
  - Remotes are verified through the host instead.
- **`TASK-004`:** the host applies its production `optimization` block only
  outside development, so React Refresh can accept updates. Production
  output is unchanged.
- **`TASK-005`:** delivered for remotes and for core, auth, and payment. The
  host keeps temporary stubs because real types surfaced 9 pre-existing
  type errors in host application code. Fixing them is a follow-up task.
- **`CON-003` / `CON-006`:** amended to `shareStrategy: 'loaded-first'` and
  `beforeRequest` (see `CONTRACTS.md`).

## Human plan approval

- **Decision:** `APPROVED`
- **Reviewer:** babak@smooth.tech
- **Date:** 2026-11-12
- **Comments:** All five documents were approved together, with DEC-001 A, DEC-002 Yes, ADEC-001 proposal, and DEC-003 Yes. Jira was skipped for this tooling feature at the user's request; traceability is through this spec and the developer reports.

Approval authorizes Jira creation and implementation according to this plan.
It does not authorize unplanned scope or production deployment.
