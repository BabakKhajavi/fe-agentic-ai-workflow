# Deployment and Client Onboarding

This is how Web Two code reaches each client in each environment, and the
ordered steps for onboarding a new client.

- **Specs:**
  - `specs/active/release-orchestration/` (per-client releases);
  - `specs/active/rspack-mf2-migration/` (build and federation);
  - `specs/active/menu-page-example/` (brand at runtime).
- **Rules:** `TECHNICAL_REQUIREMENTS.md` `TECH-158`–`TECH-164`, and
  `ENGINEERING_STANDARDS.md` "Releases and deployment".
- **Last updated:** 2026-11-16.

## 1. The model in one paragraph

**Code is built once per version and shared by every client. Branding is
data, loaded at runtime. Each client gets its own small entry page that pins
which code version it runs.**

- In **staging**, every client runs the latest build.
- In **sandbox** and **production**, each client runs the **bundle** (a
  tested set of six repository versions) it was promoted to.
- Clients can be on different bundles at the same time, and rollback is
  instant because nothing is rebuilt.

| What differs per client | How | Where it comes from |
| --- | --- | --- |
| Look (colours, logo, font) | Fetched at runtime | `/configs/<slug>-brand.json` (assets bucket) |
| Code version (sandbox and production) | Pinned in the client's entry page | `clients/<slug>/index.html` (app bucket), written by the release pipelines |

## 2. Environments and AWS resources

AWS account `869604471131`, region `ca-central-1`. All pipelines reach AWS
through the OIDC role **`V2WebOrderingDeploy`**, with no stored AWS keys.

| Environment | Versions | App bucket | Assets bucket | Distribution |
| --- | --- | --- | --- | --- |
| staging | always latest | `smooth-web-staging-869604471131-ca-central-1-an` | `smooth-web-assets-staging-869604471131-ca-central-1-an` | `E196OL3R788LB5` (`smooth-web-ordering-staging.dev.smoothcommerce.tech`) |
| sandbox | per-client bundle | `smooth-web-sandbox-869604471131-ca-central-1-an` | `smooth-web-assets-sandbox-…-an` | `EF7FM54UW3LQE` |
| production | per-client bundle | `smooth-web-production-869604471131-ca-central-1-an` | `smooth-web-assets-production-…-an` | `E2TQAHRFRZI9M7` (planned host `smooth.menu`) |

**CloudFront behaviours (every distribution):**

| Path | Origin | Notes |
| --- | --- | --- |
| `/configs/*` | assets bucket (REST + OAC, private) | brand JSON; short cache plus invalidation |
| `/brand-assets/*` | assets bucket | versioned images and fonts; long cache |
| `/global/*` | assets bucket | shared assets |
| Default `(*)` | app bucket (S3 website) | **Viewer request: CloudFront Function `client-entry-rewrite`** |

`client-entry-rewrite`
(`smooth-web-release/cloudfront/client-entry-rewrite.js`) rewrites
`/<slug>/<route>` to `/clients/<slug>/index.html`. It leaves the site root,
files with an extension, `/clients/…`, and `/smooth-web-*/…` unchanged.
The browser URL never changes.

## 3. Bucket layout

**App bucket (staging):**

```text
index.html                         shell entry for "/" (latest)
clients/<slug>/index.html          copy of the latest shell entry, one per deployed client
smooth-web-app/                    latest shell build (overwritten on each staging merge)
smooth-web-<mfe>/                  latest build of each MFE (core, data, design, auth, payment)
```

**App bucket (sandbox and production):**

```text
index.html                         shell entry for "/"
clients/<slug>/index.html          shell HTML of the client's bundle + inlined __SMOOTH_CLIENT_RELEASE__
smooth-web-app/<version>/          immutable, one folder per tagged version
smooth-web-<mfe>/<version>/        immutable, one folder per tagged version
```

- Sandbox holds **every** tagged version.
- Production holds only versions that were promoted there; the promote
  pipeline copies them from sandbox.

**Assets bucket (every environment):**

```text
configs/<slug>-brand.json          brand data (Brand / BrandColors v2)
brand-assets/clients/<slug>/…      logos, icons, images, fonts (versioned names, e.g. logo.v2.svg)
global/…                           shared assets
```

## 4. Who builds and who deploys

| | Package pipelines (the 6 app repositories) | `smooth-web-release` pipelines |
| --- | --- | --- |
| Responsibility | Build and publish **code** | Decide **which client runs which code** |
| PR / `development` | typecheck and build only | `validate` (registry, bundles, state, tests) on every push |
| `staging` branch | build → `staging-bucket/<repo>/` (latest). The host also refreshes every `clients/*/index.html`. | `deploy-staging` |
| `v*` tag | build once → `sandbox-bucket/<repo>/<version>/` (immutable) | `create-bundle` |
| Sandbox / production | — | `promote-sandbox`, `promote-production`, `rollback-sandbox`, `rollback-production` |
| Knows about clients | no | yes (`clients.json`) |

The package pipeline files are generated from one template, so all six stay
identical except for each repository's staging environment variables.

