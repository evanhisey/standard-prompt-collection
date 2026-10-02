# Jira MCP Workflow

This skill is designed around `mcp-atlassian` Jira tools exposed through VS Code MCP. Tool names and availability depend on the installed `mcp-atlassian` version and the configured Jira permissions.

## MCP Profiles

Use separate profiles for read-only and write testing.

| Profile | Purpose | Required mode |
|---|---|---|
| `acf_jira` | Read-only issue retrieval, project lookup, and JQL search. | `READ_ONLY_MODE=true` |
| `acfJiraWriterTest` | Approved writer-test comments, issue creation, field updates, links, and transitions. | `READ_ONLY_MODE=false` |
| `acfJiraDeletionTest` | Exceptional physical-deletion testing only after delete-tool validation, exact issue-key review, and separate approval. | `READ_ONLY_MODE=false` |

Do not convert `acf_jira` into a writer profile. Add writer tools only to `acfJiraWriterTest` after review.
Do not add delete tools to `acf_jira` or `acfJiraWriterTest`. Do not enable `acfJiraDeletionTest` for routine work; it is only for reviewed deletion tests after the exact Jira delete tool has been validated against the installed MCP package and the user gives separate live-delete approval.

Optional local context defaults may be configured for users who usually work in one Jira project or board:

| Environment variable | Purpose |
|---|---|
| `ACF_JIRA_DEFAULT_PROJECT` | Default project key to propose when the user omits a project. |
| `ACF_JIRA_DEFAULT_BOARD` | Default board name or ID to use as context when board metadata is available. |
| `ACF_JIRA_DEFAULT_ISSUE_TYPE` | Optional project-specific issue type override. |
| `ACF_JIRA_DEFAULT_EPIC` | Optional default Epic parent when explicitly approved for the user's workflow. |

These values are hints, not authorization. The agent must show any default it intends to use before a write, and the user can override it. Do not use defaults that conflict with the user's request or retrieved Jira data.

## Tool Allowlist Policy

Do not expose every `mcp-atlassian` tool. Keep `ENABLED_TOOLS` narrow to reduce accidental tool exposure and avoid language-model endpoint tool-count limits.

Validate exact Jira tool names from the installed `mcp-atlassian` version before approving this package for production use. Example names in package documentation are expected patterns and must be confirmed locally.

Read-only tools should cover only:

- issue retrieval;
- issue search / JQL search;
- project metadata retrieval;
- user or field metadata retrieval only when required for issue analysis.

Writer-test tools may cover only approved non-destructive operations:

- issue creation in an approved test project;
- comment creation;
- single-issue field update;
- issue link creation;
- workflow transition for a named issue.

Never expose delete, bulk update, project administration, permission administration, workflow administration, or broad write tools in the initial writer-test profile.

Physical deletion is not part of `acfJiraWriterTest`. If deletion is ever approved, use a separate deletion-test profile, such as `acfJiraDeletionTest`, with only the minimum read tools and the exact Jira delete tool validated from the installed `mcp-atlassian` version. For `mcp-atlassian` 0.23.1, source inspection validates the server function `delete_issue(issue_key)` mounted under the Jira namespace, which corresponds to the external MCP tool name `jira_delete_issue`.

## Deletion-Path Development Status

Deletion-path development for `ATO-1469` completed through planning, documentation, tool discovery, dry-run gate validation, runtime profile validation, approved live deletion, and post-delete absence verification.

Completed disposable target for this development path:

- Issue: `ATO-1469`
- Type: `Story`
- Former status before deletion: `Cancelled`
- Purpose: cancellation workflow test story reused for deletion-path validation.
- Result: deleted with `jira_delete_issue` and verified absent by exact-key retrieval and JQL key search.

Before any future live deletion can be proposed, the development path must:

1. validate the exact Jira delete tool name and required parameters from the installed MCP package;
2. define an isolated `acfJiraDeletionTest` profile that excludes normal writer tools unless a reviewed test requires them;
3. run the physical deletion gate as a dry run against the exact approved disposable issue key;
4. record pre-delete evidence for issue key, URL, summary, type, status, assignee, reporter, comments, and changelog;
5. obtain separate explicit approval naming the exact issue key and the word `delete`.

Source validation for the current local package identified `jira_delete_issue` with a single required `issue_key` parameter. Runtime use still requires enabling a separate `acfJiraDeletionTest` profile and obtaining final explicit delete approval for each target. Do not reuse the deleted `ATO-1469` key for future live-delete tests.

## Read-Only Workflow

1. Confirm the active MCP profile is `acf_jira` or another approved read-only profile.
2. Confirm `READ_ONLY_MODE=true` and a narrow `ENABLED_TOOLS` allowlist are configured.
3. Retrieve only the issue, project, or search results needed for the request.
4. Avoid broad JQL queries unless the user explicitly requests broad read-only analysis.
5. Present facts, draft text, and recommendations without writing to Jira.
6. State any unavailable MCP tools or unresolved permissions.

