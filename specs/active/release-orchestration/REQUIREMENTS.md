# Requirements: Per-client release orchestration

| Field | Value |
| --- | --- |
| Feature slug | `release-orchestration` |
| Status | `READY_FOR_SPEC` |
| Intake date | 2026-11-16 |
| Human intake approval | `APPROVED` (2026-11-16) |

## Sources

| ID | Type | Reference |
| --- | --- | --- |
| `SRC-001` | Developer-provided | User decisions, 2026-11-12 to 2026-11-16: per-client deployment in **all** environments; per-client **versions** only in sandbox and production; pipelines live in `smooth-web-release`; package pipelines only build and publish |
| `SRC-002` | Execution plan | Change 2 (AD-2.1 path slug, AD-2.6 to AD-2.10), CI/CD plan D3 to D7 (bundles plus per-client history, git-held state, gates) |
| `SRC-003` | Repository | `smooth-web-release` (`clients.json`, 30 clients); app and assets buckets and distributions per environment; brand files at `/configs/<slug>-brand.json` (30/30 live on staging, 2026-11-16) |

## Functional requirements

- **REQ-001 Client targets:** every pipeline takes `CLIENTS` = `<slug>[,<slug>…]` | `ring:<ring>` | `all`.
  - Targets resolve from `clients.json`: `status: active` and the environment listed in `environments`.
  - An unknown slug fails the run. An inactive or disallowed slug in an explicit list also fails the run.
- **REQ-002 Staging deploy:** `deploy-staging CLIENTS=…` makes the selected clients live on staging, always on the **latest** build. There is no version choice.
- **REQ-003 Bundles:** `create-bundle` records an immutable set of six repository versions (`shell`, `auth`, `design`, `data`, `core`, `payment`).
  - Each version is given explicitly, or as `latest` (the highest version tag already published to sandbox).
  - Every artifact must exist in the sandbox bucket.
- **REQ-004 Promote:** `promote-sandbox` / `promote-production BUNDLE=… CLIENTS=…` serve that bundle to each selected client in that environment and record a new numbered client release.
- **REQ-005 Rollback:** `rollback-sandbox` / `rollback-production CLIENTS=… [TO=<release#>]` serve each client's previous release (or release `TO`) again. Nothing is rebuilt.
- **REQ-006 Sandbox before production:** a bundle can be promoted to production only if it was promoted to at least one client in sandbox.
- **REQ-007 Production gate:** production promote and rollback run in the Bitbucket `production` deployment environment, which is restricted to the deployment group.
- **REQ-008 Brand check:** a run fails before changing anything if a selected client has no `configs/<slug>-brand.json` in that environment's assets bucket.
- **REQ-009 Audit:** every deploy, promote, and rollback commits `state/<env>/<slug>.json` with who, when, which bundle or channel, and which pipeline run.
- **REQ-010 One invalidation:** each run issues one CloudFront invalidation covering all selected clients.
- **REQ-011 Dry run:** every pipeline supports `DRY_RUN=true`, which prints every AWS and git operation without performing it.
- **REQ-012 Registry activation:** set all 30 clients to `status: active` (their brand files exist, `SRC-003`).

## Gaps and assumptions

| ID | Finding | Handling |
| --- | --- | --- |
| `GAP-001` | `/<slug>/<route>` currently falls through to the one root `index.html` (S3 error document), so every client would get the same release. | A CloudFront Function rewrites navigation requests to `/<slug>/index.html` (`CON-004`). The user attaches it manually (AD-2.9). |
| `GAP-002` | The host fetches one global `/smooth-web-app/manifest.json`. | The host reads the release inlined in the client's entry page instead (`CON-003`). |
| `GAP-003` | The staging host deploy uses `sync --delete`, so copied client entry pages would reference deleted asset files. | The host staging pipeline re-copies the new `index.html` to every existing `<slug>/index.html`. |
| `GAP-004` | An unknown or undeployed slug still falls back to the root `index.html` with the default brand, instead of "store not found" (AD-2.6). | Out of scope; app-level handling comes in a later feature. |
| `ASM-001` | Sandbox and production API URLs come from the variables `API_URL_RELEASE` and `API_URL_PRODUCTION`, as in the existing host pipelines. | Required for sandbox and production promote. |
| `ASM-002` | The brand snapshot inlined for first paint (CI/CD D1) is deferred. Brand is still loaded at runtime. | Follow-up. |

## Human intake approval

- **Decision:** `APPROVED`
- **Reviewer:** babak@smooth.tech
- **Date:** 2026-11-16
- **Comments:** All five documents approved together ("Approve, build it").
