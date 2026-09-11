# Feature SDD Workflow

## Stage 1 — Intake

1. A human supplies a requirement source: an authorized Confluence page URL
   or identifier, a Jira ticket key or URL, or a direct developer-written
   requirement description.
2. Run the role defined in `../agents/intake.md`.
3. The Intake Agent creates only:

   ```text
   ../specs/active/<feature-slug>/REQUIREMENTS.md
   ```

4. The agent reports `READY_FOR_SPEC`, `NEEDS_CLARIFICATION`, or `BLOCKED`.
5. A human reviews the source traceability, extracted requirements, assumptions,
   gaps, and questions.
6. The human records `APPROVED` or `CHANGES_REQUESTED` in the document.

The workflow must stop here unless the intake document records explicit human
approval. Approval at this stage authorizes specification work only; it does not
authorize Jira creation or changes to the React application.

## Stage 2 — Specification

1. Confirm `REQUIREMENTS.md` records human intake approval and has no unanswered
   blocking question.
2. Run the role defined in `../agents/spec-writer.md`.
3. The Spec Agent creates only:

   ```text
   ../specs/active/<feature-slug>/SPEC.md
   ../specs/active/<feature-slug>/CONTRACTS.md
   ```

4. The agent demonstrates requirement coverage through `AC-*` and `CON-*`
   traceability.
5. A human reviews behavior, contracts, POC shortcuts, and open decisions, then
   records the same decision in both documents.

The workflow must stop unless both documents record `APPROVED`. This approval
authorizes the Planner/Architect Agent to inspect the React repository and
prepare `IMPACT.md` and `PLAN.md`; it does not authorize implementation or Jira
creation.

## Stage 3 — Repository impact and planning

1. Confirm the requirements, specification, and contracts record the required
   human approvals and contain no blocking open decision.
2. Run the role defined in `../agents/planner.md`.
3. The Planner/Architect Agent inspects the relevant `../../smooth-web/`
   repository (or repositories) without modifying them.
4. The agent creates only:

   ```text
   ../specs/active/<feature-slug>/IMPACT.md
   ../specs/active/<feature-slug>/PLAN.md
   ```

5. A developer or technical lead reviews repository evidence, affected files,
   tasks, dependencies, risks, POC shortcuts, and verification coverage.
6. The reviewer records the same decision in both documents.

This is the main implementation approval gate. The workflow must stop unless
both documents record `APPROVED`. Only that approval may authorize downstream
Jira creation and implementation of the approved task scope.

## Stage 4 — Jira handoff

1. Confirm all five SDD documents record their required approvals.
2. Run the role defined in `../agents/jira.md`.
3. The Jira Agent inspects the target project and searches for existing feature
   and task markers before writing anything.
4. It presents one Epic and one child issue per approved `TASK-*` item using the
   structure in `../templates/JIRA_PREVIEW.template.md`.
5. A human confirms the exact Jira project and creation preview.
6. The agent creates or safely reuses the approved issues and reports the
   `TASK-*` to Jira-key mapping.

The Jira Agent must stop after the report. Jira creation does not authorize the
agent to modify React code. A human selects a Jira child issue before invoking
the Developer Agent.

## Stage 5 — Task implementation

1. A human selects one Jira child issue from the approved plan mapping.
2. Run the role defined in `../agents/developer.md` with the feature slug and Jira
   issue reference.
3. The Developer Agent verifies approvals and maps the issue to exactly one
   `TASK-*` item before editing.
4. The Developer Agent asks the human which branch (`development`,
   `staging`, `production`, or another named branch) is the source for this
   task, then creates a branch named
   `<jira-issue-key>-<short-kebab-case-title>` from that source before
   editing any files.
5. It modifies only the selected task's approved scope under the relevant
   `../../smooth-web/smooth-web-*/` repository and adds the required tests.
6. It runs task-specific checks followed by available type checking, linting,
   tests, and the repository's configured production build.
7. It returns the evidence described in
   `../templates/DEVELOPER_REPORT.template.md` with status `READY_FOR_REVIEW`,
   `BLOCKED`, or `PLAN_MISMATCH`.

The Developer Agent stops after one task. `READY_FOR_REVIEW` authorizes review,
not another Jira task, Jira transition, commit, push, deployment, or declaration
that the feature is done.

## POC completion

This POC intentionally has no automated Review Agent or separate completion
stage. After each selected Jira task, a human reviews the Developer Report and
the running application. The human may select the next approved Jira task and
run the development stage again. The POC is complete when the human accepts the
demonstrated feature.

Use the commands under `../../.claude/commands/` to run these stages from the
workspace root.