## 5. Release pipelines (`smooth-web-release`)

Run them from Bitbucket: **Pipelines → Run pipeline**, branch `development`.

| Pipeline | Inputs | What it does |
| --- | --- | --- |
| `deploy-staging` | `CLIENT`, `CLIENTS`, `DRY_RUN` | copies the latest shell entry to `clients/<slug>/index.html` |
| `create-bundle` | `SHELL_VERSION` … `PAYMENT_VERSION` (version or `latest`) | checks the artifacts exist in sandbox and that shared React matches, then writes `bundles/B<n>.json` |
| `promote-sandbox` | `BUNDLE`, `CLIENT`, `CLIENTS`, `DRY_RUN` | renders each client's entry from the bundle's shell HTML with the release inlined |
| `promote-production` | the same | the same, but needs the **production** deployment-group approval; the bundle must already be in sandbox; artifacts are copied from sandbox |
| `rollback-sandbox` / `rollback-production` | `CLIENT`, `CLIENTS`, optional `TO` | re-renders the client's previous release (or release `TO`); nothing is rebuilt |

**Choosing clients:**

| Field | Values |
| --- | --- |
| `CLIENT` (dropdown) | `ring:canary` (default), `ring:general`, `all`, or one slug |
| `CLIENTS` (optional) | a typed list such as `kinton,mary-browns`; overrides the dropdown |

The dropdown is generated from `clients.json` by
`node scripts/sync-pipeline-clients.js`, and `validate` fails if they drift.

**Every run:**
1. Validates the input before writing anything.
2. Fails if a selected client has no `configs/<slug>-brand.json`.
3. Issues **one** CloudFront invalidation.
4. Commits `state/<env>/<slug>.json` (who, when, which bundle, which
   pipeline run).

`DRY_RUN=true` prints every AWS and git change without making it. Use it
for any first run, for any run with `CLIENT=all`, and before production.

**Repository variables:**

| Variable | Notes |
| --- | --- |
| `STAGE_/SANDBOX_/PROD_APP_BUCKET`, `*_ASSETS_BUCKET`, `*_APP_DIST` | Workspace or repository level |
| `API_URL_RELEASE`, `API_URL_PRODUCTION` | Inlined runtime API URL for sandbox and production |
| `REPO_ACCESS_TOKEN` | Secured repository access token, *Repositories: Write*, expires 2027-11-16. Needed because the runs commit client state; OIDC covers AWS only, not git. |

**IAM (`V2WebOrderingDeploy`):** trusts the six app repositories and
`smooth-web-release`.

| Resource | Actions |
| --- | --- |
| App buckets | `s3:ListBucket`, `GetObject`, `PutObject` |
| Assets buckets | `s3:ListBucket` (brand-file check; the buckets stay readable only by CloudFront) |
| Distributions | `cloudfront:CreateInvalidation` |

## 6. How a client's versions are resolved (sandbox and production)

1. `/kinton/menu` → CloudFront Function → `/clients/kinton/index.html`.
2. That HTML is the **bundle's shell build**, so its `<script>` tags point at
   `/smooth-web-app/<shell version>/…`. The HTML picks the shell version.
3. The first element of `<head>` is
   `window.__SMOOTH_CLIENT_RELEASE__ = { bundle, versions, runtimeConfig }`.
4. The host's pre-bootstrap script (`smooth-web-app/public/index.html`):
   - sets `__SMOOTH_RELEASE_MANIFEST__ = versions`;
   - merges `runtimeConfig`;
   - skips the global manifest fetch.
5. The Module Federation runtime plugin
   (`smooth-web-build-config/runtime/remote-resolver.js`) resolves each remote
   to `<origin>/smooth-web-<repo>/<version>/mf-manifest.json`. Nested remotes
   (for example core → design and data) read the same versions, so the whole
   page runs exactly one bundle.
6. React, MUI, and Redux load once, from the host (shared singletons).
7. The brand loads separately from `/configs/kinton-brand.json`.

**In staging,** nothing is inlined. The resolver uses the latest
`/smooth-web-<repo>/mf-manifest.json`.

## 7. Branding

- **Format:** `configs/<slug>-brand.json` follows the app's `Brand` /
  `BrandColors` v2 types (`smooth-web-app/src/types/brand.ts`):
  - `schemaVersion`, `brandId` and `slug` (equal to the slug), `appName`;
  - `logoUrl`, `smallLogoUrl`, `faviconUrl`;
  - `fontFamily`;
  - `colors` (brand, foundation, selection);
  - `version`, `updatedAt`.
- **Upload headers:**
  - brand JSON: `Content-Type: application/json` and
    `Cache-Control: public, max-age=0, must-revalidate, s-maxage=3600`, then
    invalidate `/configs/<slug>-brand.json`;
  - files under `brand-assets/`: `public, max-age=31536000, immutable`, with
    the version in the file name.
