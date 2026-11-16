# Contracts: Per-client release orchestration

| Field | Value |
| --- | --- |
| Feature slug | `release-orchestration` |
| Integration mode | `LIVE` (S3, CloudFront, git) |
| Human spec approval | `APPROVED` (2026-11-16) |

## CON-001 — Bundle (`bundles/B<n>.json`, immutable)

```json
{ "id": "B1", "createdAt": "2026-11-16T00:00:00Z", "createdBy": "<triggerer>", "pipeline": "<build #>",
  "versions": { "shell": "1.0.0", "auth": "1.0.0", "design": "1.0.0", "data": "1.0.0", "core": "1.0.0", "payment": "1.0.0" } }
```

The repository for each key:

| Key | Repository |
| --- | --- |
| `shell` | `smooth-web-app` |
| `auth` | `smooth-web-auth-mfe` |
| `design` | `smooth-web-design-mfe` |
| `data` | `smooth-web-data-mfe` |
| `core` | `smooth-web-core-mfe` |
| `payment` | `smooth-web-payment-mfe` |

The keys equal the release-manifest keys the remote resolver reads.

## CON-002 — Client state (`state/<env>/<slug>.json`)

```json
{ "slug": "kinton", "environment": "sandbox",
  "current": { "release": 2, "bundle": "B2", "at": "…", "by": "…", "pipeline": "…", "note": "promote" },
  "history": [ { "release": 2, "bundle": "B2", "…": "…" }, { "release": 1, "bundle": "B1", "…": "…" } ] }
```

In staging, `current` is `{ "channel": "latest", … }` with no `release` or `bundle`.

## CON-003 — Client entry page (`<slug>/index.html` in the environment's app bucket, `Cache-Control: no-cache, no-store, must-revalidate`)

- **Sandbox and production:** the bundle's shell `index.html` (`smooth-web-app/<shell>/index.html`), with this script inserted as the first child of `<head>`:

  ```html
  <script>window.__SMOOTH_CLIENT_RELEASE__={"slug":"kinton","environment":"sandbox","release":2,"bundle":"B2",
  "versions":{"shell":"1.0.0",…},"runtimeConfig":{"apiUrl":"…"}};</script>
  ```

  The JSON is escaped (`<` → `<`).
- **Staging:** a plain copy of the staging root `index.html`, with no inserted release, because staging always uses the latest build.
- **Host contract:** if `window.__SMOOTH_CLIENT_RELEASE__` is set, the host pre-bootstrap script:
  1. sets `window.__SMOOTH_RELEASE_MANIFEST__ = versions`;
  2. merges `runtimeConfig` into `__SMOOTH_RUNTIME_CONFIG__`;
  3. skips fetching the global `manifest.json` and `runtime-config.json`.

  The remote resolver then loads `<origin>/<repo>/<version>/mf-manifest.json` (already implemented).

## CON-004 — CloudFront Function (`cloudfront/client-entry-rewrite.js`, viewer request, default behaviour)

- A URI **with a file extension** is unchanged.
- **Navigation requests** to `/<slug>` or `/<slug>/…` are rewritten to `/<slug>/index.html`.
- **Unchanged first segments:**
  - `configs`, `brand-assets`;
  - any `smooth-web-*` prefix;
  - host root routes: `menu`, `login`, `verify`, `verify-otp`, `reset-password`, `google-callback`, `dashboard`, `leads`, `pipelines`, `tasks`, `automations`, `communications`, `reports`, `settings`, `billing`.

## CON-005 — Pipeline variables and access

| Name | Purpose |
| --- | --- |
| `STAGE_/SANDBOX_/PROD_APP_BUCKET`, `*_APP_DIST`, `*_ASSETS_BUCKET` | Existing environment resources |
| `API_URL_RELEASE`, `API_URL_PRODUCTION` | `runtimeConfig.apiUrl` for sandbox and production entry pages (`ASM-001`) |
| `REPO_ACCESS_TOKEN` | Pushes state and bundle commits (repository write) |
| OIDC role `V2WebOrderingDeploy` | Trusts `smooth-web-release`. S3 List/Get/Put on the app buckets, List/Get on the assets buckets, `cloudfront:CreateInvalidation` on the three distributions |

## Amendments (2026-11-16, agreed with the user during implementation)

- **Client entry pages live under `clients/`:** `clients/<slug>/index.html`
  (was `<slug>/index.html`). This keeps the bucket root clean, avoids slug
  and folder collisions, and lets IAM scope release writes to `clients/*`.
  Invalidations use `/clients/<slug>/index.html`.
- **CON-004 is simplified.** The function runs only on the Default (\*)
  behaviour, so `/configs/*`, `/brand-assets/*`, and `/global/*` never reach
  it; the reserved list was removed. The rules are:
  1. a URI ending in a file is unchanged;
  2. `/`, `/clients/…`, and `/smooth-web-*` are unchanged;
  3. anything else, `/<slug>/…`, is rewritten to `/clients/<slug>/index.html`.

  Host routes without a slug (`/menu`) fall back to the root `index.html`
  through the S3 error document, as before.
- **The brand check lists `configs/`** (`s3:ListBucket`) instead of reading
  each object. The assets buckets stay readable only by CloudFront, so the
  release role needs only `s3:ListBucket` on them.

## Human spec approval

- **Decision:** `APPROVED`
- **Reviewer:** babak@smooth.tech
- **Date:** 2026-11-16
- **Comments:** All five documents approved together ("Approve, build it").
