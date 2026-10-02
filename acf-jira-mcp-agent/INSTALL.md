# End-User Installation Guide
## VS Code + Credal + ACF Jira MCP Agent

**Document version:** 0.1.0  
**Audience:** Users configuring VS Code Chat with the ACF Jira MCP Agent and `mcp-atlassian` Jira access.  
**Validated operating mode:** VS Code Chat, Local agent harness, custom agent package, Credal custom model, global VS Code MCP configuration, read-only Jira access first, optional controlled writer-test profile.

This guide follows the same installation model as the Confluence MCP runbook standardizer package, adapted for Jira. It intentionally documents only the supported VS Code Chat path:

```text
VS Code
  |
  +-- Local agent harness
  |     |
  |     +-- Credal custom OpenAI-compatible model
  |
  +-- VS Code user/global MCP configuration
        |
        +-- acf_jira
        |     |
        |     +-- uvx mcp-atlassian
        |     +-- read-only Jira retrieval and search
        |
        +-- acfJiraWriterTest
              |
              +-- uvx mcp-atlassian
              +-- controlled Jira writer-test operations
              +-- Jira PAT stored through VS Code secret input
```

The MCP server is started by VS Code when needed. It is not a separate long-running service to install permanently.

> Security requirement: Do not paste a Credal token, Jira Personal Access Token (PAT), bearer token, password, cookie, or session secret into this guide, a Git repository, `mcp.json` as a literal value, a prompt, a ticket, screenshots, logs, or source code.

> Validation requirement: Confirm exact Jira MCP tool names from the installed `mcp-atlassian` version before approving this package for production use. Example `ENABLED_TOOLS` values in this package are expected names and may need adjustment.

> Local session requirement: Use this MCP package from a VS Code Chat **Local** session / local agent harness. Do not use a Copilot-hosted session for Jira MCP validation or writer development unless that session type has been separately validated for the required MCP server access and tool count. Copilot-hosted sessions can conflict with MCP tool availability and model endpoint tool-count limits.

## 1. What You Are Installing

This deployment has four parts:

1. VS Code Chat with custom agent support using the **Local** session target / local agent harness.
2. Credal configured as the custom OpenAI-compatible model provider.
3. `mcp-atlassian` launched by VS Code as a Jira MCP server.
4. The ACF Jira MCP Agent package copied into the approved customization path for the chosen install scope.

The package provides:

```text
agents/acf-jira-mcp-agent.agent.md
skills/acf-jira-mcp-workflow/SKILL.md
skills/acf-jira-mcp-workflow/*.md
```

The validated runtime selection is:

- Session target / harness: **Local**
- Agent role: **Agent**
- Model: **Credal custom model**
- MCP server for read-only work: **acf_jira**
- MCP server for approved writer tests: **acfJiraWriterTest**

Do not claim another harness is supported for authenticated Jira MCP unless it is separately validated in the target environment.

## 2. Before You Start

Confirm the following before continuing:

- [ ] Microsoft VS Code is installed.
- [ ] VS Code Chat can run a Local session / local agent harness for MCP-backed workflows.
- [ ] VS Code Chat can discover custom agents and skills in the selected target location.
- [ ] Credal is configured as the approved OpenAI-compatible model provider.
- [ ] You can reach `https://jira.acf.gov/` from the workstation using the required ACF network/VPN connection.
- [ ] You can sign in to Jira normally.
- [ ] You are allowed to create or use a Jira PAT or approved token for MCP access.
- [ ] You are allowed to install or run local developer tooling such as `uv` / `uvx`.
- [ ] You understand that the Credal token and the Jira PAT are different credentials and must not be reused interchangeably.
- [ ] An approved read-only test issue or test project is available for validation.
- [ ] A separate approved writer-test issue or project is available before testing writes.

If the Jira UI does not provide Personal Access Tokens, stop and contact the appropriate support team. Do not substitute Basic authentication, username/password, or another token type unless explicitly approved for this environment.

## 3. Configure Credal

Use the same Credal setup procedure approved for the standard prompt collection MCP packages. Store the Credal token through the VS Code language-model secret mechanism.

