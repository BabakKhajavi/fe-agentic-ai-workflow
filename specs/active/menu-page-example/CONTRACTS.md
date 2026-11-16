# Contracts: Menu page (example)

| Field | Value |
| --- | --- |
| Feature slug | `menu-page-example` |
| Integration mode | `MOCK` (static fixtures) |
| Human spec approval | `APPROVED` (2026-11-12) |

## Route contract

- **CON-001:** host `/menu/*` mounts `smooth_core/CoreApp`. Core's internal
  route `menu` (relative to the host's `/menu`) renders `MenuPage`. With
  `DEC-001` A, `/menu/*` also renders for unauthenticated visitors.

## Component contracts (design-mfe, domain-free)

```ts
// smooth_ui/molecules
interface QuantityStepperProps {
  value: number;                 // current quantity (>= 0)
  min?: number;                  // default 0
  max?: number;
  onChange: (next: number) => void;
  removeAtMin?: boolean;         // shows a bin icon instead of "−" at value 1
  size?: 'small' | 'medium';
  'aria-label'?: string;
}
interface EmptyStateProps {
  icon?: React.ReactNode;
  title: string;
  description?: string;
  action?: React.ReactNode;
}
```

## Data shapes (core, `POC-001` fixtures)

```ts
interface MenuProduct {
  id: string;
  name: string;
  tag?: string;
  priceCents: number;
  kcal?: number;
  imageUrl?: string;
}
interface OrderLine {
  productId: string;
  quantity: number;
  description?: string;
}
interface PickupLocation {
  name: string;
  address: string;
  when: 'ASAP' | string;
}
```

## Hook boundary

- **CON-010:** `useMenuExample()` returns:
  - `{ products, location, rewards, favorites, toggleFavorite, lines, add, setQuantity, subtotalCents }`;
  - its cart logic is pure (`addLine`, `setLineQuantity`, `subtotal`) and
    unit-tested.

  Its feature name stays `menu`, so it can later be backed by `data-mfe`
  `apis/menu` (`TECH-130`–`TECH-135`).

## Human spec approval

- **Decision:** `APPROVED`
- **Reviewer:** babak@smooth.tech
- **Date:** 2026-11-12
- **Comments:** All five documents approved together. DEC-001 = A: deploy design, core, and host.
