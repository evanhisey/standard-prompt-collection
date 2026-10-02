# End-User Installation Guide
## Microsoft VS Code + Credal + ACF Confluence MCP

**Document version:** 1.0.0  
**Audience:** End users configuring VS Code Chat with a Credal OpenAI-compatible model and ACF Confluence MCP access  
**Validated operating mode:** VS Code Chat, Local agent harness, Credal custom model, global VS Code MCP configuration, read-only Confluence page and image access, controlled writer-test profile, and controlled local image upload profile

This guide describes the configuration that was validated in the target environment. It intentionally documents only the supported VS Code Chat path:

```text
VS Code
  |
  +-- Local agent harness
  |     |
  |     +-- Credal custom OpenAI-compatible model
  |
  +-- VS Code user/global MCP configuration
        |
          +-- acf_confluence
              |
              +-- uvx mcp-atlassian
            +-- read-only Confluence page access
          |
          +-- acfConfluenceImageReadTest
            |
            +-- uvx mcp-atlassian
            +-- read-only Confluence attachment/image access
              +-- Confluence PAT stored through VS Code secret input
```

The MCP server is started by VS Code when needed. It is not a separate long-running service to install permanently.

> **Security requirement:** Do not paste a Credal token or Confluence Personal Access Token (PAT) into this guide, a Git repository, model JSON, `mcp.json` as a literal value, an environment file, a prompt, a ticket, a screenshot, logs, or source code.

> **If setup or verification fails:** Stop at the failing layer and use the troubleshooting section. Do not switch authentication methods, duplicate MCP configurations, paste secrets into diagnostics, or bypass TLS verification.

---

# 1. What You Are Installing

This deployment has three parts:

1. **VS Code Chat** using the **Local** session target / agent harness.
2. **Credal** configured as a custom OpenAI-compatible language model provider.
3. **`mcp-atlassian`** launched by VS Code as read-only Confluence MCP servers for page retrieval and image retrieval.

The validated runtime selection is:

- Session target / harness: **Local**
- Agent role: **Agent**
- Model: **Credal custom model**
- MCP servers: **acf_confluence** and **acfConfluenceImageReadTest**

Do not claim another harness is supported for authenticated Confluence MCP unless it is separately validated in the target environment.

---

# 2. Before You Start

Confirm all of the following before continuing:

- [ ] Microsoft VS Code is installed.
- [ ] VS Code Chat is available and can use custom language models in the target build.
- [ ] You can reach `https://confluence.acf.gov` from the workstation using the required ACF network/VPN connection.
- [ ] You can sign in to Confluence normally.
- [ ] You are allowed to create or use a Confluence Personal Access Token (PAT).
- [ ] You are allowed to install or run local developer tooling such as `uv` / `uvx`.
- [ ] You understand that the Credal token and the Confluence PAT are different credentials and must not be reused interchangeably.

If the Confluence UI does not provide Personal Access Tokens, stop and contact the appropriate support team. Do not substitute Basic authentication, username/password, or another token type unless explicitly approved for this environment.

---

# 3. Configure Credal As The Model Provider

Credal is the OpenAI-compatible model provider for this deployment.

Recommended provider: **Credal**. Use Credal as the VS Code custom OpenAI-compatible endpoint for this workflow unless your organization validates and approves a different provider. For the detailed Credal setup procedure, follow the internal ACF guide:

```text
https://confluence.acf.gov/spaces/tech/pages/219716713/VS+Code+GitHub+Copilot+Custom+Endpoint+Setup+for+Credal?src=contextnavpagetreemode
```

The endpoint values below are included so this guide remains self-contained, but the Confluence setup page is the recommended source for current VS Code UI steps and provider-specific fields.

Base endpoint:

```text
https://app.credal.acf.gov/api/openai
```

Depending on the VS Code Custom Endpoint provider version, the actual model entry may require the complete chat-completions endpoint:

```text
https://app.credal.acf.gov/api/openai/chat/completions
```

Configure the Credal custom model through the VS Code language-model management UI, such as:

```text
Chat: Manage Language Models
```

Use the model/provider fields required by the target VS Code build. Store or update the Credal bearer token through VS Code's language-model secret mechanism.

Security requirements:

- Do not hard-code the Credal bearer token in model JSON.
- Do not put the Credal token in `mcp.json`, an environment file, documentation, or source control.
- Do not use a literal header such as `"Authorization": "Bearer <token>"` in configuration.
- If the VS Code Custom Endpoint provider constructs the `Authorization` header from `apiKey`, prefer that behavior.
- The model configuration file should contain only the VS Code secret/input reference generated or managed by VS Code.
- Keep the Credal credential completely separate from the Confluence PAT.

After configuration, select the Credal custom model in VS Code Chat before running the MCP validation.

---

# 4. Install And Verify `uv` / `uvx`

`mcp-atlassian` is launched using `uvx`, which is provided by `uv`.

