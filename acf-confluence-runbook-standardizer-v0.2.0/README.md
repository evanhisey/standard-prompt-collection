# ACF Confluence Runbook Standardizer

**Version:** 1.0.0

This package contains VS Code Chat agent instructions for standardizing ACF Confluence operational runbooks against the NGSC Operation RunBook Template while using local Confluence MCP profiles for live retrieval, controlled writer tests, and controlled image handling.

## Start here

For an end-user installation, follow:

**[`INSTALL.md`](INSTALL.md)**

It now contains the validated VS Code Chat + Credal + Confluence MCP instructions covering:

- installing and verifying `uv` / `uvx`;
- testing `mcp-atlassian` before starting the MCP server;
- configuring Credal as the VS Code custom OpenAI-compatible model provider;
- storing the Credal token through VS Code's language-model secret mechanism;
- configuring the global VS Code `mcp.json` server definition for `acf_confluence`;
- configuring the separate read-only `acfConfluenceImageReadTest` profile for attachment and image validation;
- storing the Confluence PAT through the VS Code MCP password input;
- restricting the MCP connection to confirmed read-only Confluence tools;
- optionally installing the separate `acfConfluenceWriterTest` MCP profile for approved writer testing;
- validating access against Confluence template page `115220454`;
- troubleshooting PATH, authentication, TLS, tool visibility, tool invocation, and model endpoint tool-limit problems.
- installing the packaged `acf-confluence-runbook-standardizer/` VS Code Copilot skill directory.

## What this version does

- Uses Confluence page `115220454` as the live formatting baseline.
- Supports dry-run review and standardized drafts with a read-only MCP server.
- Supports create-from-template and controlled update workflows only when an approved writer-test profile is deliberately configured.
- Supports two validated image paths: preserving existing Confluence-hosted image references and uploading user-designated local images to an approved target page.
- Preserves technical commands and identifiers unless the user explicitly requests a technical change.
- Requires human review before publishing/approval.
- Avoids embedding credentials or PATs in the skill.
- Includes a reusable diagnostic-and-improvement prompt for future support and documentation updates.

## Package contents

```text
acf-confluence-runbook-standardizer/
├── acf-confluence-runbook-standardizer/
│   ├── SKILL.md
│   ├── diagnostic-improvement-prompt.md
│   ├── mcp-workflow.md
│   ├── runbook-standard.md
│   └── validation-checklist.md
├── README.md
├── INSTALL.md
├── CHANGELOG.md
├── examples/
│   ├── acf-confluence-image-upload-test.mcp.json
│   └── acf-confluence-writer-test.mcp.json
└── eval-cases.md
```

The nested `acf-confluence-runbook-standardizer/` directory is the copy-ready VS Code skill payload. Copy that directory to the target workspace skill location. The package root also keeps maintainer documentation, examples, changelog, and evaluation cases that are useful for release work but are not required at runtime.

## Quick installation summary

The detailed steps are in `INSTALL.md`. At a high level:

1. Install `uv` / `uvx`.
2. Confirm `uvx mcp-atlassian --help` works.
3. Configure Credal as the VS Code custom OpenAI-compatible model provider and store its token through VS Code's language-model secret mechanism.
4. Add the read-only `acf_confluence` and `acfConfluenceImageReadTest` MCP servers to the global VS Code user `mcp.json` file.
5. Store the Confluence PAT through the VS Code MCP password input when prompted.
6. Enable `acf_confluence` for the Local VS Code Chat agent session.
7. Confirm the MCP can retrieve Confluence page `115220454`.
8. Confirm the image-read profile can list and retrieve the six template images from page `115220454`.
9. Run the read-only validation prompts from `INSTALL.md`.
10. Copy the nested `acf-confluence-runbook-standardizer/` skill directory to the target workspace under `.github/skills/acf-confluence-runbook-standardizer/` or the approved workspace skill location for the target VS Code build.
11. If writer testing is approved, add the separate `acfConfluenceWriterTest` profile from `INSTALL.md` without changing the read-only profiles.

## First skill test

```text
Use the acf-confluence-runbook-standardizer skill.
Review and standardize this page in draft mode only:
https://confluence.acf.gov/spaces/tech/pages/219712327/NGSC+Workspace+EKS+access+Needs+decoupling+from+cfseer

Do not modify Confluence.
Use live page 115220454 as the formatting baseline.
Show the proposed standardized runbook, missing information, change summary, and validation results.
```

## Diagnostic and future-improvement prompt

When a user encounters an installation/MCP problem, use:

**[`acf-confluence-runbook-standardizer/diagnostic-improvement-prompt.md`](acf-confluence-runbook-standardizer/diagnostic-improvement-prompt.md)**

The prompt is designed to:

- diagnose one layer at a time;
- keep troubleshooting read-only;
- prevent PAT disclosure;
- prohibit TLS-disable shortcuts;
- distinguish runtime, configuration, network, authentication, authorization, and skill-discovery failures;
- produce an exact documentation improvement and a regression/eval case for a future release.

## Publishing

Do not use production publishing as the first test. Keep the normal review profile read-only.

If writer testing is authorized, use the separately reviewed `acfConfluenceWriterTest` MCP profile with only the minimum required write tools. See `examples/acf-confluence-writer-test.mcp.json`, `examples/acf-confluence-image-upload-test.mcp.json`, and `acf-confluence-runbook-standardizer/mcp-workflow.md`.

## Review status

This package reflects the validated VS Code/Credal/Confluence MCP workflow as of this release. It is not itself an ACF/HHS policy document. Review and approve the instructions, installation procedure, tool allowlist, secret-handling approach, and any publishing workflow before organizational deployment.
