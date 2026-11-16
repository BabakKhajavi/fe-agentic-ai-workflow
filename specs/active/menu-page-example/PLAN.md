# Plan: Menu page (example)

| Field | Value |
| --- | --- |
| Feature slug | `menu-page-example` |
| Based on | REQUIREMENTS, SPEC, CONTRACTS, IMPACT |
| Human plan approval | `APPROVED` (2026-11-12) |

## Tasks

1. **TASK-001 (design):** create `QuantityStepper` and `EmptyState` in
   `molecules`, using theme tokens only, with story files.
   - Covers `AC-005` and `AC-006`.
   - Verify: typecheck and build.
2. **TASK-002 (core):**
   - create `features/menu` (page, browse nav, featured grid, product card,
     favourites section, fixtures, `useMenuExample`, with cart-logic tests
     using `node --test` on the pure functions);
   - create `features/offers`, `features/locations`, and `features/orders`
     components;
   - add the `menu` route.
   - Covers `AC-001` to `AC-005`.
   - Verify: typecheck, tests, build, then local run through the host at
     `localhost:9000/menu`.
3. **TASK-003 (host, `DEC-001` A):** make `/menu/*` public by mounting
   `smooth_core/CoreApp` for visitors.
   - Verify: typecheck, build, then signed out locally, `/menu` renders the
     page.
4. **TASK-004 (deploy):** push `staging` in design, core, and app.
   - Verify `AC-001` and `AC-007` on
     `https://smooth-web-ordering-staging.dev.smoothcommerce.tech/menu`, in
     the browser.

## Final verification

Typecheck and build in all three repositories; cart-logic tests; local
browser check; staging browser check. Results go in `DEVELOPER_REPORT.md`.

## Human plan approval

- **Decision:** `APPROVED`
- **Reviewer:** babak@smooth.tech
- **Date:** 2026-11-12
- **Comments:** All five documents approved together. DEC-001 = A: deploy design, core, and host.