Do not store the Credal token in `mcp.json`. The Credal token and Jira PAT are different credentials.

## 4. Install And Verify `uv` / `uvx`

Use the organization's approved software distribution method when available.

### Windows PowerShell

```powershell
winget install --id=astral-sh.uv -e
```

Open a new PowerShell window, then verify:

```powershell
uv --version
uvx --version
where.exe uvx
uvx mcp-atlassian --help
```

### Linux/macOS Bash

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

Open a new terminal, then verify:

```bash
uv --version
uvx --version
command -v uvx
uvx mcp-atlassian --help
```

If certificate validation fails with an unknown issuer error, use the documented `UV_SYSTEM_CERTS=true` MCP environment variable. Do not disable TLS verification.

## 5. Install The Agent Package

For post-development diagnostics or package improvements, use the source files in this package directly so corrections can be tested without repeatedly copying the agent or skill:

```text
<package-source>/agents/acf-jira-mcp-agent.agent.md
<package-source>/skills/acf-jira-mcp-workflow/SKILL.md
<package-source>/skills/acf-jira-mcp-workflow/*.md
<package-source>/skills/acf-jira-mcp-workflow/diagnostic-improvement-prompt.md
```

In diagnostic/improvement source mode, the test agent must reread these files after corrections and treat the current source content as authoritative. Use the copy/install steps below for install validation, packaged release validation, or end-user deployment. Future live writer or deletion development requires a new plan, exact target, MCP profile review, and explicit approval captured in current package docs or eval cases.

Copy the package runtime payload into the approved customization location for the intended scope. The source package uses neutral `agents/` and `skills/` directories. The source path is wherever this package was cloned, downloaded, or unpacked on the local machine. The destination can be repo-specific, workspace-specific, or user/global depending on the target VS Code/Copilot build and local policy.

Common destination patterns include:

```text
Repo-specific GitHub customization:
  <target-repo>/.github/agents/
  <target-repo>/.github/skills/

Repo or workspace agent customization:
  <target-repo-or-workspace>/.agents/agents/
  <target-repo-or-workspace>/.agents/skills/

User/global customization:
  <user-customizations>/agents/
  <user-customizations>/skills/
```

Set `$sourceRoot` / `source_root` to the actual local path of this package. Set the destination variables to the approved paths for the install scope you are using before copying.

### Windows PowerShell

```powershell
$sourceRoot = "C:\Path\To\standard-prompt-collection\acf-jira-mcp-agent"

# Choose the approved destination pair for one install scope.
# Repo-specific GitHub customization example:
# $targetAgents = "C:\Path\To\TargetRepo\.github\agents"
# $targetSkills = "C:\Path\To\TargetRepo\.github\skills"

# Repo or workspace agent customization example:
# $targetAgents = "C:\Path\To\TargetRepoOrWorkspace\.agents\agents"
# $targetSkills = "C:\Path\To\TargetRepoOrWorkspace\.agents\skills"

# User/global customization example:
$targetAgents = "C:\Path\To\UserCustomizations\agents"
$targetSkills = "C:\Path\To\UserCustomizations\skills"

New-Item -ItemType Directory -Force -Path $targetAgents | Out-Null
New-Item -ItemType Directory -Force -Path $targetSkills | Out-Null

Copy-Item -Force `
  -Path (Join-Path $sourceRoot "agents\acf-jira-mcp-agent.agent.md") `
  -Destination (Join-Path $targetAgents "acf-jira-mcp-agent.agent.md")

Copy-Item -Recurse -Force `
  -Path (Join-Path $sourceRoot "skills\acf-jira-mcp-workflow") `
  -Destination (Join-Path $targetSkills "acf-jira-mcp-workflow")
```

### Linux/macOS Bash

