# Specification: Menu page (example)

## Document control

| Field | Value |
| --- | --- |
| Feature slug | `menu-page-example` |
| Status | `READY_FOR_PLANNING` |
| Human spec approval | `APPROVED` (2026-11-12) |

## Component placement (`TECH-142`, `TECH-152`)

| Piece in the mockup | Component | Repository / path | Why there |
| --- | --- | --- | --- |
| Quantity stepper (bin or −, count, +) | `QuantityStepper` (new) | `smooth-web-design-mfe/src/molecules/quantity-stepper/` | Generic and domain-free; reused by cart and checkout later |
| Empty state (icon, title, hint) | `EmptyState` (new) | `smooth-web-design-mfe/src/molecules/empty-state/` | Generic and domain-free |
| Browse nav | `SimpleSelectableList` (reuse) | design-mfe `molecules/list` | Already exists |
| Card surfaces | `ElevatedCard` (reuse) | design-mfe `atoms/cards` | Already exists |
| Product image | `OptimizedImage` (reuse) | design-mfe `atoms/images` | Already exists |
| Rewards and offers row | `NavigableCarousel` (reuse) | design-mfe `molecules/carousel` | Already exists |
| "Review Order" button | `ContainedButton` (reuse) | design-mfe `atoms/buttons` | Already exists |
| Page and section composition | `MenuPage`, `MenuBrowseNav`, `FeaturedSection`, `FavoritesSection` | `smooth-web-core-mfe/src/features/menu/` | Menu domain |
| Product card | `ProductCard` | `core-mfe/src/features/menu/components/` | Domain-named, so not allowed in design-mfe (`TECH-152`) |
| Points-balance, rewards, and offer cards | `RewardsBalanceCard`, `RewardsOffersSection` | `core-mfe/src/features/offers/components/` | Offers domain (Change 1) |
| Pickup bar | `PickupLocationBar` | `core-mfe/src/features/locations/components/` | Locations domain |
| "Your Order" panel | `OrderSummaryPanel` | `core-mfe/src/features/orders/components/` | Orders domain |
| Example data and cart state | `useMenuExample()` hook and fixtures | `core-mfe/src/features/menu/hooks/`, `.../fixtures/` | `POC-001`; replaced later by `smooth-web-data-mfe` `apis/menu` |
| `/menu` route | core `routes-config.tsx`: `menu` → `MenuPage` | core | — |
| Public `/menu` | host `unauthenticated-routes.tsx` | `smooth-web-app` | `DEC-001` |

## Acceptance criteria

- **AC-001** (`REQ-001`, `REQ-002`): opening
  `https://smooth-web-ordering-staging.dev.smoothcommerce.tech/menu` while
  signed out shows the menu page, with every in-scope section, laid out in 3
  columns at desktop width. Manual check on staging.
- **AC-002** (`REQ-003`): clicking "Featured" scrolls to Featured and marks it
  as selected. Manual check.
- **AC-003** (`REQ-004`, `REQ-005`):
  - "+" on Butter Croissant adds it to "Your Order";
  - "+" on the stepper changes it to 2 and the subtotal to $25.98;
  - reducing to 0 removes the item.

  Automated test on the cart logic, and manual check.
- **AC-004** (`REQ-006`, `REQ-007`): Favorites shows the empty-state text;
  after the heart is clicked, the item appears. Automated and manual.
- **AC-005** (`REQ-008`): no hard-coded colours in the new components; they
  use theme tokens. Review.
- **AC-006:** `QuantityStepper` and `EmptyState` have Storybook stories
  (`TECH-143`). **Gap:** Storybook isn't set up in design-mfe yet, so the
  story files are added but cannot run (`POC-003`).
- **AC-007:** typecheck and build pass. Pushing to `staging` deploys the
  changed repositories, and the page works from the deployed staging remotes.

## POC shortcuts

| ID | Shortcut | Follow-up |
| --- | --- | --- |
| `POC-001` | Static fixtures and local React state, instead of `smooth-web-data-mfe` APIs and a global cart slice | A `menu` API slice in data-mfe, and a cart slice |
| `POC-002` | Placeholder product images | Real assets or CDN |
| `POC-003` | Storybook stories written, but no Storybook runner | Set up Storybook in design-mfe |

## Open decisions

| ID | Decision | Options | Recommendation |
| --- | --- | --- | --- |
| `DEC-001` (RESOLVED 2026-11-12: A) | What gets deployed, given "deploy only core" | **A:** deploy design + core + host. This is correct placement, and `/menu` is public. **B:** core only. Generic pieces are built in core temporarily (breaks `TECH-142`), and `/menu` is only visible after signing in. | **A.** Core alone cannot make `/menu` public or hold generic components. Design and host get one small change each. |

## Human spec approval

- **Decision:** `APPROVED`
- **Reviewer:** babak@smooth.tech
- **Date:** 2026-11-12
- **Comments:** All five documents approved together. DEC-001 = A: deploy design, core, and host.
