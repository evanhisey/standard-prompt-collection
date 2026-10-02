# Jira MCP Diagnostic and Improvement Prompt

Use this prompt when Jira MCP setup, tool discovery, authentication, authorization, skill discovery, read-only retrieval, writer-test behavior, deletion-test isolation, documentation alignment, or regression behavior needs diagnosis or improvement.

This is the primary post-development maintenance prompt for the package. The previous active-development runbook and detailed live-test evidence file were removed during closeout cleanup; future live writer or deletion development requires a new plan, exact target, MCP profile review, explicit approval, and current eval/checklist coverage.

## Goals

1. Diagnose the failing layer without exposing secrets.
2. Keep troubleshooting read-only unless writer-test behavior is the explicit subject.
3. Produce a concrete fix or a bounded escalation path.
4. Recommend documentation or eval-case improvements when the failure reveals a reusable gap.
5. Preserve the completed reader / writer-test / deletion-test separation established during active development.

## Expected Architecture

```text
VS Code
  |
  +-- Copilot Chat / agent mode
      |
      +-- ACF Jira MCP Agent
          |
          +-- acf-jira-mcp-workflow skill
              |
              +-- VS Code user/global MCP configuration
                  |
                  +-- acf_jira or acfJiraWriterTest
                      |
                      +-- uvx mcp-atlassian
                          |
                          +-- ACF Jira
```

Expected read-only profile properties:

- command: `uvx`
- args: `["mcp-atlassian"]`
- Jira URL configured for the approved ACF Jira instance
- Jira token stored through VS Code secret input
- `READ_ONLY_MODE=true`
- narrow `ENABLED_TOOLS`
- `UV_SYSTEM_CERTS=true` when required

## Safety Rules

- Do not ask the user to paste a PAT, bearer token, password, cookie, private key, or secret value.
- Do not print or decode token values.
- Do not disable TLS verification.
- Do not add broad toolsets merely to make one Jira tool appear.
- Do not convert the read-only profile into a writer profile.
- Do not test writes against production issues without explicit approval and an approved writer-test profile.

## Diagnostic Order

### 0. Post-Development Scope Check

Before changing package behavior, classify the request:

- `diagnostic`: identify why setup, retrieval, tool exposure, authentication, authorization, or validation failed;
- `improvement`: update docs, evals, checklists, examples, or prompts without changing Jira state;
- `regression`: rerun a read-only or dry-run check against the completed development evidence;
- `new writer/deletion development`: requires a new plan, exact approved Jira target, and explicit approval before any live write.

Do not create a new disposable story, transition an issue, or delete an issue as part of routine diagnostics. Live writer or deletion work must be treated as new development and must reuse the approval gates from the archived development runbook.

### 1. Skill and Agent Discovery

Check that the package was copied into the target customization location and contains:

```text
<approved-agent-destination>/acf-jira-mcp-agent.agent.md
<approved-skill-destination>/acf-jira-mcp-workflow/SKILL.md
```

If the agent or skill is not discovered, verify the target VS Code build supports the selected customization path and reload the workspace.

### 2. `uv` / `uvx` Runtime

Verify outside VS Code:

```powershell
uv --version
uvx --version
where.exe uvx
uvx mcp-atlassian --help
```

On Linux/macOS:

```bash
uv --version
uvx --version
command -v uvx
uvx mcp-atlassian --help
```

If certificate verification fails, test OS certificate use with the approved `UV_SYSTEM_CERTS=true` configuration. Do not bypass certificate validation.

### 3. MCP Configuration

Verify the global VS Code MCP configuration contains exactly one intended Jira read-only server definition and no duplicate workspace definitions for the same profile.

Check for:

- correct server name;
- `command: "uvx"`;
- `args: ["mcp-atlassian"]`;
- Jira URL env var for the approved Jira instance;
- secret input reference for the Jira token;
- `READ_ONLY_MODE=true` for the read-only profile;
- narrow `ENABLED_TOOLS`;
- `UV_SYSTEM_CERTS=true` when required.

### 4. Tool Discovery

Use VS Code MCP server output or tool listing to verify exact Jira tool names. If an expected tool is missing:

- confirm the installed `mcp-atlassian` version;
- confirm `ENABLED_TOOLS` uses exact public tool names;
- remove unavailable or renamed tools;
- avoid enabling all Jira and Confluence tools.

### 5. Authentication and Authorization

If tool calls fail:

- confirm the token is for Jira, not Confluence-only access;
- confirm the token is stored through a secret input;
- confirm the user can access the issue or project in the browser;
- confirm the token has read permission for read-only tests;
- confirm writer-test operations use an approved test project or issue with write permission.

### 6. Read-Only Smoke Test

Use a known approved test issue or project. Confirm retrieval works before attempting search or writer-test behavior.

If no approved test issue exists, stop and request one. Do not discover issues by running broad project-wide searches unless the user explicitly approves read-only search scope.

### 7. Writer-Test Smoke Test

Only when approved:

1. Confirm the writer-test profile is active.
2. Retrieve the target issue.
3. Draft the exact comment or field update.
4. Stop for explicit approval.
5. Perform the single approved write.
6. Retrieve the issue again and validate the result.

### 8. Package Improvement Output

When the diagnosis identifies a package gap, propose the smallest improvement to one or more of:

- `README.md`;
- `INSTALL.md`;
- `skills/acf-jira-mcp-workflow/SKILL.md`;
- `skills/acf-jira-mcp-workflow/mcp-workflow.md`;
- `skills/acf-jira-mcp-workflow/jira-issue-standard.md`;
- `skills/acf-jira-mcp-workflow/validation-checklist.md`;
- `eval-cases.md`;
- MCP example JSON files.

Record whether the improvement is documentation-only, eval-only, MCP-profile-only, or behavior-changing. Behavior-changing improvements require separate review before live Jira writes.

## Output Format

Return:

1. Failing layer.
2. Evidence observed.
3. Likely cause.
4. Fix or next check.
5. Secret-safety confirmation.
6. Documentation update recommendation.
7. Eval-case recommendation when the issue could recur.
8. Whether the archived development runbook needs a regression-reference update.
