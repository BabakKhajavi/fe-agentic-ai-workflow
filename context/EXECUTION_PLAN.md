# Web Two Execution Plan

> **Human-facing copy:** Confluence, "Web 2.0 Execution Plan" (draft),
> https://smoothpay.atlassian.net/wiki/spaces/~6104334d3900e60070b6d9bb/pages/3204939777
> (page ID `3204939777`).
> This local file is the agents' context and keeps the full decision
> history. Keep the two in sync. When the Confluence page is edited, record
> the change here, and the reverse.

This document records the execution decisions for **Web Two**, the
`smooth-web` rebuild this AI-assisted SDD workflow was created for. Each
change is recorded in the order it was decided, with its date, its source, and
what it supersedes, so later agents can trace why the context files say what
they say.

Feature-level SDD artifacts under `specs/active/<feature>/` must conform to
the decisions recorded here. If a feature artifact conflicts with a decision
here, stop and ask for human resolution.

---

## Change 1 — Consolidate the micro-frontends (decided 2026-11-08)

**Source:** user decision (babak@smooth.tech), 2026-11-08.

### Target structure

| Repository | Responsibility |
| --- | --- |
| `smooth-web-app` | Host shell: shell layout, sidebar, app bar, routing, white-labeling. |
| `smooth-web-data-mfe` | All APIs, global state, and configuration. |
| `smooth-web-design-mfe` | Pure, domain-free reusable components in atomic design (see `TECH-152`). |
| `smooth-web-core-mfe` | Core ordering domain (see breakdown below). |
| `smooth-web-payment-mfe` | All payment-related features: payment methods, gift cards, tips. |
| `smooth-web-auth-mfe` | The whole authentication flow and its components: accounts, login, sign-up, forgot password, and so on. APIs come from `smooth-web-data-mfe`. |

`smooth-web-core-mfe` domains. Each domain is its own top-level feature
folder, `src/features/<domain>/` (confirmed 2026-11-08). Its name pairs with
the matching `smooth-web-data-mfe/src/apis/<domain>/` folder, as
`TECH-130`–`TECH-135` require:

- **Locations** — location selection, map, pickup, delivery, drop-off.
- **Menu** — categories; products (options, favourites, upsell); combos.
- **Offers** — offers, loyalty, how it works, promo code, credit.
- **Orders** — order settings, order status, order history, order again.

### Clarifications (2026-11-08)

- The repository name is `smooth-web-core-mfe`, with the `-mfe` suffix.
- Every payment-related feature lives in `smooth-web-payment-mfe`, not in
  `smooth-web-core-mfe`.
- All APIs live in `smooth-web-data-mfe`.
- `smooth-web-design-mfe` contains only pure, reusable atomic-design
  components. Nothing in it may be named after or coupled to a business
  domain. The standards were updated to say so: `TECH-152`,
  `ENGINEERING_STANDARDS.md`, `ARCHITECTURE.md`, and the Developer Agent.

### What this supersedes

- `smooth-web-settings-mfe` is no longer part of the structure. Its confirmed
  role (2026-09-11) as owner of the shared component library moves to
  `smooth-web-design-mfe`. `TECH-142` through `TECH-147`, the engineering
  standards, and the Developer Agent now name `smooth-web-design-mfe`.
- `smooth-web-design-mfe` is no longer described as "per-client
  branding/presentation". White-labeling belongs to `smooth-web-app`.
- `smooth-web-dashboard-mfe` is not part of the structure.
- `smooth-web-menu-mfe` and the local `smooth-web-location-mfe`,
  `smooth-web-offers-mfe`, and `smooth-web-order-mfe` scaffolds are replaced
  by the single `smooth-web-core-mfe` repository.

### Current local state vs. target (inspected 2026-11-08)

