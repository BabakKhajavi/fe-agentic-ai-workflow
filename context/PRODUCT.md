# Product Context

## Target application: Smooth Commerce web ordering

`smooth-web` is the web ordering surface of **Smooth Commerce**, a white-label,
multi-tenant digital ordering, marketing, and loyalty platform for
restaurants (single-location operators through franchise/multi-unit and
enterprise restaurant groups, plus adjacent verticals such as hotels,
convenience stores, and venues).

Each restaurant brand ("client") gets its own branded ordering storefront —
menu, look and feel, and operational settings — running on the same shared
platform rather than a separate codebase per client. This is reflected in how
`smooth-web` is decomposed into independently deployed micro-frontends behind
a Module Federation host (see `ARCHITECTURE.md` for the technical structure
and the unresolved Vite-vs-MFE baseline conflict):

- `smooth-web-app` — host shell that composes the client experience.
- `smooth-web-menu-mfe` — menu browsing and ordering.
- `smooth-web-design-mfe` — per-client storefront branding/presentation.
- `smooth-web-data-mfe` — data/reporting surfaces.
- `smooth-web-settings-mfe` — client/storefront configuration.
- `smooth-web-auth-mfe`, `smooth-web-dashboard-mfe` — declared as remotes by
  the host but not present locally as of 2026-09-02; do not assume their
  scope without confirmation.

**Assumption, not yet confirmed:** the mapping above from repo name to
product capability is inferred from repository names and directory
structure only. It has not been confirmed against a product owner or
Confluence source and must be treated as a labeled assumption until
verified.

Publicly, Smooth Commerce (smooth.tech) describes itself as an all-in-one
digital ordering, customer marketing, and loyalty platform: pickup and
delivery order management with POS integration, payment processing, a
loyalty engine, upsell/suggestive selling, and a merchant portal for
analytics and reporting. It positions itself as white-label so each
restaurant brand's storefront reflects that brand's identity rather than a
generic marketplace. (Source: smooth.tech public marketing pages, retrieved
2026-09-02 — not an authenticated product source; treat as background, not
as a substitute for Confluence or a product owner.)

## Product (this workspace)

This workspace is an AI-assisted, Spec-Driven Development workflow for
building features in `smooth-web`, the Smooth Commerce web ordering
application described above.

## Audience

The primary audience is Chief of Staff and executive stakeholders evaluating
the practical value, governance, and transparency of agentic coding.

## Problem

Product requirements commonly begin in Confluence, while implementation work
is distributed across planning documents, Jira issues, source code (across
five independently versioned `smooth-web-*` repositories), and reviews.
Manual handoffs can lose context and make it difficult to prove that
delivered software matches the original intent — and multi-repo,
multi-tenant delivery makes that harder still, since a feature may need to
be scoped correctly to one repo (or coordinated across several) without
breaking other clients' storefronts.

## Proposed solution

Use specialized Claude agents to:

1. Read a Confluence product specification.
2. Normalize it into local, reviewable SDD artifacts.
3. Analyze the relevant `smooth-web-*` repository (or repositories) and
   produce an impact assessment and plan.
4. Pause for human approval.
5. Convert the approved plan into a Jira epic and implementation tasks.
6. Implement tasks in the target `smooth-web-*` repository (or
   repositories).
7. Review traceability from requirements through code.
8. Pause for final human review.

## Demonstration goals

- Show that agents can move from business intent to working software.
- Keep humans in control of consequential decisions.
- Make every implementation decision traceable.
- Demonstrate repeatable, role-based agent collaboration.
- Make ambiguity and risk visible before code is written.
- Show that a feature can be scoped correctly within a multi-repo,
  multi-tenant Module Federation application without unintended impact on
  other micro-frontends or restaurant clients.

## Non-goals

- Fully autonomous production deployment.
- Replacing product, design, architecture, security, or engineering
  approval.
- Treating generated specifications or code as correct without
  verification.
- Automatically resolving contradictory or incomplete requirements.

## Success criteria

- A Confluence requirement becomes a complete local SDD package.
- A human can review and approve the package before implementation begins.
- Approved work is represented in Jira with requirement traceability.
- Claude implements the feature in the correct `smooth-web-*` repository
  (or repositories), consistent with the multi-tenant Module Federation
  architecture.
- Automated checks and human POC review verify the result.
- An executive viewer can follow the complete lifecycle without needing to
  read source code.
