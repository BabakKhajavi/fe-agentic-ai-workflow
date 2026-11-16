# Developer Report: release-orchestration / TASK-001 to TASK-005

This is an execution report for the review handoff, not an approved SDD
artifact.

## Result

- **Status:** `READY_FOR_REVIEW`. The code is pushed; the first real runs are
  waiting on the Bitbucket setup.
- **Commits:**

  | Repository | Commit |
  | --- | --- |
  | `smooth-web-release` (`development`) | `91e3a8f` |
  | `smooth-web-app` (`staging`) | `1c933ac` |

- **Jira:** none.

## Traceability

| Requirement | Acceptance criteria | Where | Evidence |
| --- | --- | --- | --- |
| REQ-001 client targets | AC-001 | `scripts/lib/clients.js` | 6 unit tests (list, ring, all, unknown, inactive, disallowed) |
| REQ-002 staging deploy | AC-002 | `scripts/deploy-staging.js` | E2E test, plus a dry run against the real staging bucket (3 canary clients) |
| REQ-003 bundles | AC-003 | `scripts/create-bundle.js`, `lib/bundles.js` | E2E: `latest` resolution, missing artifact, React mismatch |
| REQ-004 promote | AC-004 | `scripts/promote.js`, `release.js`, `lib/render.js` | E2E: sandbox entry carries B1, then B2 for one client while the other stays on B1 |
| REQ-005 rollback | AC-005 | `scripts/rollback.js`, `lib/state.js` | E2E: release #3 = "rollback of #1"; a second rollback goes back to B2 |
| REQ-006 sandbox first, REQ-007 gate | AC-006 | `promote.js`; `bitbucket-pipelines.yml` (production deployment) | E2E refusal; YAML check |
| REQ-008 brand check | — | `lib/common.js` (one listing of `configs/`) | E2E failure before any write; real bucket listing |
| REQ-009 audit, REQ-010 one invalidation | AC-007 | `lib/git.js`, `lib/aws.js` | E2E: one invalidation and one commit per run |
| REQ-011 dry run | AC-008 | `lib/aws.js`, `lib/git.js` | E2E: reads only, no push |
| GAP-001 CloudFront Function | AC-009 | `cloudfront/client-entry-rewrite.js` | 3 tests run the exact file pasted into the console |
| GAP-002 host contract | — | host `public/index.html` | Host typecheck 0 errors, staging build OK; browser check pending (see below) |
| GAP-003 staging refresh | — | host `bitbucket-pipelines.yml` (from the generator) | YAML and bash validated |
| REQ-012 registry | AC-010 | `clients.json` | `validate`: 30 clients, 30 active |

## Verification

- `npm test`: **28/28** (21 unit tests, plus end-to-end flows against a
  fake AWS CLI and a local git remote).
- `node scripts/validate.js`: OK.
- Dry runs with real AWS (local `react-deployment` credentials, reads only):
  - `deploy-staging CLIENTS=ring:canary` printed the exact 3 copies, 1
    invalidation, and 1 commit.
  - `create-bundle` correctly reports that no tagged versions exist in sandbox
    yet.
- Pipelines YAML: valid. 7 steps pass `bash -n`. Only the production
  pipelines use the `production` deployment environment.

## Deviations and amendments

These are recorded in `CONTRACTS.md` under "Amendments":

- **Client entry pages live under `clients/<slug>/index.html`,** at the
  user's request.
- **The CloudFront Function was simplified.** It runs on Default (\*) only,
  so the reserved list was removed.
- **The brand check uses `s3:ListBucket` on `configs/`.** The local IAM user
  got 403 on HeadObject, because the assets buckets are readable only by
  CloudFront.
- **The full local "sandbox simulation" was skipped at the user's request.**
  It would have built all six repositories as versioned releases and opened a
  rendered entry page in the browser. So the host contract has not been
  exercised in a browser yet: the first `promote-sandbox` on real sandbox is
  that check.

## Remaining

1. Bitbucket setup for `smooth-web-release`:
   - Pipelines and OIDC enabled;
   - variables and `REPO_ACCESS_TOKEN`;
   - a restricted `production` deployment environment.
2. First runs:
   1. `deploy-staging` (dry run, then real);
   2. tag the six repositories;
   3. `create-bundle`;
   4. `promote-sandbox`;
   5. `promote-production`.