Use your organization's approved software distribution method when one is provided.

## 4.1 Windows

If WinGet is permitted, run in PowerShell:

```powershell
winget install --id=astral-sh.uv -e
```

When installation completes, close PowerShell and all VS Code windows, then open a new PowerShell window.

Verify:

```powershell
uv --version
uvx --version
where.exe uvx
uvx mcp-atlassian --help
```

Expected result:

- `uv --version` returns a version number.
- `uvx --version` returns a version number.
- `where.exe uvx` returns the path to `uvx.exe`.
- `uvx mcp-atlassian --help` returns command help and exits without error.

## 4.2 Linux

If the official `uv` installer is permitted, run:

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

Open a new terminal so the updated PATH is loaded.

Verify:

```bash
uv --version
uvx --version
command -v uvx
uvx mcp-atlassian --help
```

Expected result:

- `uv --version` returns a version number.
- `uvx --version` returns a version number.
- `command -v uvx` returns the path to `uvx`.
- `uvx mcp-atlassian --help` returns command help and exits without error.

If `uvx mcp-atlassian --help` fails with `invalid peer certificate: UnknownIssuer`, see the TLS troubleshooting section before continuing.

---

# 5. Configure The Global VS Code MCP Server

Use the VS Code user/global MCP configuration as the single source of truth for `acf_confluence` and `acfConfluenceImageReadTest`.

Validated Windows location:

```text
%APPDATA%\Code\User\mcp.json
```

Example resolved path from testing:

```text
C:\Users\<user>\AppData\Roaming\Code\User\mcp.json
```

Open the global MCP configuration using the VS Code command for your build. In validated builds this is typically available through the Command Palette as:

```text
MCP: Open User Configuration
```

If that exact command is unavailable, use the MCP configuration command exposed by the target VS Code build, or edit the global path above directly.

Do not create a duplicate `acf_confluence` or `acfConfluenceImageReadTest` server definition in:

- `.vscode/mcp.json`
- `.mcp.json`
- `~/.copilot/mcp-config.json`
- any other local or workspace-specific MCP file

A workspace `.vscode/mcp.json` may exist for other servers, but it should not contain duplicate Confluence server definitions when these servers are managed globally.

## 5.1 Confirm The Tool Allowlist For The Installed Version

Do not expose all `mcp-atlassian` tools. During validation, the server initially reported `Discovered 20 tools`, while the Credal/OpenAI-compatible endpoint later rejected a request containing 145 tools because the endpoint accepted a maximum of 128. The MCP log also reported that `TOOLSETS` was unset and all applicable toolsets were exposed.

Before publishing or approving this guide for a managed environment, verify the exact tool names exposed by the installed `mcp-atlassian` version. Use the MCP startup output and VS Code's server listing/output views as evidence. Remove unavailable or renamed tools, and do not include Jira tools unless the deployment explicitly requires Jira.

The following Confluence read-only tool names were used for this reviewed configuration and must be revalidated after `mcp-atlassian` upgrades:

```text
confluence_search,
confluence_get_page,
confluence_get_page_children,
confluence_get_page_history,
confluence_get_page_diff,
confluence_get_page_restrictions
```

At minimum, `confluence_get_page` was validated successfully by retrieving page `115220454`.

Images and screenshots are part of successful runbook retrieval. The following Confluence attachment/image read-only tool names were used for the reviewed image-read profile and must be revalidated after `mcp-atlassian` upgrades:

```text
confluence_get_page,
confluence_get_attachments,
confluence_download_attachment,
confluence_download_content_attachments,
confluence_get_page_images
```

The image-read profile remains read-only. It can list and retrieve existing page images, but it cannot upload, delete, or modify attachments.

## 5.2 Global `mcp.json` Pattern

Add or merge the following entries. If the file already has other `inputs` or `servers`, merge this input and these servers without replacing unrelated entries.

```json
{
  "inputs": [
    {
      "type": "promptString",
      "id": "acf-confluence-personal-token",
      "description": "ACF Confluence Personal Access Token",
      "password": true
    }
  ],
  "servers": {
    "acf_confluence": {
      "type": "stdio",
      "command": "uvx",
      "args": ["mcp-atlassian"],
      "env": {
        "CONFLUENCE_URL": "https://confluence.acf.gov",
        "CONFLUENCE_PERSONAL_TOKEN": "${input:acf-confluence-personal-token}",
        "READ_ONLY_MODE": "true",
        "ENABLED_TOOLS": "confluence_search,confluence_get_page,confluence_get_page_children,confluence_get_page_history,confluence_get_page_diff,confluence_get_page_restrictions",
        "UV_SYSTEM_CERTS": "true"
      }
    },
    "acfConfluenceImageReadTest": {
      "type": "stdio",
      "command": "uvx",
      "args": ["mcp-atlassian"],
      "env": {
        "CONFLUENCE_URL": "https://confluence.acf.gov",
        "CONFLUENCE_PERSONAL_TOKEN": "${input:acf-confluence-personal-token}",
        "READ_ONLY_MODE": "true",
        "ENABLED_TOOLS": "confluence_get_page,confluence_get_attachments,confluence_download_attachment,confluence_download_content_attachments,confluence_get_page_images",
        "UV_SYSTEM_CERTS": "true"
      }
    }
  }
}
```