- **Fonts:** the app loads only its supported Google fonts (Onest, DM Sans,
  Plus Jakarta Sans, Manrope, Work Sans, Archivo, IBM Plex Sans, Sora, Outfit,
  Space Grotesk). Other fonts fall back to Onest until added to the app list
  (Google fonts) or uploaded as `woff2` (custom fonts).
- **Existing clients:** the 30 brand files were generated on 2026-11-16 from
  `web-ordering-client-config` (one branch per slug) and are live in staging.
- **Brand service:** a DynamoDB + Lambda brand service that would publish
  these files is designed but **not built**. Brand files are uploaded by hand
  for now.

## 8. Onboarding a new client: ordered steps

| # | Step | Who | Done when |
| --- | --- | --- | --- |
| 1 | **Pick the slug.** Lowercase, digits, hyphens (`^[a-z0-9]+(-[a-z0-9]+)*$`). It is fixed forever and becomes the URL (`/<slug>/menu`), the brand file name, and the state file name. | Product / ops | Slug agreed |
| 2 | **Create the brand file** `<slug>-brand.json` in the format of section 7. For a client that already exists in Web One, generate it from its `web-ordering-client-config` branch. Check contrast (body text ≥ 7:1, muted text ≥ 4.5:1) and the font. | Developer / design | File validates against the `Brand` shape |
| 3 | **Upload brand assets** (logo, small logo, favicon, optional images and fonts) to `brand-assets/clients/<slug>/` in **each** assets bucket, with versioned names. Point the URLs in the brand file at them. | Developer / ops | Asset URLs return 200 through CloudFront |
| 4 | **Upload the brand file** to `configs/<slug>-brand.json` in **each** environment's assets bucket, with the headers in section 7, then invalidate that path. | Ops | `https://<env-host>/configs/<slug>-brand.json` returns 200 `application/json` |
| 5 | **Register the client** in `smooth-web-release/clients.json` (`slug`, `name`, `status: onboarding`, `environments`, `ring`, usually `general`). Run `node scripts/sync-pipeline-clients.js`, commit, and merge to `development`. | Developer | `validate` passes; the slug appears in the `CLIENT` dropdown |
| 6 | **Set `status: active`** in `clients.json` once the brand file exists in the target environments, and merge. Only active clients can be deployed, and only they are included in `all` and rings. | Developer | `validate` passes |
| 7 | **Deploy to staging:** run `deploy-staging` with `CLIENT=<slug>`, `DRY_RUN=true`, then again with `false`. | Release owner | `/<slug>/menu` on staging shows the client's brand |
| 8 | **Promote to sandbox:** run `promote-sandbox` with `BUNDLE=<current bundle>` and `CLIENT=<slug>`. | Release owner | `/<slug>/menu` on sandbox loads from `/smooth-web-<repo>/<version>/` |
| 9 | **Client or QA acceptance** on sandbox. | Client / QA | Sign-off |
| 10 | **Promote to production:** run `promote-production` with the same bundle (deployment-group approval). | Release owner + approver | `/<slug>/menu` live in production |

**Removing or pausing a client:** set `status: suspended` (excluded from all
runs), run the sync script, and merge. Its entry pages stay until removed by
hand.

## 9. Everyday release flow (existing clients)

1. **Merge to `staging`** in any app repository. That repository's latest
   build goes to staging. Clients already deployed to staging pick it up
   immediately.
2. **Cut versions:** run `prepare-release` in each changed repository. It
   bumps the version, tags it, and the tag build publishes to sandbox.
3. **Run `create-bundle`** (explicit versions or `latest`) to get `B<n>`.
4. **Run `promote-sandbox`** with `BUNDLE=B<n>` and `CLIENT=ring:canary`,
   verify, then widen with `ring:general` or `all`.
5. **Run `promote-production`** with the same bundle, canary first, then
   wider. It needs deployment-group approval.
6. **If needed, run `rollback-production`** with `CLIENT=<slug>` (optionally
   `TO=<release#>`).

## 10. Known gaps (2026-11-16)

| Gap | Effect | Follow-up |
| --- | --- | --- |
| CloudFront error pages are not mapped | Deep links and `/menu` return HTTP 404 with the app body | Map 403/404 → `/index.html` with 200 on the Default (*) behaviour only |
| Unknown slug | Falls back to the root `index.html` with the default brand, not "store not found" (AD-2.6) | App-level handling |
| Brand snapshot not inlined in entry pages | First paint waits for the brand fetch | CI/CD D1 follow-up |
| Logos on external hosts (Zeplin, Squarespace, Google Drive, client sites) | Can disappear | Re-host under `brand-assets/clients/<slug>/` |
| 28 clients' fonts not loaded | They render in Onest | Add Google fonts to the app list; upload custom `woff2` files |
| Consumer MFEs skip typecheck in CI | Remote types aren't reachable in CI (`rspack-mf2-migration` RISK-006) | Publish federated types with artifacts |
| Package pipeline generator lives outside the repositories | Template changes depend on a local script | Move it into `smooth-web-build-config` |
| Sandbox and production hostnames | Not final (production planned as `smooth.menu`) | Execution Plan Q2.6b |
