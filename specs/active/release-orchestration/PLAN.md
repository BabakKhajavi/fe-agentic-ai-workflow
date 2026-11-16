# Plan: Per-client release orchestration

| Field | Value |
| --- | --- |
| Feature slug | `release-orchestration` |
| Based on | REQUIREMENTS, SPEC, CONTRACTS, IMPACT |
| Human plan approval | `APPROVED` (2026-11-16) |

## Tasks

1. **TASK-001: release library and tests.** Write the pure logic in `smooth-web-release/scripts/lib`, with `node --test` tests.
   - Covers: client resolution, the bundle and state model, entry-page rendering and escaping, and the CloudFront rewrite.
   - Acceptance criteria: `AC-001`, `AC-005`, `AC-008`, `AC-009`.
2. **TASK-002: pipeline scripts.** Write `deploy-staging`, `create-bundle`, `promote`, `rollback`, and `validate`.
   - AWS and git go through one wrapper that honours `DRY_RUN`.
   - Rehearse each script locally against the real staging bucket in dry-run mode.
   - Acceptance criteria: `AC-002` to `AC-007`.
3. **TASK-003: Bitbucket pipelines and runbook.** Write `bitbucket-pipelines.yml` with custom pipelines and production deployment environments, plus `RUNBOOK.md` (CloudFront Function, IAM, variables). Set all clients to active (`AC-010`).
4. **TASK-004: host contract.**
   - In `index.html`, honour `__SMOOTH_CLIENT_RELEASE__`.
   - In the host staging pipeline, refresh the client entry pages.
   - Verify locally with a rendered entry page in the browser.
5. **TASK-005: push and first run.**
   - Push `smooth-web-release` (`development`) and the host `staging` branch.
   - The user attaches the CloudFront Function, then runs `deploy-staging CLIENTS=kinton` (dry run first).

## Verification

- Unit tests and the `validate` script.
- Dry-run plans against the real buckets.
- Local browser check of a rendered entry page.
- The staging run (`AC-002`) after the user's AWS steps.

## Human plan approval

- **Decision:** `APPROVED`
- **Reviewer:** babak@smooth.tech
- **Date:** 2026-11-16
- **Comments:** All five documents approved together ("Approve, build it").