Important notes:

- `CONFLUENCE_PERSONAL_TOKEN` uses a VS Code password input reference. The PAT is not stored as plaintext in `mcp.json`.
- `READ_ONLY_MODE=true` is required for both read-only profiles.
- `ENABLED_TOOLS` is required to keep the advertised MCP tools below the model endpoint's tool limit.
- `acfConfluenceImageReadTest` is separate from `acf_confluence` so image diagnostics can be validated without adding attachment tools to the general page reader.
- `UV_SYSTEM_CERTS=true` tells `uv` / `uvx` to use the operating-system certificate store. It is useful when `uvx` otherwise fails with `UnknownIssuer`. It does not disable TLS verification.
- Do not use `--insecure`, `verify=false`, `CONFLUENCE_SSL_VERIFY=false`, or any other TLS-bypass setting.

If `UV_SYSTEM_CERTS` is not needed in a future environment, it may be removed after `uvx mcp-atlassian --help` and MCP startup both succeed without certificate errors.

---

# 6. Enable The Global Server In VS Code

Defining `acf_confluence` and `acfConfluenceImageReadTest` globally and enabling them for a workspace/session are separate steps.

Use VS Code's MCP UI or Command Palette to manage the global server. Validated builds expose server management through:

```text
MCP: List Servers
```

Select `acf_confluence` and `acfConfluenceImageReadTest`, then choose actions such as:

```text
Start Server
Stop Server
Restart Server
Show Output
Show Configuration
```

Exact labels can vary by VS Code build. If a direct command such as `MCP: Start Server` or `MCP: Restart Server` exists, use it and select `acf_confluence` when prompted.

When VS Code prompts for **ACF Confluence Personal Access Token**, paste the PAT once. The same password input is reused by both read-only profiles. The input is hidden because `password` is `true`, and VS Code stores it securely.

Approve the MCP server trust prompt only after confirming:

- command: `uvx`
- argument: `mcp-atlassian`
- server name: `acf_confluence` or `acfConfluenceImageReadTest`
- read-only mode is configured
- the tool allowlist is configured

Useful non-secret startup evidence includes:

```text
Confluence configuration loaded and authentication is configured
Read-only mode: ENABLED
Discovered N tools
```

The discovered tool count should be small and should match the constrained Confluence allowlist for the installed version. If the count is unexpectedly high, stop and see the troubleshooting section for model endpoint tool-limit failures.

---

# 7. Read-Only Functional Validation

## 7.0 Install The Skill Package

The MCP profiles expose Confluence tools, but the runbook behavior comes from the packaged skill. Install the skill before asking VS Code Chat to standardize, write, or diagnose runbooks.

Recommended repository-scoped location:

```text
<target-workspace>/.github/skills/acf-confluence-runbook-standardizer/
```

Alternative workspace skill locations may be supported by the target VS Code build, such as:

```text
<target-workspace>/.agents/skills/acf-confluence-runbook-standardizer/
```

The installed skill directory must contain at least:

```text
SKILL.md
runbook-standard.md
mcp-workflow.md
validation-checklist.md
diagnostic-improvement-prompt.md
```

For simple installation from this package, copy the nested source directory:

```text
acf-confluence-runbook-standardizer/
```

to the target skill location so the result is:

```text
<target-workspace>/.github/skills/acf-confluence-runbook-standardizer/
├── SKILL.md
├── runbook-standard.md
├── mcp-workflow.md
├── validation-checklist.md
└── diagnostic-improvement-prompt.md
```

Keep the supporting files with `SKILL.md`; the skill explicitly reads them during runbook transformation, MCP writes, validation, and troubleshooting. Do not include maintainer-only files such as `eval-cases.md` in the installed skill unless the user is actively testing or developing the skill package.

After installing or updating the skill, start a new VS Code Chat session so the updated skill description and instructions can be discovered. Use this smoke-test prompt:

```text
Use the acf-confluence-runbook-standardizer skill.
Read only. Confirm which support files the skill will read before transforming or writing a Confluence runbook.
Do not modify Confluence.
```

Expected result:

- The skill is discovered by name.
- The response mentions `runbook-standard.md`, `mcp-workflow.md`, and `validation-checklist.md`.
- No Confluence write tool is called.

Use VS Code Chat with:

- Session target / harness: **Local**
- Agent role: **Agent**
- Model: **Credal custom model**
- MCP: **acf_confluence** and **acfConfluenceImageReadTest**

Run this read-only prompt:

