# Requirements: Menu page (example from the Web Ordering 2.0 mockup)

## Document control

| Field | Value |
| --- | --- |
| Feature slug | `menu-page-example` |
| Status | `READY_FOR_SPEC` |
| Intake date | 2026-11-12 |
| Prepared by | Intake Agent (run inline by Claude) |
| Human intake approval | `APPROVED` (2026-11-12) |

## Sources

| Source ID | Type | Title | Reference | Notes |
| --- | --- | --- | --- | --- |
| `SRC-001` | Developer-provided | Figma "Web Ordering 2.0 \| Redesign", frame "01 · Menu · full overview" | `figma.com/design/DLwiUNvFOkbOOHFwqfxtGs`, node `381-43598` (screenshot in session, 2026-11-12) | Desktop layout only |
| `SRC-002` | Developer-provided | User request: build the frame in core under Menu, reachable at `/menu`, as an example to verify the CI/CD flow | User session, 2026-11-12 | "let us deploy core to see it works" |
| `SRC-003` | Execution plan | Phase 4: UI is built as mockups arrive; `TECH-152` placement rules | `context/EXECUTION_PLAN.md` | — |

## Business objective

Prove the end-to-end flow on staging with a realistic screen: mockup → spec →
components in the correct repositories → staging deploy → page visible at
`/menu`. This is an **example**: it uses static data, not live APIs.

## Scope

**In scope:** the content area of the frame, which is everything below the
app bar:

- the pickup bar ("Pick Up · Bayview Village Mall · 2901 Bayview Ave · ASAP"
  and "Change");
- the "Browse" navigation (Rewards & Offers, Featured, Order Again,
  Favorites);
- the "Rewards & Offers" row: a points-balance card, a Rewards card, and an
  Offers card;
- the Favorites empty state;
- the Featured product grid (image, favourite heart, name, tag, price, kcal,
  add button);
- the "Your Order" panel (location, items, quantity stepper, subtotal,
  "Review Order").

**Out of scope:**

- the app-bar header (logo, Menu/Locations, language, Sign In, cart). It is
  owned by the shell (`smooth-web-app`), see `GAP-001`;
- mobile and tablet frames;
- real APIs, cart persistence, authentication, checkout, and navigation
  behind "Change" or "Review Order";
- white-label theming beyond the MUI theme the shell already provides.

## Functional requirements

- **REQ-001:** `/menu` renders the menu page from `smooth-web-core-mfe`
  (`features/menu`) inside the host, without signing in (`SRC-002`).
- **REQ-002:** the page shows each section listed in scope, laid out as in
  `SRC-001` at desktop width (≥ 1200 px): a 3-column layout (browse nav,
  content, order panel).
- **REQ-003:** selecting a "Browse" entry scrolls to its section and marks it
  as selected.
- **REQ-004:** the Featured grid lists five example products with name, tag,
  price, and kcal (`SRC-001` values).
- **REQ-005:** the "+" button on a product adds one of it to "Your Order".
  The panel's quantity stepper increments and decrements (removing at 0),
  and the subtotal updates. Local state only.
- **REQ-006:** the heart on a product toggles it as a favourite. Favorites
  shows the empty state until something is favourited, then lists the
  favourites (local state only).
- **REQ-007:** the Favorites empty state reads "You Have Not Added Any
  Favorites / Tap the heart on an item to save it here."
- **REQ-008:** every colour, font, and radius comes from the shell's MUI theme
  (`TECH-147`), so client branding applies.

## Gaps and assumptions

| ID | Kind | Finding | Impact |
| --- | --- | --- | --- |
| `GAP-001` | Conflict | The mockup includes the app-bar header, but the app bar belongs to `smooth-web-app` (Change 1). | The header is excluded here. It becomes a separate shell feature. |
| `GAP-002` | Conflict | The host sends unauthenticated visitors on `/menu` to `/login` (`unauthenticated-routes.tsx` `protectedSegments`). | Showing `/menu` without signing in needs a small host route change and a host deploy (`DEC-001`). |
| `GAP-003` | Missing | No menu API or contract exists yet. | Static fixtures are used (`POC-001`). |
| `ASM-001` | Assumption | Product images are placeholders; no image assets were provided. | Visual only. |

## Applicable technical requirements

`TECH-142`–`TECH-147` and `TECH-152` (component placement, Storybook, and
theme), plus `TECH-153`–`TECH-157` (build).

## Human intake approval

- **Decision:** `APPROVED`
- **Reviewer:** babak@smooth.tech
- **Date:** 2026-11-12
- **Comments:** All five documents approved together. DEC-001 = A: deploy design, core, and host.
