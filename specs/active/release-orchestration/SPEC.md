# Specification: Per-client release orchestration

| Field | Value |
| --- | --- |
| Feature slug | `release-orchestration` |
| Status | `READY_FOR_PLANNING` |
| Human spec approval | `APPROVED` (2026-11-16) |

## Responsibilities

| Package pipelines (6 app repositories) | `smooth-web-release` pipelines |
| --- | --- |
| Build and publish code: on `staging`, the latest build goes to `<repo>/`; on a `v*` tag, a versioned build goes to `<repo>/<version>/` in the sandbox bucket | Decide which client runs which code: `deploy-staging`, `create-bundle`, `promote-*`, `rollback-*` |
| The host's staging deploy also refreshes the existing staging client entry pages (`GAP-003`) | Writes client entry pages, bundles, and state; one invalidation per run |

## Flows

| Pipeline | Inputs | Steps |
| --- | --- | --- |
| `deploy-staging` | `CLIENTS` | 1. Resolve the targets. 2. Check each brand file exists. 3. Copy staging `index.html` → `<slug>/index.html`. 4. Invalidate. 5. Commit `state/staging/<slug>.json` with `{channel: latest}`. |
| `create-bundle` | `SHELL`, `AUTH`, `DESIGN`, `DATA`, `CORE`, `PAYMENT` (version or `latest`) | 1. Resolve `latest` from the sandbox bucket. 2. Check every artifact exists. 3. Check React and react-dom versions match across the federation manifests. 4. Write `bundles/B<n>.json`. 5. Commit. |
| `promote-sandbox` | `BUNDLE`, `CLIENTS` | 1. Resolve the targets and check the brands. 2. Render each `<slug>/index.html` from the bundle's shell `index.html`, with the release inlined. 3. Upload. 4. Invalidate. 5. Commit client release `n+1`. |
| `promote-production` | `BUNDLE`, `CLIENTS` | The same as sandbox, plus: the bundle must already be in sandbox state (`REQ-006`), and missing artifacts are copied sandbox → production. Runs in the `production` deployment environment. |
| `rollback-sandbox` / `rollback-production` | `CLIENTS`, optional `TO` | Re-render each client from its previous release (or release `TO`), record it as a new release (`rollback of #k`), invalidate, commit. |
| `validate` (every push) | — | `clients.json` schema, bundle and state shapes, unit tests |

## Acceptance criteria

- **AC-001** (`REQ-001`):
  - `CLIENTS=kinton,mary-browns`, `ring:canary`, and `all` resolve to the expected slugs (unit tests);
  - an unknown slug fails.
- **AC-002** (`REQ-002`, `REQ-008`): `deploy-staging CLIENTS=kinton` creates `kinton/index.html` in the staging bucket. `/kinton/menu` loads with the Kinton brand. A client without a brand file fails before any upload.
- **AC-003** (`REQ-003`): `create-bundle` with `latest` produces `bundles/B1.json` with six versions. A missing artifact fails the run.
- **AC-004** (`REQ-004`):
  - after `promote-sandbox BUNDLE=B1 CLIENTS=kinton`, `/kinton/menu` on sandbox loads its shell and remotes from `/<repo>/<B1 version>/`;
  - `state/sandbox/kinton.json` shows release 1 → B1.
- **AC-005** (`REQ-005`): with two releases, rollback serves the previous bundle again and records release 3 = "rollback of #1".
- **AC-006** (`REQ-006`, `REQ-007`): `promote-production` of a bundle never seen in sandbox fails. The production pipelines are in the `production` deployment environment.
- **AC-007** (`REQ-009`, `REQ-010`): each run makes one invalidation and one git commit.
- **AC-008** (`REQ-011`): `DRY_RUN=true` performs no AWS writes and no git push. The printed plan lists every operation.
- **AC-009** (`GAP-001`): the CloudFront Function rewrites `/kinton/menu` → `/kinton/index.html` and leaves `/smooth-web-app/x.js`, `/configs/…`, `/brand-assets/…`, `/menu`, and `/login` unchanged (unit tests).
- **AC-010** (`REQ-012`): `clients.json` has 30 active clients.

## Open decisions

None. Decided by the user: staging has per-client deploys but always the latest version; per-client versions only in sandbox and production; the pipelines live in `smooth-web-release`; access is OIDC (`V2WebOrderingDeploy`).

## Human spec approval

- **Decision:** `APPROVED`
- **Reviewer:** babak@smooth.tech
- **Date:** 2026-11-16
- **Comments:** All five documents approved together ("Approve, build it").