```text
Use the Confluence MCP connection to retrieve page 115220454.

Read only. Do not create, update, copy, move, comment on, or delete anything.
Return only:
- page title
- page ID
- visible top-level section names
- whether retrieval succeeded
```

Expected result:

- page title: `NGSC Operation RunBook Template`
- page ID: `115220454`
- visible top-level sections include:
  - Brief
  - Prerequisites
  - Procedure
  - Version
- retrieval succeeded: yes

If this succeeds, the validated MCP path is working. If it fails, use troubleshooting rather than weakening security settings.

## 7.1 Image Read Validation

Run this read-only prompt after the basic page retrieval succeeds:

```text
Use the Confluence MCP image-read profile to inspect page 115220454.

Read only. Do not create, update, copy, upload, delete, move, comment on, or modify anything.
Return only:
- whether attachments were listed
- number of image attachments found
- number of images successfully retrieved
- any failed image retrievals
- whether image read validation succeeded
```

Expected result from validation:

- attachments were listed: yes
- image attachments found: `6`
- images successfully retrieved: `6`
- failed image retrievals: `0`
- image read validation succeeded: yes

Images and screenshots are part of runbook fidelity. If the page body can be retrieved but images cannot be listed or retrieved, the read installation is incomplete.

---

# 8. Optional Writer-Test MCP Installation

Install this profile only when writer testing has been explicitly approved. The default end-user profile remains `acf_confluence` in read-only mode.

The writer-test profile is intentionally separate so users can keep the known-good reader profile available while testing limited write behavior. Do not change `acf_confluence` from read-only to writable.

Use this profile only for approved copy-and-update tests in an approved Confluence test location. It is not a general publishing profile.

## 8.1 Writer-Test Entry Requirements

Before installing or enabling the writer-test profile, confirm:

- [ ] The read-only `acf_confluence` validation in this guide passes.
- [ ] The target test space and parent page have been approved for writer testing.
- [ ] Your Confluence account is authorized to create and edit pages in that test location.
- [ ] A human reviewer is identified for the writer test.
- [ ] You understand that every write requires explicit approval in chat before the agent calls a write tool.
- [ ] You will not use the writer-test profile against an existing operational page until the staged copy-and-populate workflow has been reviewed.

If any item is false, stop and use only the read-only profile.

## 8.2 Add The Separate Writer-Test Server

Open the same global VS Code MCP configuration used for the read-only server:

```text
%APPDATA%\Code\User\mcp.json
```

Merge the writer-test server into the existing JSON. Do not paste the example as a second JSON document at the bottom of the file.

JSON rules to watch:

- The file must start with one `{` and end with one matching `}`.
- There is one top-level `"inputs"` array and one top-level `"servers"` object.
- Reuse the existing Confluence PAT input unless your team intentionally approves separate reader and writer tokens.
- Add the writer server as another property inside `"servers"`.
- Put a comma between sibling items, but not after the last item in an array or object.

If your file already contains the read-only `acf_confluence` server from this guide, the combined structure should look like this:

```json
{
  "inputs": [
    {
      "type": "promptString",
      "id": "acf-confluence-personal-token",
      "description": "ACF Confluence Personal Access Token",
      "password": true
    }
  ],
  "servers": {
    "acf_confluence": {
      "type": "stdio",
      "command": "uvx",
      "args": ["mcp-atlassian"],
      "env": {
        "CONFLUENCE_URL": "https://confluence.acf.gov",
        "CONFLUENCE_PERSONAL_TOKEN": "${input:acf-confluence-personal-token}",
        "READ_ONLY_MODE": "true",
        "ENABLED_TOOLS": "confluence_search,confluence_get_page,confluence_get_page_children,confluence_get_page_history,confluence_get_page_diff,confluence_get_page_restrictions",
        "UV_SYSTEM_CERTS": "true"
      }
    },
    "acfConfluenceWriterTest": {
      "type": "stdio",
      "command": "uvx",
      "args": ["mcp-atlassian"],
      "env": {
        "CONFLUENCE_URL": "https://confluence.acf.gov",
        "CONFLUENCE_PERSONAL_TOKEN": "${input:acf-confluence-personal-token}",
        "READ_ONLY_MODE": "false",
        "ENABLED_TOOLS": "confluence_search,confluence_get_page,confluence_copy_page,confluence_update_page_section,confluence_update_page,confluence_get_page_history,confluence_get_page_diff",
        "UV_SYSTEM_CERTS": "true"
      }
    }
  }
}
```

The comma after the `acf_confluence` server is required because another server follows it. The writer-test server is last in this example, so it does not have a trailing comma.

The same input id, `acf-confluence-personal-token`, is used by both profiles. That means VS Code prompts for one Confluence PAT and passes it to either MCP server when that server starts. This is the simplest end-user setup when the same Confluence account is approved for both read-only validation and writer testing.

