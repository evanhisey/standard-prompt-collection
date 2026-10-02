# ACF Jira MCP Agent

This package provides a VS Code custom agent plus a bundled skill for safe Jira MCP work in the ACF environment.

The agent is the selectable Jira persona. The bundled `acf-jira-mcp-workflow` skill contains the detailed Jira MCP rules for read-only analysis, issue triage, comment drafting, approved writer-test operations, validation, and troubleshooting.

Default new-ticket drafting is Story-centered. Bug, Task, and Spike are not default issue types for this package. Epic drafts are supported when explicitly requested or clearly required as a planning parent.

When an Epic request does not provide or directly imply child story/task work, assume the user intended only the Epic. When the user provides both Epic and child story/task work, process both as a proposed multi-create plan and require explicit approval for each issue.

Teams that usually work in one Jira project or board can configure optional local defaults for project and board context. Defaults are shown before writes and can be overridden by the user; they do not replace explicit approval.

For best results, users should provide complete Story inputs up front:

```text
Create a Jira Story in <PROJECT> for <work to perform>.

Scope:
- <included work>

Acceptance Criteria:
- <observable completion condition>

Validation:
- <evidence, command, version number, review, or check that proves completion>

Optional:
- Epic/parent: <issue key or standalone>
- Environment/system: <target host, service, repo, or platform>
- Exclusions: <what not to change>
```

Minimal usable example:

```text
Create a Jira Story in ATO to update the application monitoring agent.
Validation: confirmed updated version number.
```

> **Local session requirement:** Use this MCP package from a VS Code Chat **Local** session / local agent harness. Do not run Jira MCP workflows from a Copilot-hosted session unless that session type has been separately validated for the required MCP server access and tool count. Copilot-hosted sessions can conflict with MCP tool availability and model endpoint tool-count limits.

## Package Contents

```text
acf-jira-mcp-agent/
├── agents/
│   └── acf-jira-mcp-agent.agent.md
├── skills/
│   └── acf-jira-mcp-workflow/
│       ├── SKILL.md
│       ├── diagnostic-improvement-prompt.md
│       ├── jira-issue-standard.md
│       ├── mcp-workflow.md
│       └── validation-checklist.md
├── examples/
│   ├── acf-jira-readonly.mcp.json
│   ├── acf-jira-writer-test.mcp.json
│   └── acf-jira-deletion-test.pending.mcp.json
├── README.md
├── INSTALL.md
├── CHANGELOG.md
└── eval-cases.md
```

## Use When

Use this package when a workspace needs Jira MCP support for:

- issue retrieval and summarization;
- JQL search and backlog review;
- issue triage and dependency analysis;
- drafting issue descriptions, acceptance criteria, comments, or field updates;
- approved Jira comment creation, field updates, links, or transitions through a writer-test profile;
- Jira MCP setup or troubleshooting.

## Operating Model

Default behavior is read-only. The agent may retrieve, search, analyze, summarize, and draft proposed Jira updates without writing to Jira.

Jira writes require all of the following:

1. A separate approved writer-test MCP profile.
2. Exact Jira server, project key, issue key, and operation identification.
3. Current issue state retrieval before writing.
4. Explicit human approval naming the issue key or project key and operation.
5. Post-write retrieval and validation.

Do not use this package for bulk updates, delete operations, project administration, workflow administration, permission changes, or broad JQL-driven writes.

Incorrect or unwanted story submissions should be handled through non-destructive correction, comment evidence, replacement linking, or a project-approved cancelled/closed transition. Physical deletion is outside the initial writer-test scope unless a separate delete-capability review explicitly approves it.

Physical story deletion, when approved, must use a separate reviewed deletion-capable MCP profile and exact issue-key approval. The normal read-only and writer-test profiles must not expose delete tools.

Deletion-path development for disposable story `ATO-1469` has completed through live deletion and post-delete absence verification. The deletion profile example remains intentionally separated from the normal install path and must be used only for reviewed deletion testing against an exact approved issue key.

## Install

Follow `INSTALL.md`. This package is more than a simple skill copy because it includes:

- a custom agent;
- a bundled skill;
- Jira MCP server profiles;
- Credal and VS Code MCP prerequisites;
- read-only validation;
- optional writer-test configuration.

At a high level, copy the package runtime payload into the approved customization location for the intended scope:

```text
agents/acf-jira-mcp-agent.agent.md -> <approved-agent-destination>/acf-jira-mcp-agent.agent.md
skills/acf-jira-mcp-workflow/ -> <approved-skill-destination>/acf-jira-mcp-workflow/
```

For a VS Code user/global agent install, the `.agent.md` file goes directly in the user `prompts` folder: `%APPDATA%\Code\User\prompts\` on Windows or `~/.config/Code/User/prompts/` on Linux. Do not create a nested `agents/` folder there, and do not copy bundled skills into the user `prompts` folder. Keep the bundled skill installed in the target repository or workspace.

Then configure the Jira MCP profile in the VS Code user/global MCP configuration as described in `INSTALL.md`.

## Diagnostic and Improvement Source Mode

Active development has completed through Stage 20. During diagnostics, regression checks, or package improvements, do not require the agent or skill to be installed after every correction. Use this package source as the active maintenance reference:

```text
agents/acf-jira-mcp-agent.agent.md
skills/acf-jira-mcp-workflow/SKILL.md
skills/acf-jira-mcp-workflow/*.md
skills/acf-jira-mcp-workflow/diagnostic-improvement-prompt.md
```

When testing a correction, reload or reread the source files from this package location and treat them as authoritative for the current test. Copy to a repo, workspace, or user/global customization destination only when validating installation behavior or preparing a packaged release.

## Diagnostic and Improvement Prompt

Use `skills/acf-jira-mcp-workflow/diagnostic-improvement-prompt.md` as the primary post-development prompt for setup failures, MCP tool discovery issues, regression checks, documentation alignment, and package improvements.

The prior active-development runbook and detailed live-test evidence file were removed during cleanup because their historical value was low after Stage 21 closeout. Future writer or deletion capability changes require a new plan, exact target, MCP profile review, and explicit human approval captured in current package docs or eval cases.

## First Test Prompt

```text
Use the ACF Jira MCP Agent.
Retrieve issue <APPROVED-TEST-ISSUE> in read-only mode.
Summarize the issue state, missing acceptance criteria, blockers, and a draft comment.
Do not modify Jira.
```

Replace `<APPROVED-TEST-ISSUE>` with a Jira issue that is approved for read-only validation.

## Review Status

This package is a source customization package. It does not grant Jira permissions and does not replace Jira project workflow policy. Validate Jira MCP tool names, Jira URL, token permissions, and writer-test target issues in the destination environment before using write operations.
