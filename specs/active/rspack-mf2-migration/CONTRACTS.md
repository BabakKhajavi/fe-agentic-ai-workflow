# Contracts: Rspack and Module Federation 2 migration with a shared build config

## Document control

| Field | Value |
| --- | --- |
| Feature slug | `rspack-mf2-migration` |
| Status | `READY_FOR_PLANNING` |
| Integration mode | `NONE`. Build tooling only; no API changes. |
| Human spec approval | `APPROVED` (2026-11-12) |

Route, form, API, and global-state contracts: **Not needed**. Behaviour is
unchanged.

## Build-tooling contracts

### CON-001 — Shared config package API

Package `@smooth/web-build-config`, repository `smooth-web-build-config`
(`DEC-001` A). The package is ESM.

```ts
// @smooth/web-build-config
export type RemoteKey = 'auth' | 'design' | 'data' | 'core' | 'payment';
export type AppEnv = 'local' | 'development' | 'staging' | 'release' | 'production';

export interface SmoothRspackOptions {
  dirname: string;                       // the repository root (import.meta.dirname)
  kind: 'host' | 'remote';
  name: string;                          // federation name, e.g. 'smooth_core', or 'host'
  port: number;                          // dev-server port (see CON-004)
  entry?: string;                        // default './src/index.tsx'
  exposes?: Record<string, string>;      // federation exposes, unchanged keys
  remotes?: RemoteKey[];                 // remotes this repository consumes
  html?: false | { template: string; templateParameters?: Record<string, string> };
  define?: Record<string, string>;       // values for DefinePlugin (JSON-stringified by the package)
  publicPath?: string;                   // the host passes its own value; remotes default to 'auto'
  css?: boolean;                         // built-in CSS support (design-mfe)
}

export function createSmoothRspackConfig(opts: SmoothRspackOptions): import('@rspack/core').Configuration;
export const SHARED_SINGLETONS: Record<string, object>;   // CON-003
export const REMOTES: Record<RemoteKey, RemoteDefinition>; // CON-004
export function resolveRemoteEntry(key: RemoteKey, env: AppEnv, manifest?: Record<string, string> | null, origin?: string, buildVersion?: string): string; // CON-006
export function getAppEnv(): AppEnv;   // from NODE_ENV; the same semantics as today
```

What the base config owns:

- **Compilation:** `builtin:swc-loader` (TSX, automatic React runtime;
  `refresh` in development).
- **Dev reloading:** `@rspack/plugin-react-refresh` in development.
- **Filenames:** `[name].[contenthash:8].js` in production.
- **Source maps:** `source-map` in production, `eval-source-map` in
  development.
- **Resolution:** the `@` alias to `src`, plus the existing
  `@mui/material` / `@mui/system` aliases.
- **CSS:** built-in when `css: true`.
- **HTML:** `HtmlRspackPlugin` when `html` is set.
- **Federation:** the MF 2 `ModuleFederationPlugin` with `SHARED_SINGLETONS`,
  `manifest: true`, and the runtime plugin from `CON-006`.
- **Dev server:** remote CORS headers, `historyApiFallback`, and `hot`.
- **Optimization:** the host keeps its current `optimization` block, unless
  the build check in `IMPACT.md` `RISK-002` forces a change.

### CON-002 — Repository config shape

Each repository's `rspack.config.js` contains only repository-specific values:

```js
import { createSmoothRspackConfig } from '@smooth/web-build-config';
export default createSmoothRspackConfig({
  dirname: import.meta.dirname,
  kind: 'remote',
  name: 'smooth_core',
  port: 9007,
  exposes: { './CoreApp': './src/routes/CoreRoutes.tsx' },
  remotes: ['design', 'data'],
  html: { template: './public/index.html' },
  define: { 'process.env.SMOOTH_ASSET_URL': process.env.SMOOTH_ASSET_URL ?? '' },
});
```

`package.json` scripts in every repository:

| Script | Command |
| --- | --- |
| `start` | `rspack serve` (with the existing `NODE_ENV` / `APP_ENV` prefixes) |
| `build` | `rspack build` |
| `typecheck` | `tsc --noEmit` |

### CON-003 — Shared singletons (unchanged values)

| Package | Settings |
| --- | --- |
| `react`, `react-dom` | `singleton`, `strictVersion`, `requiredVersion: '19.2.x'`; `import: false` in remotes |
| `react-router-dom` | singleton, `7.x.x` |
| `react-redux` | singleton, `9.x.x` |
| `@reduxjs/toolkit` | singleton, `2.x.x` |
| `@mui/material`, `@mui/system` | singleton, `7.x.x` |
| `@emotion/react`, `@emotion/styled` | singleton, `11.x.x` |
| `rxjs` | singleton, `7.x.x` |

`shareStrategy` is **`'loaded-first'`**. This was amended on 2026-11-12,
during `TASK-001`; the approved text said `'version-first'`.

- **Why:** the MF 2 runtime source (`runtime-core/dist/shared/index.js`)
  shows that `version-first` makes `initializeSharing` load the entry of
  every registered remote eagerly.
  - That can happen before the host's pre-bootstrap script has fetched
    `/smooth-web-app/manifest.json`, so per-client version pins (`CON-006`)
    would be ignored.
  - It would also download all remotes on startup.