Optional separate-token setup: if your team wants a different PAT for writer testing, add a second item inside `"inputs"` with a unique id such as `acf-confluence-writer-test-personal-token`, then change only the writer-test server's `CONFLUENCE_PERSONAL_TOKEN` value to `${input:acf-confluence-writer-test-personal-token}`. Do not create two `inputs` entries with the same `id`.

If your `mcp.json` contains other MCP servers, keep them and add commas using the same pattern.

Common JSON error to avoid:

```text
}
{
  "servers": {
    "acfConfluenceWriterTest": {}
  }
}
```

That is invalid because it creates two separate top-level JSON objects. Put `acfConfluenceWriterTest` inside the existing `"servers"` object instead.

The same example is provided in this package at:

```text
examples/acf-confluence-writer-test.mcp.json
```

Important requirements:

- Keep the server name exactly `acfConfluenceWriterTest`.
- Keep `READ_ONLY_MODE=false` only on this writer-test profile.
- Keep `acf_confluence` unchanged with `READ_ONLY_MODE=true`.
- Keep `CONFLUENCE_PERSONAL_TOKEN` pointed at the existing Confluence PAT input unless a separate writer-test token has been deliberately approved.
- Keep the writer-test `ENABLED_TOOLS` allowlist narrow.
- Use `confluence_update_page` only for reviewed storage-aware panel updates that preserve macros, images, and layout. Do not use it for broad replacement or convenience rewrites.
- Do not add delete, move, restriction, comment, attachment, or Jira write tools.
- Do not paste the PAT directly into `mcp.json`; use the VS Code password input reference.

## 8.3 Enable And Verify The Writer-Test Server

Use VS Code's MCP UI or Command Palette:

```text
MCP: List Servers
```

Select `acfConfluenceWriterTest`, then start or restart the server. When VS Code prompts for **ACF Confluence Personal Access Token for writer-test MCP profile**, paste the PAT once.

Approve the MCP server trust prompt only after confirming:

- command: `uvx`
- argument: `mcp-atlassian`
- server name: `acfConfluenceWriterTest`
- read-only mode is disabled only for this writer-test profile
- the exposed tool list is limited to the approved writer-test allowlist

Useful non-secret startup evidence includes:

```text
Confluence configuration loaded and authentication is configured
Read-only mode: DISABLED
Discovered N tools
```

The discovered tool count should be small. If the output shows broad Jira tools, delete/move/restriction/comment/attachment tools, or a large unconstrained tool list, stop and correct `ENABLED_TOOLS` before using the profile.

## 8.4 Writer-Test Validation Prompt

Use this prompt after the writer-test server starts. It validates that the copy tool is available without authorizing a write.

```text
Use the acf-confluence-runbook-standardizer skill in writer-test mode.

Do not write yet.
Confirm whether the active MCP session exposes the approved writer-test profile named acfConfluenceWriterTest and whether confluence_copy_page is available.

Retrieve source template page 115220454 and the approved test parent page.
Show the proposed copy source, destination parent, and new title.
State the exact MCP write tool you intend to call.
Then stop and wait for my explicit approval before any write.
```

For the first actual write test, use only `confluence_copy_page` to copy template page `115220454` into an approved test parent. After the copy, retrieve the new page, confirm the source template is unchanged, and report the new page URL and ID.

Do not proceed to section updates until the copy-only stage passes and the user gives a separate explicit approval.

## 8.5 Writer-Test Stop Conditions

Stop without writing if any of these occur:

- The active server is still the read-only `acf_confluence` profile.
- `confluence_copy_page` is not available.
- The destination parent is unclear or not approved.
- The proposed page title is unclear.
- The user has not explicitly approved the write.
- The requested operation would modify an existing operational page.
- The operation requires full-page replacement, delete, move, restrictions, comments, attachments, or Jira writes.

Stopping is the correct outcome when the writer-test profile is not available or the requested write is outside the approved tool set.

---

# 9. Optional Image Upload Test Profile

Do not enable image upload tools as part of the default read-only installation. Start with the read-only `acfConfluenceImageReadTest` validation in section 7.1.

Use a separate image upload profile only when all of the following are true:

- [ ] Basic read-only page retrieval passed.
- [ ] Image read validation passed against source page `115220454`.
- [ ] The target page is confirmed and its current image references/attachments are understood.
- [ ] The target page is an approved test page, not an operational page.
- [ ] A human explicitly approves uploading the exact local image file or files onto that exact test page.
- [ ] The upload test profile is started only for the upload test and stopped afterward.

## 9.1 Add The Separate Image Upload Test Server

Merge this server into the existing global `mcp.json` only for approved image upload development. Keep `acf_confluence`, `acfConfluenceImageReadTest`, and `acfConfluenceWriterTest` unchanged.