| Local repository | `origin` remote | Module Federation name | Target |
| --- | --- | --- | --- |
| `smooth-web-app` | `smooth-web-app.git` | host | keep |
| `smooth-web-data-mfe` | `smooth-web-data-mfe.git` | `smooth_data` | keep |
| `smooth-web-design-mfe` | `smooth-web-design-mfe.git` | `smooth_ui` | keep (already has `src/atoms`, `molecules`, `organisms`, `templates`) |
| `smooth-web-menu-mfe` | — | — | **removed locally 2026-11-08**; scope moves to `smooth-web-core-mfe` |
| `smooth-web-location-mfe` | — | — | **removed locally 2026-11-08**; scope moves to `smooth-web-core-mfe` |
| `smooth-web-offers-mfe` | — | — | **removed locally 2026-11-08**; scope moves to `smooth-web-core-mfe` |
| `smooth-web-order-mfe` | — | — | **removed locally 2026-11-08**; scope moves to `smooth-web-core-mfe` |
| `smooth-web-payment-mfe` | `smooth-web-settings-mfe.git` ⚠ | `smooth_settings` ⚠ | keep, needs its own identity |
| `smooth-web-auth-mfe` | `smooth-web-menu-mfe.git` ⚠ | `smooth_menu` ⚠ | keep, needs its own identity |
| `smooth-web-core-mfe` | none yet | `smooth_core` | **scaffolded locally 2026-11-08**: no commits, no Bitbucket remote, dev port 9007 |

⚠ These repositories are unmodified copies of `smooth-web-menu-mfe` or
`smooth-web-settings-mfe`. Each has a `Placeholder` module and still carries
the source repository's git remote and federation name. Pushing from one of
them would push to the wrong Bitbucket repository, and loading two of them
would collide on the same federation name.

The host `smooth-web-app/webpack.config.js` still declares `smooth_auth`,
`smooth_ui`, `smooth_data`, `smooth_dashboard`, `smooth_settings`, and
`smooth_menu`. It does not yet match the target.

### Consequences for execution

1. ~~Scaffold `smooth-web-core-mfe` locally.~~ Done 2026-11-08 at the
   user's request. It was copied from the menu-mfe template, renamed to
   `smooth_core` (`./CoreApp`, `CoreProvider`, dev port 9007), and given
   empty `src/features/{locations,menu,offers,orders}/` folders. It is
   `git init`-ed with nothing committed.
   **Remaining:** create the Bitbucket repository, add it as `origin`, and
   make the first commit. The workspace file
   `../smooth-web-sdd.code-workspace` now lists the six target repositories.
2. Give `smooth-web-payment-mfe` and `smooth-web-auth-mfe` their own
   Bitbucket remotes and the federation names `smooth_payment` and
   `smooth_auth`.
3. Update the host remotes to the six target repositories: remove
   `smooth_dashboard`, `smooth_settings`, and `smooth_menu`, and add core and
   payment.
4. ~~Remove the local `smooth-web-menu-mfe`, `-location-mfe`, `-offers-mfe`,
   and `-order-mfe` folders.~~ Done 2026-11-08 at the user's request. All
   four were clean, with no uncommitted changes, unpushed commits, or stashes.
   Their content still exists on Bitbucket.
5. Update the references that still name the retired repositories:
   - `smooth-web-app`: `webpack.config.js` remotes,
     `scripts/utils/getMfeBaseUrl.js`, `scripts/utils/createPromiseRemote.js`,
     and the `bitbucket-pipelines.yml` release manifest (menu, settings,
     dashboard);
   - `smooth-web-auth-mfe/package.json`, whose `"name"` is still
     `smooth-web-menu-mfe`.

These are infrastructure steps. Like any other implementation work, they
need human approval before agents carry them out.

### Resolved questions (2026-11-08)

- **Q1.1** The repository name is `smooth-web-core-mfe`.
- **Q1.2** The federation names are `smooth_core` and `smooth_payment`.
- **Q1.3** `smooth-web-auth-mfe` owns the whole authentication flow and its
  components. Its APIs come from `smooth-web-data-mfe`.
- **Q1.4** Each core domain gets its own folder:
  `src/features/<domain>/`.
- **Q1.5** White-labeling and the MUI theme belong to the shell,
  `smooth-web-app`, which already has them in `src/theme/createAppTheme.ts`
  and `src/providers/theme-provider.tsx`.
  - *Note:* `smooth-web-data-mfe/src/theme/brand-colors.ts` also exists.
    Confirm whether it should move to the shell or stay as brand config data.
- **Q1.6** The design system follows its existing atomic-design structure:
  `src/<atoms|molecules|organisms|templates>/<group>/`. The engineering
  standards were updated to match.
