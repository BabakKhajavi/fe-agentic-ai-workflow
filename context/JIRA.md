# Jira Context

## POC target

- **Atlassian site:** `smoothpay.atlassian.net`
- **Project name:** `Smooth Dev Master`
- **Project key:** `SDM`
- **Project type:** Software (classic)
- **Board name:** `Web`
- **Board ID:** `151`
- **Board URL:** `https://smoothpay.atlassian.net/jira/software/c/projects/SDM/boards/151`
- **Workstream field:** Component
- **Component name:** `Web`
- **Component ID:** `10044`

For this POC, all Jira searches, previews, and issue creation must be restricted
to project `SDM` and component `Web`.

The project has both a board named `Web` (board ID `151`) and a component named
`Web` (component ID `10044`). They serve different purposes:

- The board is the human-facing agile view.
- The component is an issue field that can be applied and queried through Jira.

Do not substitute the separate `Web` label for either one.

Jira issues are not directly assigned to a board. A board displays issues that
match its saved filter. Apply component `Web` to created issues, then verify that
they appear on board `151`. If they do not, inspect the board filter and required
sprint/backlog conditions rather than changing issue scope silently.

## Observed issue configuration

Web-component work has been observed using:

- `Task` — issue type ID `10002`
- `Bug` — issue type ID `10004`

The create screen requires project, issue type, reporter, and summary. Reporter
defaults to the authenticated user. Component, description, assignee, priority,
and labels are optional based on the inspected project configuration.

For planned feature work, default child issues to `Task`. Use `Bug` only when an
approved `TASK-*` explicitly describes correction of a defect.

The Jira Agent must inspect the current project metadata before creation because
Jira configuration can change. If this context conflicts with live Jira data,
stop and request human resolution.

## Safety rules

- Include component `Web` on every issue created for this POC.
- Search for duplicates within `project = SDM AND component = Web`.
- Never create work in another project or component unless this context is
  explicitly changed and human-approved.
- Show `SDM / Web` and component ID `10044` in the creation preview.
- Show board `Web` and board ID `151` as the expected destination view.
- After creation, verify whether the issues are visible on board `151` and report
  the result. Do not add an issue to a sprint unless a human explicitly asks.
- Do not assign users, estimates, priorities, sprints, labels, or due dates by
  default.
