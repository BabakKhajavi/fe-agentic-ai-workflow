# Impact: Menu page (example)

| Field | Value |
| --- | --- |
| Feature slug | `menu-page-example` |
| Repositories inspected | core, design, and app on the `staging` branch (2026-11-12) |
| Human plan approval | `APPROVED` (2026-11-12) |

## Baseline

- **Core:**
  - `routes-config.tsx` has only a `Placeholder` at the index;
  - `features/{menu,offers,locations,orders}` are empty;
  - `Layout` renders design-mfe `PageHeader` and an outlet.
- **Design:** exports `SimpleSelectableList`, `ElevatedCard`,
  `OptimizedImage`, `NavigableCarousel`, and `ContainedButton`. It has no
  stepper and no empty-state component.
- **Host:**
  - `/menu/*` is mounted **only** inside the authenticated routes;
  - for unauthenticated visitors, `protectedSegments` redirects `/menu` to
    `/login`.
- **Staging:** `/` returns 200, and `/menu` falls back to the root
  `index.html` through the S3 error document (status 404, body is the SPA).

## Affected areas

| ID | Action | Path |
| --- | --- | --- |
| `IMP-001` | Create | design `src/molecules/quantity-stepper/{quantity-stepper.tsx,quantity-stepper.stories.tsx,index.ts}` and `src/molecules/empty-state/…`; export them from `src/molecules/index.ts` |
| `IMP-002` | Create | core `src/features/menu/{pages,components,hooks,fixtures,__tests__}`; `features/offers/components`, `features/locations/components`, `features/orders/components` |
| `IMP-003` | Modify | core `src/routes/routes-config.tsx`: add the `menu` route (the existing index placeholder is kept) |
| `IMP-004` | Modify | host `src/routes/unauthenticated-routes.tsx`: drop `AppPaths.Menu` from `protectedSegments`, and mount `smooth_core/CoreApp` at `/menu/*` for visitors |

## Risks

- **`RISK-001`:** the Layout's `PageHeader` and `marginTop` don't match the
  mockup. MenuPage uses its own full-width layout under the same outlet.
- **`RISK-002`:** the host typecheck uses `any` stubs, so core's exports
  aren't type-checked from the host side (known, from the previous feature).
- **`RISK-003`:** the `/menu` deep link returns HTTP 404 with the SPA body
  until CloudFront maps 403/404 to `/index.html` with 200. It works in
  browsers.

## Human plan approval

- **Decision:** `APPROVED`
- **Reviewer:** babak@smooth.tech
- **Date:** 2026-11-12
- **Comments:** All five documents approved together. DEC-001 = A: deploy design, core, and host.