- **Q1.7** The existing `smooth-web-design-mfe` structure and components are
  accepted as they are. `TECH-152` applies to new and changed components.

---

## Change 2 — Delivery phases and runtime architecture (decided 2026-11-08)

**Source:** user decisions (babak@smooth.tech), 2026-11-08, made in two
rounds of architecture questions.

The user is finishing the repository restructure from Change 1 (host
remotes, auth/payment identity, the core-mfe remote) outside this plan.

### Phase order

Reordered on 2026-11-08 to put foundations first:

1. **Foundations:** repositories, Rspack + Module Federation 2 build
   tooling, Bitbucket-to-AWS OIDC, AWS frontend resources (S3 and CloudFront
   only), and a basic staging deploy
2. **White-labeling and client configuration**
3. **Release CI/CD:** versioning, per-client and per-environment releases,
   all clients in one run, rollback, and the production gate
4. **Gradual package and flow updates**, building the UI as mockup designs
   arrive

**Why foundations come first:** white-labeling can only be verified on
staging, and the backend can only publish `/configs/<slug>-brand.json`,
once the repositories, AWS resources, IAM identities, and a staging deploy
exist. White-labeling code may start locally, against sample brand JSON
files, while Phase 1 is in progress.

Each phase is delivered as one or more SDD features under
`specs/active/<feature>/`. The normal approval gates apply.

### Architecture decisions

| ID | Decision |
| --- | --- |
| AD-2.1 | **Client resolution is by path:** `https://<base-url>/<slug>/<route>`, for example `/<slug>/menu`. The base URL is shared by all clients; the slug identifies the client. |
| AD-2.2 | **Build once, white-label at runtime.** The same versioned build serves every client. The client is resolved on first load, and its config is cached after that. |
| AD-2.3 | **Client config is a static JSON file, `/configs/<slug>-brand.json`, on S3 behind CloudFront.** It holds brand, theme, feature flags, API base URL, and so on. |
| AD-2.4 | **Caching:** CloudFront caches `slug-brand.json`. The app keeps a local copy, renders from it straight away, and refreshes it in the background (stale-while-revalidate). When the config changes, the file is invalidated on CloudFront. |
| AD-2.5 | **The backend (merchant portal) publishes config.** It writes `slug-brand.json` to S3 and invalidates CloudFront. The frontend only reads this file. |
| AD-2.6 | **Release versions are per client.** Clients in the same environment can run different releases, for example client A on R12 and client B on R11. |
| AD-2.7 | **A bootstrap loader pins each client's release.** A small, unversioned entry page reads the client's release manifest, then loads that release's shell and remotes. This extends the existing `createPromiseRemote` and release-manifest mechanism. Rollback means pointing the client's manifest back to an earlier release and invalidating it. |
| AD-2.8 | **Environments:** `staging` always runs the latest build from the `staging` branch and is not versioned. `release` and `production` run versioned releases. Branches are `staging` and `production`; tags create releases. A `release` environment exists, also called sandbox. |
| AD-2.9 | **AWS resources are created by hand in the console.** There is no infrastructure-as-code. Every resource and setting must be written down in a runbook so each environment can be rebuilt. |
| AD-2.10 | **Pipelines reach AWS through OIDC, with no long-lived keys** (added 2026-11-08). Bitbucket Pipelines assume a per-environment IAM role with short-lived credentials. No AWS access keys are stored in repository variables. |

### Constraints that follow from these decisions

- **All clients share one browser origin** (AD-2.1). `localStorage`,
  `sessionStorage`, cookies, and auth tokens are therefore shared across
  slugs, so every client-scoped key must be namespaced by slug. Also
  confirm with the backend how auth sessions should behave across slugs.
- **Deep links** such as `/<slug>/menu/123` must resolve to the bootstrap
  loader. CloudFront needs an SPA fallback for `/<slug>/*` that does not
  break the versioned asset paths.
- **Cache policies by content type:**
  - Versioned artifacts are immutable and can be cached for a long time.
  - The bootstrap entry, the client manifests, and `slug-brand.json` need a
    short TTL or explicit invalidation.
