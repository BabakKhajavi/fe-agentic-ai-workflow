# Impact: Per-client release orchestration

| Field | Value |
| --- | --- |
| Feature slug | `release-orchestration` |
| Repositories inspected | `smooth-web-release` (`development` at `3319281`), `smooth-web-app` (`staging` at `d92ce7f`) |
| Human plan approval | `APPROVED` (2026-11-16) |

## Baseline

- **`smooth-web-release`:** `clients.json` (30 clients; 2 active, 28 onboarding), the schema, and empty `bundles/`, `state/`, and `scripts/`. No pipeline.
- **Host `public/index.html`:**
  - sets `__SMOOTH_RELEASE_MANIFEST__ = null`;
  - fetches the global `/smooth-web-app/manifest.json` and `runtime-config.json` on non-localhost hosts.
- **Host staging pipeline:**
  - `sync --delete` to `smooth-web-app/`;
  - copies `dist/index.html` to the bucket root.
- **Sandbox:** tag builds publish to `<repo>/<version>/`.
- **Host pipelines** still have the old global `create-release` and `promote-to-production`. They're left in place and superseded by this feature.

## Affected areas

| ID | Action | Path |
| --- | --- | --- |
| `IMP-001` | Create | `smooth-web-release`: `package.json`, `scripts/lib/{clients,bundles,state,render,aws,env,git}.js`, `scripts/{deploy-staging,create-bundle,promote,rollback,validate}.js`, `test/*.test.js`, `cloudfront/client-entry-rewrite.js` plus its test, `bitbucket-pipelines.yml`, `RUNBOOK.md`; `clients.json` → all active |
| `IMP-002` | Modify | `smooth-web-app/public/index.html`: honour `__SMOOTH_CLIENT_RELEASE__` (`CON-003`) |
| `IMP-003` | Modify | `smooth-web-app/bitbucket-pipelines.yml` (from the generator): the staging step refreshes the existing `<slug>/index.html` entries and invalidates `/*` |

## Risks

| ID | Risk | Mitigation |
| --- | --- | --- |
| `RISK-001` | Two release runs at once can conflict on the git push. | `git pull --rebase` then retry once; otherwise fail clearly. |
| `RISK-002` | The CloudFront Function and IAM changes are manual. | `RUNBOOK.md` with exact steps; `DRY_RUN` for a first rehearsal. |
| `RISK-003` | Bitbucket custom pipelines take free-text variables. | Every input is validated (slug pattern, bundle id, semver) before any write. |
| `RISK-004` | `--delete` on the staging host removes old hashed assets before the client entries are refreshed. | The refresh runs immediately afterwards, in the same step. |

## Human plan approval

- **Decision:** `APPROVED`
- **Reviewer:** babak@smooth.tech
- **Date:** 2026-11-16
- **Comments:** All five documents approved together ("Approve, build it").