```bash
source_root="/path/to/standard-prompt-collection/acf-jira-mcp-agent"

# Choose the approved destination pair for one install scope.
# Repo-specific GitHub customization example:
# target_agents="/path/to/target-repo/.github/agents"
# target_skills="/path/to/target-repo/.github/skills"

# Repo or workspace agent customization example:
# target_agents="/path/to/target-repo-or-workspace/.agents/agents"
# target_skills="/path/to/target-repo-or-workspace/.agents/skills"

# User/global customization example:
target_agents="/path/to/user-customizations/agents"
target_skills="/path/to/user-customizations/skills"

mkdir -p "$target_agents" "$target_skills"
cp "$source_root/agents/acf-jira-mcp-agent.agent.md" "$target_agents/acf-jira-mcp-agent.agent.md"
cp -R "$source_root/skills/acf-jira-mcp-workflow" "$target_skills/acf-jira-mcp-workflow"
```

Reload VS Code after copying the package.

## 6. Configure The Global VS Code Jira MCP Server

Use the VS Code user/global MCP configuration as the single source of truth for `acf_jira` and `acfJiraWriterTest`.

Validated Windows location:

```text
%APPDATA%\Code\User\mcp.json
```

Example resolved path:

```text
C:\Users\<user>\AppData\Roaming\Code\User\mcp.json
```

Open the global MCP configuration using the VS Code command for your build. In validated builds this is typically available through the Command Palette as:

```text
MCP: Open User Configuration
```

If that exact command is unavailable, use the MCP configuration command exposed by the target VS Code build, or open the global user configuration file directly with the VS Code `code` CLI:

```powershell
code "$env:APPDATA\Code\User\mcp.json"
```

If the `code` command is not on `PATH`, run **Shell Command: Install 'code' command in PATH** from the VS Code Command Palette, or edit the global path above directly.

Do not create a duplicate `acf_jira` or `acfJiraWriterTest` server definition in:

- `.vscode/mcp.json`
- `.mcp.json`
- `~/.copilot/mcp-config.json`
- any other local or workspace-specific MCP file

A workspace `.vscode/mcp.json` may exist for other servers, but it should not contain duplicate Jira server definitions when these servers are managed globally.

## 6.1 Confirm The Tool Allowlist For The Installed Version

Do not expose all `mcp-atlassian` tools. Keep the Jira MCP tool allowlist narrow. Do not broaden tool exposure to compensate for a Copilot-hosted session limitation. Use the Local session target and validate the exact Jira tool names exposed by the installed MCP version.

Before publishing or approving this guide for a managed environment, verify the exact Jira tool names exposed by the installed `mcp-atlassian` version. Use the MCP startup output and VS Code's server listing/output views as evidence. Remove unavailable or renamed tools, and do not include Confluence tools unless the deployment explicitly requires Confluence in the same profile.

The following Jira read-only tool names are expected by this package and must be revalidated after `mcp-atlassian` upgrades:

```text
jira_search,
jira_get_issue,
jira_get_project
```

The following writer-test tool names are expected by this package and must be revalidated before writer testing:

```text
jira_create_issue,
jira_add_comment,
jira_update_issue,
jira_create_issue_link,
jira_transition_issue
```

Keep writer tools isolated to the `acfJiraWriterTest` profile. Do not add writer tools to `acf_jira`.

## 6.2 Global `mcp.json` Pattern

Add a read-only Jira profile modeled on `examples/acf-jira-readonly.mcp.json`.

Required properties:

- `command`: `uvx`
- `args`: `["mcp-atlassian"]`
- Jira URL for the approved ACF Jira instance: `https://jira.acf.gov/`
- Jira token stored through VS Code input secret reference
- `READ_ONLY_MODE=true`
- narrow `ENABLED_TOOLS`
- `UV_SYSTEM_CERTS=true` when required by the environment
- optional `ACF_JIRA_DEFAULT_PROJECT` and `ACF_JIRA_DEFAULT_BOARD` values when a user normally works in one project or board

Example skeleton:

```json
{
  "inputs": [
    {
      "type": "promptString",
      "id": "acf-jira-personal-token",
      "description": "ACF Jira Personal Access Token",
      "password": true
    }
  ],
  "servers": {
    "acf_jira": {
      "type": "stdio",
      "command": "uvx",
      "args": ["mcp-atlassian"],
      "env": {
        "JIRA_URL": "https://jira.acf.gov/",
        "JIRA_PERSONAL_TOKEN": "${input:acf-jira-personal-token}",
        "READ_ONLY_MODE": "true",
        "ENABLED_TOOLS": "jira_search,jira_get_issue,jira_get_project",
        "ACF_JIRA_DEFAULT_PROJECT": "",
        "ACF_JIRA_DEFAULT_BOARD": "",
        "UV_SYSTEM_CERTS": "true"
      }
    }
  }
}
```

Use `https://jira.acf.gov/` as the approved Jira base URL. Adjust the token variable name and `ENABLED_TOOLS` only after validating the installed `mcp-atlassian` version and the approved Jira authentication model.

Important notes:

- `JIRA_PERSONAL_TOKEN` uses a VS Code password input reference. The PAT is not stored as plaintext in `mcp.json`.
- `READ_ONLY_MODE=true` is required for the read-only profile.
- `ENABLED_TOOLS` is required to keep advertised MCP tools constrained and below model endpoint tool-count limits.
- `ACF_JIRA_DEFAULT_PROJECT` and `ACF_JIRA_DEFAULT_BOARD` are optional convenience hints. Leave them empty unless a local user or team has an approved default context. The agent must show these defaults before using them for a write.
- `UV_SYSTEM_CERTS=true` tells `uv` / `uvx` to use the operating-system certificate store. It is useful when `uvx` otherwise fails with `UnknownIssuer`. It does not disable TLS verification.
- Do not use `--insecure`, `verify=false`, `JIRA_SSL_VERIFY=false`, or any other TLS-bypass setting.

If `UV_SYSTEM_CERTS` is not needed in a future environment, it may be removed after `uvx mcp-atlassian --help` and MCP startup both succeed without certificate errors.

## 7. Enable The Global Server In VS Code

Defining `acf_jira` globally and enabling it for a workspace/session are separate steps.

Use VS Code's MCP UI or Command Palette to manage the global server. Validated builds expose server management through:

```text
MCP: List Servers
```

Select `acf_jira`, then choose actions such as:

```text
Start Server
Stop Server
Restart Server
Show Output
Show Configuration
```

Exact labels can vary by VS Code build. If a direct command such as `MCP: Start Server` or `MCP: Restart Server` exists, use it and select `acf_jira` when prompted.

When VS Code prompts for **ACF Jira Personal Access Token**, paste the PAT once. The input is hidden because `password` is `true`, and VS Code stores it securely.

Approve the MCP server trust prompt only after confirming:

- command: `uvx`
- argument: `mcp-atlassian`
- server name: `acf_jira`
- read-only mode is configured
- the tool allowlist is configured

## 8. Read-Only Functional Validation

Use an approved test issue.

Use VS Code Chat with:

- Session target / harness: **Local**
- Agent role: **Agent**
- Model: **Credal custom model**
- MCP server: **acf_jira**

Prompt:

```text
Use the ACF Jira MCP Agent.
Retrieve issue <APPROVED-TEST-ISSUE> in read-only mode.
Summarize the issue state and draft a comment.
Do not modify Jira.
```

Expected result:

- The issue is retrieved.
- The response separates retrieved facts from draft recommendations.
- No Jira write tool is called.
- No token or secret value appears in the response.

If this succeeds, the validated read-only MCP path is working. If it fails, use troubleshooting rather than weakening security settings.

## 9. Optional Writer-Test MCP Installation

Install this profile only when writer testing has been explicitly approved. The default end-user profile remains `acf_jira` in read-only mode.

The writer-test profile is intentionally separate so users can keep the known-good reader profile available while testing limited write behavior. Do not change `acf_jira` from read-only to writable.

Use this profile only for approved test issues, approved disposable test stories, or approved test projects. It is not a general production editing profile.

## 9.1 Writer-Test Entry Requirements

Before installing or enabling the writer-test profile, confirm:

- [ ] The read-only `acf_jira` validation in this guide passes.
- [ ] The approved Jira test project, board, and/or test issue are known.
- [ ] Your Jira account is authorized to create, edit, comment, link, and transition issues only in the approved test scope.
- [ ] A human reviewer is identified for the writer test.
- [ ] You understand that every write requires explicit approval in chat before the agent calls a write tool unless a bounded disposable-story sequence is explicitly approved.
- [ ] You will not use the writer-test profile against an existing operational issue during development.

If any item is false, stop and use only the read-only profile.

## 9.2 Add The Separate Writer-Test Server

Open the same global VS Code MCP configuration used for the read-only server:

```text
%APPDATA%\Code\User\mcp.json
```

Merge the writer-test server into the existing JSON. Do not paste the example as a second JSON document at the bottom of the file.

JSON rules to watch:

- The file must start with one `{` and end with one matching `}`.
- There is one top-level `"inputs"` array and one top-level `"servers"` object.
- Reuse the existing Jira PAT input unless your team intentionally approves separate reader and writer tokens.
- Add the writer server as another property inside `"servers"`.
- Put a comma between sibling items, but not after the last item in an array or object.

If your file already contains the read-only `acf_jira` server from this guide, the combined structure should look like this:

```json
{
  "inputs": [
    {
      "type": "promptString",
      "id": "acf-jira-personal-token",
      "description": "ACF Jira Personal Access Token",
      "password": true
    }
  ],
  "servers": {
    "acf_jira": {
      "type": "stdio",
      "command": "uvx",
      "args": ["mcp-atlassian"],
      "env": {
        "JIRA_URL": "https://jira.acf.gov/",
        "JIRA_PERSONAL_TOKEN": "${input:acf-jira-personal-token}",
        "READ_ONLY_MODE": "true",
        "ENABLED_TOOLS": "jira_search,jira_get_issue,jira_get_project",
        "UV_SYSTEM_CERTS": "true"
      }
    },
    "acfJiraWriterTest": {
      "type": "stdio",
      "command": "uvx",
      "args": ["mcp-atlassian"],
      "env": {
        "JIRA_URL": "https://jira.acf.gov/",
        "JIRA_PERSONAL_TOKEN": "${input:acf-jira-personal-token}",
        "READ_ONLY_MODE": "false",
        "ENABLED_TOOLS": "jira_search,jira_get_issue,jira_get_project,jira_create_issue,jira_add_comment,jira_update_issue,jira_create_issue_link,jira_transition_issue",
        "UV_SYSTEM_CERTS": "true"
      }
    }
  }
}
```

The comma after the `acf_jira` server is required because another server follows it. The writer-test server is last in this example, so it does not have a trailing comma.

The same input id, `acf-jira-personal-token`, is used by both profiles. That means VS Code prompts for one Jira PAT and passes it to either MCP server when that server starts. This is the simplest end-user setup when the same Jira account is approved for both read-only validation and writer testing.

Optional separate-token setup: if your team wants a different PAT for writer testing, add a second item inside `"inputs"` with a unique id such as `acf-jira-writer-test-personal-token`, then change only the writer-test server's `JIRA_PERSONAL_TOKEN` value to `${input:acf-jira-writer-test-personal-token}`. Do not create two `inputs` entries with the same `id`.

If your `mcp.json` contains other MCP servers, keep them and add commas using the same pattern.

Common JSON error to avoid:

```text
}
{
  "servers": {
    "acfJiraWriterTest": {}
  }
}
```

That is invalid because it creates two separate top-level JSON objects. Put `acfJiraWriterTest` inside the existing `"servers"` object instead.

The same writer-test server example is provided in this package at:

```text
examples/acf-jira-writer-test.mcp.json
```

Important requirements:

- Keep the server name exactly `acfJiraWriterTest`.
- Keep `READ_ONLY_MODE=false` only on this writer-test profile.
- Keep `acf_jira` unchanged with `READ_ONLY_MODE=true`.
- Keep `JIRA_PERSONAL_TOKEN` pointed at the existing Jira PAT input unless a separate writer-test token has been deliberately approved.
- Keep the writer-test `ENABLED_TOOLS` allowlist narrow.
- Do not add delete, bulk update, project administration, workflow administration, permission, user administration, or Confluence tools.
- Do not paste the PAT directly into `mcp.json`; use the VS Code password input reference.