```json
{
  "servers": {
    "acfConfluenceImageUploadTest": {
      "type": "stdio",
      "command": "uvx",
      "args": ["mcp-atlassian"],
      "env": {
        "CONFLUENCE_URL": "https://confluence.acf.gov",
        "CONFLUENCE_PERSONAL_TOKEN": "${input:acf-confluence-personal-token}",
        "READ_ONLY_MODE": "false",
        "ENABLED_TOOLS": "confluence_get_page,confluence_get_attachments,confluence_download_attachment,confluence_download_content_attachments,confluence_get_page_images,confluence_upload_attachment,confluence_upload_attachments",
        "UV_SYSTEM_CERTS": "true"
      }
    }
  }
}
```

This snippet is not a complete `mcp.json` file. Add `acfConfluenceImageUploadTest` as another server inside the existing top-level `"servers"` object.

Do not include `confluence_delete_attachment` in the upload-test profile unless a separate delete test is explicitly approved.

## 9.2 Upload-Test Development Sequence

After adding or changing the upload-test profile, re-confirm the target page ID, title, space, current version, current attachments, and current image references. The image upload test depends on a known approved target page and an explicitly designated local image path or directory.

Local-file upload is the primary upload path. This is the expected runbook workflow for screenshots, diagrams, exported images, and transient files created during page updates. Use `content_base64` only as a fallback for MCP-internal attachment copying or environments where the MCP server cannot read local files.

Current `mcp-atlassian 0.23.1` supports uploading images from a user-designated local directory. Existing Confluence page updates should preserve Confluence-hosted image references and attachments in place rather than exporting them locally. If the team later requires automated Confluence export into local staging, deliver that capability through an approved upstream `mcp-atlassian` release or an approved internally packaged MCP fork. Do not patch an individual user's `uvx` cache as part of installation.

Use a page-specific staging directory such as:

```text
.image-upload-staging/<page-id>/
```

The staging directory may be temporary. Keep it only if the user wants audit evidence; otherwise remove it after post-upload validation.

Recommended sequence:

1. Keep `acf_confluence` and `acfConfluenceImageReadTest` unchanged.
2. Restart VS Code MCP servers so `acfConfluenceImageUploadTest` is loaded.
3. Confirm the approved target page ID, title, space, current version, and current attachment list.
4. Retrieve the target page raw storage and current page images.
5. Confirm the user-designated local image source path or directory.
6. Verify filename, file size, extension, and image signature before upload.
7. Copy the required user-designated local images into the page-specific staging directory when staging is needed.
8. Show the exact filenames, source path or directory, staged file paths when used, file sizes, and destination page ID before upload.
9. Stop and wait for explicit approval naming `acfConfluenceImageUploadTest` and `confluence_upload_attachments`.
10. Upload only those approved files to the target page.
11. Append or update page storage only when the user separately approves the page-body change.
12. Re-run image read validation against the target page.
13. Confirm the uploaded target-local image resolves and that existing image references remain intact.
14. Remove the staging directory, or report that it was retained for audit.

Stop before upload if the target page ID, source image list, filenames, or approval are unclear.

---

# 10. Optional Direct REST Diagnostic

Use this only when you need to separate network/TLS/PAT problems from VS Code MCP startup problems. This diagnostic must not print the PAT or dump page content.

PowerShell:

```powershell
$patSecure = Read-Host "Paste Confluence PAT" -AsSecureString
$ptr = [Runtime.InteropServices.Marshal]::SecureStringToBSTR($patSecure)
try {
    $pat = [Runtime.InteropServices.Marshal]::PtrToStringBSTR($ptr)
    $headers = @{ Authorization = "Bearer $pat" }

    $currentUser = Invoke-WebRequest `
        -Uri "https://confluence.acf.gov/rest/api/user/current" `
        -Headers $headers `
        -UseBasicParsing
    "Current user status: $($currentUser.StatusCode)"

    $page = Invoke-RestMethod `
        -Uri "https://confluence.acf.gov/rest/api/content/115220454?expand=title" `
        -Headers $headers `
        -UseBasicParsing
    "Page status: 200"
    "Page title: $($page.title)"
}
finally {
    if ($ptr) { [Runtime.InteropServices.Marshal]::ZeroFreeBSTR($ptr) }
    Remove-Variable patSecure, ptr, pat, headers, currentUser, page -ErrorAction SilentlyContinue
}
```

Expected diagnostic result from validation:

- `GET https://confluence.acf.gov/rest/api/user/current` returned HTTP 200.
- `GET https://confluence.acf.gov/rest/api/content/115220454` returned HTTP 200.
- Page title was `NGSC Operation RunBook Template`.

This proves network connectivity, TLS, Bearer PAT authentication, and page access. It does not replace the MCP functional validation.

---

# 11. Troubleshooting

Troubleshoot one layer at a time. Do not paste secrets into chat or logs.

## 11.1 401 Authentication Failure

A 401 indicates that Confluence did not accept the credential.

Check:

- The VS Code PAT prompt appeared and was completed.
- `CONFLUENCE_PERSONAL_TOKEN` in `mcp.json` is exactly `${input:acf-confluence-personal-token}`.
- The PAT has not expired or been revoked.
- The PAT belongs to the same account that can open the target Confluence page in a browser.
- The Credal token was not reused as the Confluence PAT.