- **`slug-brand.json` is a contract between the backend and the frontend.**
  Phase 2 must define its schema and version stamp in `CONTRACTS.md`.
- **Ownership** follows Change 1:
  - Loading, caching, and state for client config live in
    `smooth-web-data-mfe` (config).
  - Turning the brand into the MUI theme and routing by slug live in
    `smooth-web-app` (shell). It already does this through
    `createAppTheme(tenantBrand)`.

### Phase outlines

**Phase 1: Foundations (repositories, OIDC, AWS resources, staging deploy)**

*1a. Bitbucket repositories*
- Create the missing Bitbucket repositories: `smooth-web-core-mfe` and
  `smooth-web-payment-mfe`. Give `smooth-web-auth-mfe` its own repository;
  it still points at the menu-mfe repository.
- Set up all six repositories the same way:
  - `staging` and `production` branches;
  - branch permissions: a pull request and an approval are needed to merge;
  - a tag naming convention for releases;
  - Bitbucket deployment environments: staging, sandbox, and production.
    Only the deployment group can deploy to production.
- Archive (make read-only) the retired repositories (menu, settings,
  dashboard) once nothing references them.

*1b. Build tooling: Rspack 2 + Module Federation 2 + shared config*
  (Change 3; spec `specs/active/rspack-mf2-migration/`)
- Move all six repositories from Webpack 5 to Rspack 2, using the
  Module Federation 2 plugin.
- Build all six from the shared config package `@smooth/web-build-config`
  (repository `smooth-web-build-config`).
- Replace `createPromiseRemote` with a Module Federation runtime plugin.
- Use Node 22 and add a `typecheck` gate.

*1c. Bitbucket to AWS OIDC*
- Register the Bitbucket workspace as an IAM OIDC identity provider, once
  per AWS account.
- Create one IAM role per environment. Each role:
  - trusts only the six Web Two repositories (production trusts only the
    production deployment environment);
  - has least-privilege write access to that environment's builds, entry
    page, and manifests, plus invalidation of that environment's CloudFront
    distribution.
- Pipeline steps use `oidc: true` and assume their role. Then remove the
  long-lived AWS keys from repository variables.
- The backend's `slug-brand.json` writer gets its own separate AWS identity,
  limited to `/configs/*` and invalidation.

*1d. AWS frontend resources (manual)*
- Per environment:
  - S3 locations for versioned artifacts, the bootstrap entry, client
    manifests, and client configs (`/configs/`);
  - a CloudFront distribution with path behaviours, cache policies, the SPA
    fallback for `/<slug>/*`, and certificates.
- Record every resource, the OIDC provider, and the roles in the runbook.

*1e. Basic staging deploy*
- On each merge to `staging`, every one of the six repositories runs lint,
  typecheck, and tests, builds, and deploys the latest build to staging
  through its OIDC role.
- The staging shell loads the latest build of every MFE.

**Exit:**
- All six repositories exist and are configured.
- All six build with Rspack 2 + Module Federation 2 from the shared config.
- In each environment, a test step assumes its role through OIDC, can write
  only where the role allows, and is refused elsewhere.
- Merging to `staging` deploys any repository, and the shell loads all six
  on the staging hostname.
- The runbook is complete and has been checked against one environment.

**Phase 2: White-labeling and client configuration**
- The code can start locally during Phase 1, against sample brand JSON.
  Staging verification and backend integration need Phase 1 to be finished.
- Add slug-based routing in the shell (router basename `/<slug>`), with
  the neutral "store not found" page for unknown slugs.
- Build a config loader and cache in `smooth-web-data-mfe`
  (stale-while-revalidate, slug-namespaced, version-stamp check).
- Apply the brand to the theme in `smooth-web-app`, reusing
  `createAppTheme` and `resolveBrandColors`.
- Define the `slug-brand.json` schema and version stamp with the backend,
  and add local fixtures for at least two clients.
- The backend publishes to staging `/configs/` and invalidates the file,
  using its own AWS identity.
- **Exit:**
  - Two slugs render with their own branding on staging, from brand JSON
    the backend published.
  - A change made by the backend appears after the cache refreshes.

**Phase 3: Release CI/CD**
- Builds on the Phase 1 staging pipeline and OIDC roles. All AWS access
  goes through the Phase 1c roles.
