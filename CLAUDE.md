# AI-Assisted SDD Workflow

## Purpose

This workspace demonstrates how Claude agentic coding can turn an approved
Confluence product specification into a working React application by following
Spec-Driven Development (SDD).

The demonstration is intended for Chief of Staff and executive stakeholders.
The workflow must therefore remain understandable, traceable, and reviewable—not
just produce code quickly.

## Workspace layout

- `context/` contains stable product, technical requirements, engineering
  standards, and architecture context.
- `agents/` defines the responsibility and boundaries of each agent role.
- `workflows/` defines end-to-end orchestration.
- `specs/active/<feature>/` contains the active feature's SDD artifacts.
- `templates/` contains reusable templates for generated artifacts.
- `../smooth-web/` contains the React application, split across sibling
  repositories:
  - `smooth-web-app` — host shell (Webpack + Module Federation).
  - `smooth-web-data-mfe`, `smooth-web-design-mfe`, `smooth-web-menu-mfe`,
    `smooth-web-settings-mfe` — remote micro-frontends.
  - The host also declares `smooth-web-auth-mfe` and
    `smooth-web-dashboard-mfe` as remotes; these were not present locally as
    of 2026-09-02. Confirm with the user before assuming they don't exist.
  - Each subdirectory is its own git repository. A feature task typically
    touches only the repo(s) named in its Jira issue — do not assume changes
    span all five.

Before planning or implementing application changes, read:

- `context/PRODUCT.md`
- `context/TECHNICAL_REQUIREMENTS.md`
- `context/ENGINEERING_STANDARDS.md`
- `context/ARCHITECTURE.md`
- `context/JIRA.md` when reading or writing Jira work

## Sources of truth

1. Product intent originates from Confluence, a Jira ticket, or a
   developer-provided requirement description, depending on how the feature
   was intaken.
2. The local SDD artifacts are the source of implementation truth.
3. Jira tracks implementation work only after the local plan is approved.
4. The `smooth-web` repositories contain the implementation produced from the
   approved specification and plan.

If these sources disagree, stop and ask for human resolution. Do not silently
choose one source or invent missing requirements.

## Required feature artifacts

Every feature must have its own directory under `specs/active/` containing:

- `REQUIREMENTS.md` — normalized, testable requirements and source references.
- `SPEC.md` — behavior, user flows, acceptance criteria, and constraints.
- `CONTRACTS.md` — interfaces, data shapes, APIs, and integration contracts.
- `IMPACT.md` — affected components, dependencies, risks, and migrations.
- `PLAN.md` — ordered implementation and verification plan.

## Workflow gates

The required sequence is:

1. Intake
2. Specification
3. Architecture and planning
4. Human approval
5. Jira creation
6. Development
7. Human POC review
8. Done

No Jira issue may be created and no implementation may begin until a human has
explicitly approved `REQUIREMENTS.md`, `SPEC.md`, `CONTRACTS.md`, `IMPACT.md`, and
`PLAN.md`.

## Agent rules

- Preserve traceability from source requirement to specification, plan, Jira
  issue, code, and verification evidence.
- Clearly label assumptions, ambiguities, exclusions, and unresolved questions.
- Never fabricate Confluence content, Jira identifiers, user decisions, API
  contracts, or repository behavior.
- Prefer small, reviewable changes.
- Inspect the React repository before proposing architectural changes.
- Do not modify generated SDD artifacts owned by an earlier stage without
  recording why the change is necessary.
- Stop at every required human-approval gate.

## Definition of done

A feature is done only when:

- Approved requirements have corresponding acceptance criteria.
- Jira work maps back to the approved plan.
- The implementation maps back to the specification and contracts.
- Relevant automated checks pass.
- The Developer Report records traceability and verification evidence.
- A human accepts the demonstrated implementation.