## Writer-Test Workflow

Before a write:

1. Confirm the user explicitly requested a Jira write.
2. Confirm the active writer profile is `acfJiraWriterTest`.
3. Retrieve the target issue or project with read-only tools.
4. Confirm the target Jira URL, project key, issue key, current status, issue type, and relevant current fields.
5. For approved test-project story creation, include assignee as an intended field and default it to the story creator/reporter when Jira allows assignment on create.
6. For story creation, apply the package Story defaults; do not infer priority, sprint, component, label, fix version, due date, or parent Epic.
7. Show any configured default project, board, issue type, or Epic context that will be applied, and do not apply it if the user overrides it.
8. For Epic creation, default to Epic-only when no child story/task work is provided or directly implied; when Epic and child work are both provided, show an ordered multi-create plan and require explicit approval for each issue.
9. Confirm the write is one issue or one approved test-project issue creation.
10. Show the exact operation and target.
11. Stop for explicit approval naming the issue key or test project and the operation.

After a write:

1. Retrieve the affected issue.
2. Confirm the expected changed field, comment, link, created issue, or transition result.
3. Confirm unrelated fields were not changed when the MCP response exposes enough data.
4. For disposable writer-test story changes, add or verify a Jira comment summarizing the change and validation evidence.
5. Report the issue key, operation, result, and validation gaps.
6. Require browser review for visible comments, assignment changes, workflow transitions, and issue creation.

## Incorrect Story Disposition

Incorrect, duplicate, or unwanted story submissions must be handled as controlled single-story dispositions. The default disposition is non-destructive:

1. retrieve the exact issue and confirm it is the intended incorrect story;
2. prefer correcting the story, adding a visible disposition comment, linking to a replacement story, or transitioning to `Cancelled`/closed equivalent when Jira exposes that transition;
3. show the exact issue key, current status, proposed disposition, comment text, and Jira MCP tool;
4. stop for explicit approval naming the issue key and disposition;
5. execute only the approved operation;
6. retrieve the issue afterward and verify the resulting state.

Do not add delete tools to the initial writer-test profile. Physical deletion requires a separate reviewed profile, explicit delete-tool validation, audit/recovery review, and exact issue-key approval.

## Physical Story Deletion - Exceptional Workflow

Physical deletion is an exceptional cleanup workflow for incorrect story submissions that cannot be resolved cleanly through correction, comment evidence, replacement linking, or a cancelled/closed transition.

Before any physical deletion can be proposed:

1. confirm a separate deletion-capable MCP profile has been reviewed and enabled outside `acf_jira` and `acfJiraWriterTest`;
2. validate the exact Jira delete tool exposed by the installed MCP package;
3. retrieve the issue by exact key through read-only tools;
4. confirm the issue is a disposable or incorrect submission and not operational work;
5. check for child issues, linked issues, attachments, comments, and useful audit/history evidence when tools expose them;
6. capture the issue key, URL, summary, status, reporter, assignee, created timestamp, and reason for deletion in the test evidence before deleting;
7. show the exact delete tool and target issue key;
8. require explicit human approval using the exact issue key and the word `delete`.

After an approved deletion:

1. attempt to retrieve the deleted issue by exact key;
2. confirm Jira reports the issue is unavailable, deleted, or not found;
3. search for the deleted issue summary or key if the search tool can safely verify absence;
4. report the deletion result and any validation uncertainty.

Never delete by JQL result set, summary search, board contents, sprint contents, assignee, label, component, or any bulk selector. Never retry a delete after an ambiguous response until a read-only retrieval/search determines whether the first request succeeded.

## Stop Conditions

Stop and ask for confirmation or missing evidence when:

- the Jira base URL is unknown;
- the issue key or project key is ambiguous;
- the requested write affects more than one issue;
- the operation requires project administration, workflow administration, delete, or bulk update tools;
- required field metadata or allowed values are unavailable;
- the requested transition is not available for the current issue status;
- the writer profile is unavailable;
- the MCP server advertises broad Jira or Confluence tools outside the approved allowlist;
- the user asks to delete a Jira story but no separately reviewed delete-capable profile has been approved;
- the user asks to paste, print, or store credentials.

## Secret Handling

Jira credentials must be stored through VS Code MCP input secrets or another approved VS Code secret mechanism. Do not place Jira PATs or bearer tokens directly in `mcp.json`, documentation, prompts, screenshots, logs, or source control.

Use `UV_SYSTEM_CERTS=true` when the environment requires OS certificate-store trust for `uvx mcp-atlassian`. Do not disable TLS verification.