## 9.3 Enable And Verify The Writer-Test Server

Use VS Code's MCP UI or Command Palette:

```text
MCP: List Servers
```

Select `acfJiraWriterTest`, then start or restart the server. When VS Code prompts for **ACF Jira Personal Access Token**, paste the PAT once.

Approve the MCP server trust prompt only after confirming:

- command: `uvx`
- argument: `mcp-atlassian`
- server name: `acfJiraWriterTest`
- read-only mode is disabled only for this writer-test profile
- the exposed tool list is limited to the approved writer-test allowlist

Useful non-secret startup evidence includes:

```text
Jira configuration loaded and authentication is configured
Read-only mode: DISABLED
Discovered N tools
```

The discovered tool count should be small. If the output shows broad Confluence tools, delete/bulk/admin tools, or a large unconstrained tool list, stop and correct `ENABLED_TOOLS` before using the profile.

## 9.4 Writer-Test Validation Prompt

Use this prompt after the writer-test server starts. It validates that the writer profile is available without authorizing a write.

```text
Use the ACF Jira MCP Agent with the acfJiraWriterTest profile.

Do not write yet.
Confirm whether the active MCP session exposes the approved writer-test profile named acfJiraWriterTest and whether Jira create, comment, update, link, and transition tools are available.

Retrieve the approved read-only test issue or approved test project metadata.
Show the proposed test scope, the exact issue or project target, and the exact MCP write tool you would use for the first test.
Then stop and wait for my explicit approval before any write.
```

For post-development troubleshooting or package improvement, use `skills/acf-jira-mcp-workflow/diagnostic-improvement-prompt.md`. New writer capability development must be treated as a new approved development plan with current package documentation and eval coverage.

Do not proceed to issue creation, comment creation, field updates, links, or transitions until the profile-verification stage passes and the user gives separate explicit approval.

## 9.5 Writer-Test Stop Conditions

Stop without writing if any of these occur:

- The active server is still the read-only `acf_jira` profile.
- The required writer tool is not available.
- The approved test project, board, issue type, or target issue is unclear.
- The requested operation targets an existing operational issue during development.
- The user has not explicitly approved the write.
- The requested transition is not one of Jira's currently permitted transitions for the issue.
- The requested operation would require delete, bulk update, project administration, workflow administration, permission administration, or user administration.

Stopping is the correct outcome when the writer-test profile is not available or the requested write is outside the approved tool set.

## 9.6 Writer-Test Profile Summary

The writer-test profile must:

- use `READ_ONLY_MODE=false` only in the writer-test profile;
- expose a narrow set of non-destructive writer tools;
- be limited by Jira permissions to an approved test project or test issue where possible;
- exclude delete, bulk update, project administration, workflow administration, and permission tools.

See `examples/acf-jira-writer-test.mcp.json`.

## 10. Writer-Test Smoke Test

Only after approval, use a known test issue and a harmless comment.

For post-development troubleshooting or package improvement, use `skills/acf-jira-mcp-workflow/diagnostic-improvement-prompt.md` instead of restarting the full writer test path. The archived development prompt remains available only for future, separately approved writer capability changes.

Prompt:

```text
Use the ACF Jira MCP Agent with the acfJiraWriterTest profile.
Retrieve issue <APPROVED-TEST-ISSUE>.
Draft the exact comment: "MCP writer-test validation comment."
Stop for approval before posting.
```

After approving the exact operation, the agent must retrieve the issue again and confirm the comment is present.

## 10A. Deletion-Test Profile - Exceptional Use Only

Do not install or enable a deletion-capable profile during normal read-only or writer-test setup. Deletion-path development is a separate phase for exceptional physical-deletion testing only.

The package includes a separated deletion-test example:

```text
examples/acf-jira-deletion-test.pending.mcp.json
```

This file is intentionally separated from the normal install path. For the locally inspected `mcp-atlassian` 0.23.1 package, source validation identified `jira_delete_issue` as the external Jira delete tool name, backed by `delete_issue(issue_key)`. Do not install or enable the profile until a human reviewer accepts deletion testing for an exact target issue.

