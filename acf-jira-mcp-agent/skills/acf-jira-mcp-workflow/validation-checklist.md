# Jira MCP Validation Checklist

Use this checklist before presenting Jira analysis and before or after any approved Jira write.

## Read-Only Analysis

- [ ] The Jira base URL or MCP profile is known.
- [ ] The project key, issue key, or JQL query is explicit.
- [ ] The active profile is read-only unless the user explicitly requested a write.
- [ ] Retrieved facts are separated from recommendations.
- [ ] Missing fields, permissions, or metadata are called out.
- [ ] Draft comments or issue text are clearly marked as drafts.
- [ ] No Jira write tool was called.
- [ ] No credentials or secrets are included in the response.

## JQL Search

- [ ] The query is scoped to the user's requested project, issue set, or purpose.
- [ ] Result volume is limited when possible.
- [ ] The result list is not used as a write target list.
- [ ] Private details outside the requested scope are not summarized unnecessarily.

## Pre-Write Controls

- [ ] The user explicitly requested a Jira write.
- [ ] Post-development diagnostics or package improvements are following `skills/acf-jira-mcp-workflow/diagnostic-improvement-prompt.md`.
- [ ] New writer/deletion development has a new plan, exact approved Jira target, MCP profile review, explicit approval, and current eval/checklist coverage before any live Jira write.
- [ ] During diagnostics or improvement work, current source package files have been reread after the latest correction, or this test is explicitly validating installed package discovery.
- [ ] The active profile is the approved writer-test profile.
- [ ] The issue key or test project is exact and unambiguous.
- [ ] Current issue state was retrieved before writing.
- [ ] The operation is non-destructive and single-target.
- [ ] The exact field, comment, link, issue creation, or transition was shown to the user.
- [ ] For disposable writer-test story changes, the accompanying Jira comment text was shown to the user.
- [ ] New test stories default assignee to the story creator/reporter when Jira permits it, or the approved exception is documented.
- [ ] New story drafts use the package Story defaults and do not guess priority, sprint, component, labels, fix version, due date, or parent Epic.
- [ ] Configured default project, board, issue type, or Epic context is shown to the user before use and does not conflict with the request or retrieved Jira data.
- [ ] Epic drafts default to Epic-only only when no child story/task work is provided or directly implied; if Epic and child work are both provided, each issue creation is shown and approved.
- [ ] The user explicitly approved the issue key or project key and operation.
- [ ] The requested write does not require delete, bulk update, project administration, workflow administration, or permission administration.

## Post-Write Validation

- [ ] The affected issue was retrieved after the write.
- [ ] The expected comment, field value, link, created issue, or status is visible.
- [ ] The writer-test change comment is visible when the write changed a disposable story.
- [ ] Unrelated fields were not changed when sufficient data is available.
- [ ] The result includes issue key, operation, and validation summary.
- [ ] Browser review is required for visible comments, new issues, assignments, links, and transitions.

## Story Discovery and Board Progression

- [ ] Story discovery tests pass before modifying any issue.
- [ ] Ambiguous story identification stops safely and requests human selection.
- [ ] Wrong-project, wrong-board, or wrong-issue-type candidates are rejected as write targets.
- [ ] Board column names are not treated as workflow status names.
- [ ] Current issue status and currently permitted Jira transitions are retrieved before each transition.
- [ ] Only one approved transition is executed at a time.
- [ ] Each approved progression change includes a Jira comment recording the transition and test evidence.
- [ ] Changelog/history evidence is retrieved after each transition when available.

## Incorrect Story Disposition

- [ ] The story to correct, cancel, close, or link is identified by exact issue key.
- [ ] The current issue summary, type, status, assignee, reporter, and relevant links were retrieved before disposition.
- [ ] Non-destructive correction, comment, link, or workflow transition was considered before deletion.
- [ ] The proposed disposition and comment text were shown to the user.
- [ ] The user explicitly approved the exact issue key and disposition.
- [ ] No physical deletion was attempted unless a separate reviewed delete-capable profile and exact approval exist.
- [ ] The issue was retrieved afterward and the resulting status, comment, link, or field update was verified.

## Physical Story Deletion

- [ ] Deletion-path development approval is limited to planning, documentation, tool discovery, profile design, and dry-run validation until a separate live-delete approval is provided.
- [ ] For the current deletion-path development run, the only accepted target is cancelled disposable Story `ATO-1469` unless a new exact key is separately approved.
- [ ] A separate deletion-capable MCP profile is reviewed and enabled outside `acf_jira` and `acfJiraWriterTest`.
- [ ] The exact Jira delete tool name has been validated against the installed MCP package.
- [ ] For locally inspected `mcp-atlassian` 0.23.1, source validation identified `jira_delete_issue` with required parameter `issue_key`.
- [ ] Pending profile examples are not enabled until separate live-delete approval is granted.
- [ ] The delete target is one exact issue key, not a search result, board, sprint, label, component, assignee, or bulk selector.
- [ ] The issue was retrieved by exact key before deletion.
- [ ] The issue is confirmed to be disposable, incorrect, duplicate, or otherwise approved for deletion.
- [ ] Non-destructive alternatives were presented first.
- [ ] Available child issue, link, attachment, comment, and audit/history evidence was reviewed or captured before deletion.
- [ ] The user explicitly approved deletion using the exact issue key and the word `delete`.
- [ ] The issue was retrieved or searched afterward to verify it is deleted, unavailable, or not found.
- [ ] Ambiguous delete responses were not retried until read-only verification determined whether the first delete occurred.

## MCP Configuration

- [ ] `uvx mcp-atlassian --help` works before MCP use.
- [ ] Jira PAT or token is stored through an approved secret input.
- [ ] `READ_ONLY_MODE=true` is present in the read-only profile.
- [ ] `ENABLED_TOOLS` is present and narrow.
- [ ] `UV_SYSTEM_CERTS=true` is present when required by the environment.
- [ ] Writer tools are isolated to a separate writer-test profile.
- [ ] Exact Jira tool names were validated against the installed `mcp-atlassian` version.
