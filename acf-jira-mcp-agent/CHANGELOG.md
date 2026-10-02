# Changelog

## 0.1.0

Initial package draft.

- Added `ACF Jira MCP Agent` custom agent.
- Added bundled `acf-jira-mcp-workflow` skill.
- Added Jira MCP workflow, issue standard, validation checklist, and diagnostic prompt.
- Added end-user installation guide.
- Added read-only and writer-test MCP example configuration files.
- Kept default behavior read-only with explicit approval gates for writer-test operations.
- Added and later removed the temporary controlled writer capability development prompt after active development closeout.
- Updated writer-test story creation guidance so new stories default to the story creator/reporter as assignee when Jira allows it, improving board visibility.
- Updated writer-test guidance so each approved disposable story change includes a Jira comment documenting the change and validation evidence.
- Added and later removed the temporary detailed live-test evidence file after closeout, retaining durable behavior in docs, eval cases, and checklists.
- Added Section 14 promotion criteria review documenting writer-test approval status and accepted caveats.
- Added Section 15 execution behavior review documenting approval gates, scoped writes, verification behavior, and accepted caveats.
- Added Story-centered default drafting guidance, Epic draft gating, and tightly controlled incorrect-story disposition guidance that prefers non-destructive correction over deletion.
- Added complete Story input coaching so users can quickly learn how to provide project, scope, acceptance criteria, validation, environment, and exclusions up front.
- Replaced product-specific Story coaching examples with generic application monitoring agent wording.
- Added optional configured project/board defaults and refined Epic gating so Epic-only creation is assumed only when no child story/task work is provided or directly implied.
- Added an exceptional physical story deletion workflow requiring a separate reviewed deletion-capable profile, exact issue-key approval, pre-delete evidence capture, and post-delete verification.
- Added accepted deletion-path development plan artifacts for cancelled disposable story `ATO-1469`, including a separated deletion-test profile example and dry-run eval.
- Validated the local `mcp-atlassian` 0.23.1 Jira delete tool source as `jira_delete_issue` backed by `delete_issue(issue_key)`.
- Completed the approved live deletion test for disposable story `ATO-1469`, verified the key absent afterward, and confirmed delete-tool isolation from normal read-only and writer-test profiles.
- Transitioned the package from active development mode to diagnostic/improvement mode, making `diagnostic-improvement-prompt.md` the primary maintenance prompt.
- Removed low-value historical active-development files after closeout cleanup: the temporary writer development prompt and detailed Stage 12-21 evidence log.
- Added required VS Code Local harness settings and explicit **ACF Jira MCP Agent** selection guidance for Jira MCP validation and best runtime behavior.