Deletion-path development requirements:

- Keep `acf_jira` read-only.
- Keep `acfJiraWriterTest` non-destructive and unchanged.
- Use a separate profile name, `acfJiraDeletionTest`, only after review.
- Include only the minimum read tools plus `jira_delete_issue` for the locally validated `mcp-atlassian` 0.23.1 package.
- Do not include bulk, admin, workflow-configuration, permission, or Confluence tools.
- Do not physically delete any issue during profile setup or dry-run validation.

Completed development-path target:

```text
ATO-1469 - MCP writer-test cancellation and deletion workflow disposable story
Result: deleted and verified absent during the approved live deletion test
```

Any future physical deletion test requires a new exact disposable issue key, pre-delete retrieval, explicit approval naming the issue key and `jira_delete_issue`, one delete call only, and post-delete verification.

## 11. Troubleshooting

Use `skills/acf-jira-mcp-workflow/diagnostic-improvement-prompt.md` from this package source, or the copied equivalent in the installed skill directory, for troubleshooting.

Common stop points:

- VS Code does not discover the custom agent or skill.
- `uvx mcp-atlassian --help` fails.
- TLS fails because the environment requires OS certificate-store trust.
- MCP profile is duplicated or loaded from the wrong location.
- `ENABLED_TOOLS` contains unavailable or renamed Jira tool names.
- Jira token has Confluence-only access or insufficient Jira project permissions.
- The approved test issue or project is unavailable.

## 12. Final Validation Checklist

Before using the package for real issue work:

- [ ] Credal is configured as the VS Code custom OpenAI-compatible model provider.
- [ ] Credal token is stored through VS Code's language-model secret mechanism.
- [ ] Jira PAT is stored through the VS Code MCP password input.
- [ ] Credal and Jira credentials are separate.
- [ ] The only authoritative `acf_jira` server definition is in global VS Code `mcp.json`.
- [ ] Workspace MCP files do not contain a duplicate `acf_jira` definition.
- [ ] `uv --version`, `uvx --version`, and `uvx mcp-atlassian --help` succeed.
- [ ] MCP trust prompt shows command `uvx` and argument `mcp-atlassian`.
- [ ] `JIRA_URL` is `https://jira.acf.gov/`.
- [ ] `READ_ONLY_MODE=true` is configured for `acf_jira`.
- [ ] `ENABLED_TOOLS` contains only confirmed read-only Jira tools for the installed `mcp-atlassian` version.
- [ ] MCP startup output shows read-only mode enabled and a constrained discovered tool count.
- [ ] VS Code Chat session uses Local harness, Agent role, Credal custom model, and enabled `acf_jira` MCP server.
- [ ] The agent is visible in VS Code Chat or current source files are explicitly loaded for development source-mode testing.
- [ ] The bundled skill is visible or current source files are explicitly loaded for development source-mode testing.
- [ ] The read-only profile retrieves an approved test issue.
- [ ] The read-only profile does not advertise writer tools.
- [ ] Writer-test profile is separate and disabled unless needed.
- [ ] Exact Jira MCP tool names are validated for the installed `mcp-atlassian` version.
- [ ] No secrets are stored in files or prompts.
- [ ] No write, transition, comment, link, delete, bulk, or production-changing action is performed during read-only validation.

## 13. Unresolved Assumptions And Version-Dependent Items

- Exact VS Code command labels for MCP management can vary by build; use the target build's visible MCP UI labels when they differ.
- The Credal custom model JSON shape can vary by VS Code Custom Endpoint provider version; the token must still be stored through VS Code's supported language-model secret mechanism.
- The `mcp-atlassian` public Jira tool names must be revalidated after package upgrades before publishing a managed version of this guide.
- `UV_SYSTEM_CERTS=true` is environment-dependent and should remain only where needed for `uv` / `uvx` certificate trust.
- Any future write-enabled Jira workflow requires separate approval, separate review, and a minimum write-tool configuration. It is outside the read-only validation path.
