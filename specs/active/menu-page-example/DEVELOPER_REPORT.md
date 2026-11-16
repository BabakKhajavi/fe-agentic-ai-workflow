# Developer Report: menu-page-example / TASK-001 to TASK-004

This is an execution report for the review handoff, not an approved SDD
artifact.

## Result

- **Feature:** `menu-page-example`
- **Jira:** none (the approved scope was built directly on `staging`, at the
  user's request).
- **Status:** `READY_FOR_REVIEW`. Verified on staging.
- **Commits on `staging`:**

  | Repository | Commit |
  | --- | --- |
  | design-mfe | `adfd7fd` |
  | core-mfe | `5176c55` |
  | app | `9bdf8cc` |

## Traceability

| Requirement | Acceptance criteria | Where |
| --- | --- | --- |
| `REQ-001` | `AC-001` | host `src/routes/unauthenticated-routes.tsx`: public `/menu/*` → `smooth_core/CoreApp`; core `src/routes/routes-config.tsx`: index → `MenuPage` |
| `REQ-002` | `AC-001` | core `features/menu/pages/menu-page.tsx`: 3-column grid, from 180 px nav \| content \| 300 px panel down to stacked on small screens |
| `REQ-003` | `AC-002` | `features/menu/components/menu-browse-nav.tsx` (reuses `SimpleSelectableList`) and `scrollTo` |
| `REQ-004`, `REQ-005` | `AC-003` | `features/menu/hooks/cart.ts` and `use-menu-example.ts`; `features/orders/components/order-summary-panel.tsx` with `QuantityStepper` |
| `REQ-006`, `REQ-007` | `AC-004` | `features/menu/components/favorites-section.tsx` with `EmptyState` |
| `REQ-008` | `AC-005` | Theme tokens only (`primary.*`, `alpha(theme.palette…)`, `divider`, `background.*`); no hex values in the new components |
| — | `AC-006` | design `molecules/quantity-stepper` and `molecules/empty-state` with `*.stories.tsx` (plain CSF) |

## Component placement (as built)

| Repository | Components |
| --- | --- |
| **design-mfe** (domain-free) | `QuantityStepper`, `EmptyState` (new). Reused: `SimpleSelectableList`, `OptimizedImage`, `ContainedButton`. |
| **core `features/menu`** | `MenuPage`, `MenuBrowseNav`, `MenuSection`, `ProductGrid`, `ProductCard`, `ProductImage`, `FavoritesSection`, fixtures, `useMenuExample`, `cart.ts` |
| **core `features/offers`** | `RewardsBalanceCard`, `RewardsOffersRow` |
| **core `features/locations`** | `PickupLocationBar` |
| **core `features/orders`** | `OrderSummaryPanel` |
| **core `modules/icons`** | 6 Material-path `SvgIcon`s, so no new icon dependency is needed |
| **host** | the public `/menu/*` route only |

## Verification

| Check | Result |
| --- | --- |
| `npm run typecheck` | design 0, core 0 (against real federated types, including the new molecules), host 0 |
| `npm test` (core, `node --test`, type stripping) | 7/7 passed: add, increment, $25.98 at quantity 2, remove at 0, unknown product, favourite toggle, mockup initial $12.99 |
| CI simulation (fresh clone of `staging`: `npm ci`, typecheck where enabled, staging build) | design ✓, core ✓ (typecheck skipped in CI, RISK-006), host ✓ |
| Local browser, `localhost:9000/menu`, signed out | Renders every in-scope section with the shell theme; no console errors. "+" on Butter Croissant gives 2 items and **$25.98**, and the stepper switches from bin to "−". The heart on Almond Danish moves it into Favorites. |
| Staging browser, `https://smooth-web-ordering-staging.dev.smoothcommerce.tech/menu`, signed out | ✓ Renders every section; no console errors. All 5 remotes resolve **same-origin** (`<staging domain>/<repo>/mf-manifest.json`). "+" gives 2 items and **$25.98**. The document status is **404**, because S3 serves the SPA through its error document (`RISK-003`); fix it with a CloudFront error response 403/404 → `/index.html` 200. |

## Deviations

1. **The route differs from CON-001.** Core's **index** renders `MenuPage`;
   the plan said an internal `menu` route. The host mounts CoreApp at
   `/menu/*`, so inside core `/menu` is the index.
2. **The rewards row is a plain horizontal scroll row**, not
   `NavigableCarousel`. That component autoplays and needs fixed slide
   widths, which the static 3-card row in the mockup doesn't need. Revisit
   when the offers list becomes dynamic.
3. **Core's unit tests aren't in the pipeline yet.** The shared pipeline
   template has no test step, and the other repositories' `test` script
   exits 1. Add `npm test` to the template once every repository has a real
   test script.
4. **The header (app bar) is not included** (`GAP-001`). It is the shell's
   responsibility.

## Review focus

- Component placement against `TECH-142` and `TECH-152` (table above).
- `/menu` is now public. The authenticated `/menu/*` route still mounts the
  same CoreApp.

## Amendment (2026-11-13): per-client brand on the menu page

Requested in chat by the user: "apply this themes to and reflect them in the
menu page … push to staging". This follows Execution Plan AD-2.1 (slug in
the path) and the brand-service design (same-origin `/configs/<slug>-brand.json`).

| Change | Where |
| --- | --- |
| `/<slug>/menu` is a public route that mounts `smooth_core/CoreApp` | host `src/routes/unauthenticated-routes.tsx` |
| The pre-React script resolves the slug from `/<slug>/menu`, always preloads the brand on storefront pages, and fetches **`/configs/<slug>-brand.json`** from the same origin (it used to fetch `<brandAssetBaseUrl>/assets/brands/<slug>/brand.json` on the Web One host). `/menu` without a slug keeps the default Smooth brand. | host `public/index.html` |
| No change needed in data-mfe: `brand$` already prefers `window.__brandConfig` | — |

**Verified locally** with the two brand files served from `public/configs/`
(temporary, not committed):

| URL | Result |
| --- | --- |
| `/kinton/menu` | Kinton: dark canvas, yellow `#FFCD00` buttons, chips, and nav; `Archivo` font |
| `/mary-browns/menu` | Mary Browns: light canvas, orange `#E35A1B`; `Work Sans`; `appName` "Mary Brown's Chicken" |
| `/menu` | Smooth default (`#165A50`) |

No console errors. Host typecheck 0 errors; staging build OK.

**Staging prerequisite:** the CloudFront behaviour must be `/configs/*`. It
was still `/config/*` on 2026-11-13, so `/configs/...` fell through to the
app bucket and returned 404.