Do not switch to Basic authentication or username/password as a fallback.

## 11.2 TLS `UnknownIssuer`

If `uvx mcp-atlassian --help` fails while fetching or launching with `invalid peer certificate: UnknownIssuer`, verify whether the OS certificate store resolves the issue:

```powershell
uvx --system-certs mcp-atlassian --help
```

If that succeeds, keep this in the MCP server `env` block:

```text
"UV_SYSTEM_CERTS": "true"
```

This causes `uv` / `uvx` to use the operating-system certificate store. It is not a TLS-verification bypass.

Do not use `--insecure`, `verify=false`, or disabled SSL verification settings.

## 11.3 MCP Server Not Starting

Check:

- `uvx mcp-atlassian --help` succeeds outside VS Code.
- `uvx` is available on the PATH visible to VS Code.
- The global `%APPDATA%\Code\User\mcp.json` file is valid JSON.
- The server name is exactly `acf_confluence`.
- The command is `uvx` and args are `["mcp-atlassian"]`.
- The PAT prompt was not cancelled.
- The workstation can reach `https://confluence.acf.gov`.

Use `MCP: List Servers` and select `Show Output` or `Show Configuration` when available. Exact command labels may vary by VS Code build.

`MCP_VERBOSE=true` may be added temporarily for troubleshooting, but do not make verbose logging permanent unless justified. Do not share logs that contain secrets, Authorization headers, cookies, or PAT values.

## 11.4 Tool Not Registered

If VS Code reports that a tool is unknown or unavailable:

- Check the MCP output for the discovered tool names and count.
- Confirm the name in `ENABLED_TOOLS` exactly matches the installed `mcp-atlassian` public tool name.
- Remove unavailable or renamed tools from `ENABLED_TOOLS`.
- Restart `acf_confluence` after editing `mcp.json`.
- Keep `confluence_get_page` in the allowlist because it was validated for the functional test.

For writer testing, confirm the active server is `acfConfluenceWriterTest` and the allowlist includes `confluence_copy_page`. Do not add broad toolsets or switch the read-only server to writable mode to make writer tools appear.

Do not add broad toolsets or Jira tools merely to make one tool appear.

## 10.5 No `CallToolRequest` / Tool Not Invoked

If the MCP server starts but no tool call appears in the MCP output:

- Confirm the session target / harness is **Local**.
- Confirm VS Code Chat is in **Agent** role.
- Confirm the selected model is the Credal custom model.
- Confirm `acf_confluence` is enabled for the workspace/session, not merely defined globally.
- Ask for a direct read-only retrieval of page `115220454` rather than a broad task.
- Review the model endpoint response for tool-limit or tool-schema errors.

Do not create a second local MCP configuration just to force enablement of the global server.

## 10.6 Model Endpoint Reports More Than 128 Tools

If the Credal/OpenAI-compatible endpoint rejects a request because too many tools were supplied:

- Confirm `ENABLED_TOOLS` is set in the `acf_confluence` `env` block.
- Confirm `TOOLSETS` is not causing all Jira and Confluence toolsets to be exposed.
- Restart `acf_confluence`.
- Check the MCP startup output for `Discovered N tools`.
- Keep only the minimum confirmed read-only Confluence tools needed for this workflow.

Do not rely on `READ_ONLY_MODE=true` alone to reduce advertised tool count.

---

# 11. Validated Configuration

## Requirements

- VS Code Chat with custom language-model support.
- Local session target / agent harness.
- Credal custom OpenAI-compatible model.
- Credal endpoint `https://app.credal.acf.gov/api/openai`, or the complete chat-completions endpoint when required by the VS Code Custom Endpoint provider.
- Credal token stored by VS Code's language-model secret mechanism.
- Global VS Code MCP configuration at `%APPDATA%\Code\User\mcp.json` on Windows.
- `acf_confluence` server using `uvx mcp-atlassian`.
- Confluence PAT stored through a VS Code MCP password input.
- `READ_ONLY_MODE=true`.
- Explicit `ENABLED_TOOLS` allowlist.
- Optional `acfConfluenceWriterTest` server only when writer testing is approved.
- Writer-test profile with `READ_ONLY_MODE=false` and only the approved copy/section-update/history/diff tools.
- Human review before changing production or managed deployment standards.

## Environment-Specific Findings

- Direct REST validation with a valid Confluence PAT returned HTTP 200 for `GET https://confluence.acf.gov/rest/api/user/current`.
- Direct page retrieval returned HTTP 200 for page `115220454` with title `NGSC Operation RunBook Template`.
- The MCP configuration successfully retrieved page `115220454` when run through the Local harness.
- The successful page retrieval showed top-level sections including Brief, Prerequisites, Procedure, and Version.
- During validation, `mcp-atlassian` initially reported `Discovered 20 tools`.
- The Credal/OpenAI-compatible endpoint rejected a later request containing 145 tools because the endpoint accepted a maximum of 128.
- MCP output showed that `TOOLSETS` was unset and all applicable toolsets were exposed.