- Tags produce immutable versioned artifacts. The existing immutability
  check is kept.
- The release manifest covers the six target repositories.
- Releases are promoted per client, to a list of clients, or to all clients
  in one run, to `release` or `production`.
- Rollback points a client, or all clients, back to an earlier release ID.
  Nothing is rebuilt.
- Production promotion and rollback need the deployment-group gate.
- Every promotion and rollback is recorded (who, when, which release, which
  clients).
- **Exit:** a demonstrated promotion to two clients on different releases,
  followed by a one-run rollback.

**Phase 4: Gradual package and flow updates**
- **The UI is built as mockup designs arrive** (decided 2026-11-08). Each
  mockup that arrives (a screen or a flow) becomes its own SDD feature, with
  the mockup as its intake source:
  - Reusable, domain-free pieces go into `smooth-web-design-mfe` first
    (`TECH-152`).
  - The screens are built in the owning repository: core, payment, auth, or
    the shell.
  - The UI uses the client's theme, so every client's branding applies.
- Each flow or package upgrade goes through its own SDD feature.
- The `TECH-148` version freeze applies; changing a frozen version needs
  separate approval.
- The build tool is settled: Rspack 2 + Module Federation 2 (Change 3).

### Resolved questions (2026-11-08)

- **Q2.1 Config path:** `/configs/<slug>-brand.json`. All client configs
  sit under one dedicated `configs/` prefix. That prefix gets its own
  CloudFront cache behaviour and its own IAM scope for the backend writer,
  and it stays clear of the `/<slug>/*` app routes and the SPA fallback.
- **Q2.2 Unknown or missing slug:** show a neutral, unbranded Smooth "store
  not found" (404) page, with no client branding.
- **Q2.3 Reaching production:** promote an approved release ID.
  - Tags build immutable versioned artifacts.
  - Production only repoints client manifests to an already-tested release
    ID. Production never gets a fresh build.
  - This matches the existing `promote-to-production` mechanism.
- **Q2.4 Per-client pins:** they apply in both the `release` (sandbox)
  environment and `production`, so a client's exact promotion can be
  rehearsed before production.
- **Q2.5a Who changes per-client manifests:** the pipeline only, through its
  promote and rollback steps. The backend writes only
  `/configs/<slug>-brand.json`.
- **Q2.5b Production gate:** a manual pipeline run, restricted to a
  deployment group through Bitbucket deployment permissions. Promotion and
  rollback use the same gate.
- **Q2.6a Custom domains:** maybe later. Web Two stays path-based for now,
  but slug resolution must be pluggable, so a domain-to-slug mapping can be
  added later without a redesign.
- **Q2.6b Base URLs.** The current Web One URLs, for reference:

  | Environment | Current URL (example client) |
  | --- | --- |
  | staging | `https://staging-ordering.dev.smoothcommerce.tech/sweat-and-tonic/menu/` |
  | release (sandbox) | `https://sandbox-ordering.dev.smoothcommerce.tech/sweat-and-tonic/menu/` |
  | production | `https://smooth.menu/mary-browns/menu/` |

  For Web Two:
  - **Production keeps `https://smooth.menu/<slug>/`.**
  - **Staging and release (sandbox) get alternative hostnames,** similar to
    the current ones with a small difference, so Web Two can run alongside
    Web One.

  The user will provide the exact staging and sandbox hostnames later.
  Until then, specs and runbooks refer to them as `<staging-base-url>` and
  `<sandbox-base-url>`.

### Remaining open items

- Exact Web Two hostnames for staging and sandbox. The user will provide
  them. Needed by Phase 1 (staging deploy and CloudFront).
- Is there one AWS account per environment, or one shared account? This
  decides where the OIDC provider and roles live. Needed by Phase 1.
- How Web Two production on `smooth.menu` cuts over from, or coexists
  with, Web One on the same hostname. Decide this before Phase 3 reaches
  production.
- Whether `smooth-web-data-mfe/src/theme/brand-colors.ts` moves to the shell
  with the theme. Needed by Phase 2.
- How login sessions behave when one browser visits several client slugs.
  Needed by Phase 2.

---