- **Effect:** `loaded-first` initialises remotes lazily on their first
  request, which matches today's MF 1 behaviour. The singletons and
  `strictVersion` rules are unchanged, so the version checks still apply.

### CON-004 — Remote registry

The target set applies if `DEC-002` is accepted.

| Key | Federation name | Repository | Local port | Exposes (consumed by host) |
| --- | --- | --- | --- | --- |
| `auth` | `smooth_auth` | `smooth-web-auth-mfe` | 9001 | `./AuthApp` |
| `design` | `smooth_ui` | `smooth-web-design-mfe` | 9002 | `./atoms`, `./molecules`, `./organisms`, `./templates` |
| `data` | `smooth_data` | `smooth-web-data-mfe` | 9003 | `./Store`, `./AuthApi`, `./BrandApi`, `./Auth`, `./Brand`, `./Role`, `./UiState` |
| `payment` | `smooth_payment` | `smooth-web-payment-mfe` | 9005 | `./PaymentApp` |
| `core` | `smooth_core` | `smooth-web-core-mfe` | 9007 | `./CoreApp` |

The host itself runs on port 9000.

Environment base URLs per repository:

| Environment | Base URL |
| --- | --- |
| `local` | `http://localhost:<port>` |
| `staging` | `https://stage.app.smooth.tech/<repo>` |
| `release` | `https://release.app.smooth.tech/<repo>` |
| `production` | `https://app.smooth.tech/<repo>` |

These are the existing values (`ASM-001` / `POC-002`) and will be replaced
in Phase 1c.

### CON-005 — Remote entry and manifest

- Every remote build emits `dist/remoteEntry.js` and `dist/mf-manifest.json`.
- Consumers reference remotes as `<name>@<base>/mf-manifest.json`
  (`ASM-002`).

### CON-006 — Remote-resolver runtime plugin

The plugin is `@smooth/web-build-config/runtime/remote-resolver`, an MF 2
`ModuleFederationRuntimePlugin`:

- **`beforeRequest`** (amended 2026-11-12 from `beforeInit`, for the same
  timing reason as `CON-003`): on a remote's first request, it replaces the
  entry with
  `resolveRemoteEntry(key, env, window.__SMOOTH_RELEASE_MANIFEST__,
  location.origin, window.__SMOOTH_BUILD_VERSION__)`.
- **Resolution rules:**
  1. If the manifest has a version for the key, the entry is
     `<origin>/<repo>/<version>/mf-manifest.json`.
  2. Otherwise, the entry is `<envBase>/mf-manifest.json`.
  3. `?v=<buildVersion>` is appended when a build version is set.
     `Date.now()` is not used, so CDN caching is no longer defeated.
- **`errorLoadRemote`:** logs `Failed to load remote <name> from <url>` and
  rethrows.
- **Manifest keys:** `auth`, `design`, `data`, `core`, `payment` (the same
  keys as the release manifest).

### CON-007 — Runtime and type-check baseline

- `.nvmrc` contains `22`.
- `package.json` has `"engines": { "node": ">=22.12" }`.
- `tsconfig.json` has `"isolatedModules": true`.

### CON-008 — Federated types (`DEC-003`)

- Remotes set `dts: { generateTypes: true }`; consumers set
  `dts: { consumeTypes: true }`.
- The generated `@mf-types` folder replaces the `declare module 'smooth_*/…'`
  stubs in the consumers' `src/types/mfe.d.ts`.
- `@mf-types` is git-ignored and generated by dev and build.

## Contract traceability

| Contract | Requirements | Acceptance criteria |
| --- | --- | --- |
| `CON-001` | `REQ-001`, `REQ-002`, `REQ-003`, `REQ-007`, `REQ-008` | `AC-001`, `AC-005`, `AC-007` |
| `CON-002` | `REQ-001`, `REQ-007` | `AC-005` |
| `CON-003` | `REQ-004` | `AC-002`, `AC-006` |
| `CON-004` | `REQ-004`, `REQ-006` | `AC-002`, `AC-006` |
| `CON-005` | `REQ-005`, `REQ-006` | `AC-001`, `AC-002` |
| `CON-006` | `REQ-006` | `AC-004` |
| `CON-007` | `REQ-009`, `REQ-010` | `AC-001`, `AC-008` |
| `CON-008` | `REQ-012` | `AC-008` |

## Open contract decisions

None open. `DEC-001`–`DEC-003` were resolved on 2026-11-12 (see `SPEC.md`): A, Yes, and Yes. The target remote set in `CON-004` applies.

## Human spec approval

- **Decision:** `APPROVED`
- **Reviewer:** babak@smooth.tech
- **Date:** 2026-11-12
- **Comments:** All five documents were approved together, with DEC-001 A, DEC-002 Yes, ADEC-001 proposal, and DEC-003 Yes. Jira was skipped for this tooling feature at the user's request; traceability is through this spec and the developer reports.