## Troubleshooting-Only Settings

- `MCP_VERBOSE=true` may be used temporarily to collect startup details.
- `UV_SYSTEM_CERTS=true` is needed only where `uv` / `uvx` otherwise fails with `UnknownIssuer`; it uses the operating-system certificate store and does not disable TLS verification.
- Optional direct REST diagnostics may be used to isolate network/TLS/PAT failures, but only with secure PAT input and status/title-only output.

---

# 12. Changes From The Previous Guide

- Removed all Codex deployment instructions because Codex is not part of the validated deployment.
- Removed the `[mcp_servers.acf_confluence]` TOML configuration block because the authoritative MCP configuration is VS Code user/global `mcp.json`.
- Removed instructions implying `config.toml` is needed for Confluence MCP because this deployment uses VS Code MCP configuration.
- Corrected the Windows MCP configuration path to `%APPDATA%\Code\User\mcp.json` with the tested resolved path pattern `C:\Users\<user>\AppData\Roaming\Code\User\mcp.json`.
- Consolidated MCP configuration around a single global `acf_confluence` definition to avoid duplicate server definitions in workspace or local config files.
- Moved Credal authentication to the VS Code language-model secret mechanism so the bearer token is not hard-coded.
- Moved Confluence PAT handling to a VS Code MCP password input so the PAT is not written as plaintext.
- Documented **Local** as the validated agent harness because that is the path that successfully retrieved Confluence content.
- Added initial end-user instructions for installing the separate `acfConfluenceWriterTest` MCP profile without changing the default read-only profile.
- Added a clear distinction between defining the global MCP server and enabling it for a workspace/session.
- Added an explicit `ENABLED_TOOLS` filter because the endpoint rejected requests exceeding its 128-tool maximum and `READ_ONLY_MODE` alone does not sufficiently limit advertised tools.
- Added tool-name revalidation requirements for installed `mcp-atlassian` versions rather than assuming a stale tool list will remain valid.
- Kept `UV_SYSTEM_CERTS=true` explanation accurate: it uses the OS certificate store and does not disable TLS verification.
- Reframed direct REST testing as optional and security-conscious, using `Read-Host -AsSecureString` and status/title-only output.
- Added concise troubleshooting branches for 401 authentication failure, TLS `UnknownIssuer`, MCP startup failure, missing tools, missing tool invocation, and model endpoint tool-limit failures.
- Removed temporary session environment-variable setup from the normal installation flow because the validated path uses VS Code-managed secret inputs.

---

# 12. Final Validation Checklist

- [ ] No Codex dependency or Codex MCP configuration remains in the supported installation path.
- [ ] Credal is configured as the VS Code custom OpenAI-compatible model provider.
- [ ] Credal token is stored through VS Code's language-model secret mechanism.
- [ ] Confluence PAT is stored through the VS Code MCP password input.
- [ ] Credal and Confluence credentials are separate.
- [ ] The only authoritative `acf_confluence` server definition is in global VS Code `mcp.json`.
- [ ] Workspace MCP files do not contain a duplicate `acf_confluence` definition.
- [ ] `uv --version`, `uvx --version`, and `uvx mcp-atlassian --help` succeed.
- [ ] MCP trust prompt shows command `uvx` and argument `mcp-atlassian`.
- [ ] `READ_ONLY_MODE=true` is configured.
- [ ] `ENABLED_TOOLS` contains only confirmed read-only Confluence tools for the installed `mcp-atlassian` version.
- [ ] MCP startup output shows read-only mode enabled and a constrained discovered tool count.
- [ ] VS Code Chat session uses Local harness, Agent role, Credal custom model, and enabled `acf_confluence` MCP server.
- [ ] Read-only retrieval of page `115220454` succeeds.
- [ ] Retrieved page title is `NGSC Operation RunBook Template`.
- [ ] Visible top-level sections include Brief, Prerequisites, Procedure, and Version.
- [ ] No write, copy, move, comment, delete, or production-changing action is performed during validation.

---

# 13. Unresolved Assumptions And Version-Dependent Items

- Exact VS Code command labels for MCP management can vary by build; use the target build's visible MCP UI labels when they differ.
- The Credal custom model JSON shape can vary by VS Code Custom Endpoint provider version; the token must still be stored through VS Code's supported language-model secret mechanism.
- The `mcp-atlassian` public tool names must be revalidated after package upgrades before publishing a managed version of this guide.
- `UV_SYSTEM_CERTS=true` is environment-dependent and should remain only where needed for `uv` / `uvx` certificate trust.
- Any future write-enabled Confluence workflow requires separate approval, separate review, and a minimum write-tool configuration. It is outside this read-only deployment guide.