## Change 3 — Rspack 2 + Module Federation 2 with a shared build config (decided 2026-11-11)

**Source:** user decisions (babak@smooth.tech), 2026-11-11.

### Decisions

| ID | Decision |
| --- | --- |
| AD-3.1 | **All six repositories move from Webpack 5 to raw Rspack 2** (`@rspack/core` 2.3.2). Rsbuild is not used, so the full Rspack config stays explicit and customizable. |
| AD-3.2 | **Module Federation 2** comes from `@module-federation/enhanced/rspack` (2.9.3), pinned to one version in every repository. Remotes emit `mf-manifest.json`. |
| AD-3.3 | **Remote URLs are resolved at runtime** by a Module Federation runtime plugin, which replaces `createPromiseRemote`. This is the base for the per-client release loader (AD-2.7). |
| AD-3.4 | **One shared build config,** `@smooth/web-build-config`, owns the base config, shared singletons, remote registry, and runtime plugin. Repository configs declare only repository-specific values. |
| AD-3.5 | **Node 22** and a `typecheck` gate (`tsc --noEmit`, `isolatedModules`) in every repository. |
| AD-3.6 | **This resolves the Vite-vs-Webpack baseline conflict:** the baseline is Rspack. |

### Where it sits

Phase 1, step 1b, before the OIDC, AWS, and staging-deploy steps, so the
staging pipeline is built once, on the final tooling.

### Documents updated

- `ARCHITECTURE.md` ("Build tooling and shared config", technology
  baseline, conflict note);
- `TECHNICAL_REQUIREMENTS.md` (`TECH-002`, `TECH-043`, `TECH-149`, and the
  new `TECH-153`–`TECH-157`);
- `ENGINEERING_STANDARDS.md` ("Build tooling (Rspack + Module Federation
  2)");
- the agent docs, templates, and `CLAUDE.md`.

### Open items

- The spec decisions `DEC-001` (shared-config location), `DEC-002` (move to
  the six-remote target set now), `DEC-003` (federated types now), and
  `ADEC-001` (host route mapping).
- A pre-existing conflict found during this work: `TECH-148` freezes
  versions (React 19.3.0, TS 7.0.2, MUI 9.4.0, RTK 2.12.0, react-redux
  9.3.0) that the repositories do not use (React 19.2.3, TS 5.9, MUI 7.3.6,
  RTK 2.11.2, react-redux 9.2.0). The migration keeps the installed
  versions. A separate decision is needed.

---

## Change 4 — Per-client deployment implemented (2026-11-12 to 2026-11-16)

**Source:** user decisions in session. Spec:
`specs/active/release-orchestration/` (approved 2026-11-16). Operational
description: `context/DEPLOYMENT.md`.

| ID | Decision |
| --- | --- |
| AD-4.1 | Builds are brand-free and environment-free. Branding is runtime data in `/configs/<slug>-brand.json`, in a separate assets bucket per environment (`/configs/*`, `/brand-assets/*`, `/global/*` behaviours). |
| AD-4.2 | Per-client **deployment** in all three environments; per-client **versions** only in sandbox and production. Staging is always the latest build. |
| AD-4.3 | Release model: shared, immutable **bundles** (sets of six versions), plus a per-client release history in `state/<env>/<slug>.json`. |
| AD-4.4 | Orchestration lives in the `smooth-web-release` repository (`clients.json`, 30 clients). Package pipelines only build and publish. |
| AD-4.5 | Per-client entry pages are `clients/<slug>/index.html`, served by the CloudFront Function `client-entry-rewrite` (Default behaviour, viewer request). In sandbox and production the release is inlined into the entry page. This supersedes AD-2.7's "entry page fetches the manifest". |
| AD-4.6 | Client selection in the Bitbucket form: the `CLIENT` dropdown (rings, `all`, each slug), generated from `clients.json`, with an optional `CLIENTS` list override. |
| AD-4.7 | Release state is committed to git, which needs `REPO_ACCESS_TOKEN`. AWS access is through OIDC (`V2WebOrderingDeploy`). |
| AD-4.8 | The 30 client brand files were generated from `web-ordering-client-config` and are live in staging. The brand service (DynamoDB + Lambda) is deferred. |
