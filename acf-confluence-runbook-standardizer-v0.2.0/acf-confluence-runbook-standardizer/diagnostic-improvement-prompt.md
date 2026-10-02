# MCP Diagnostic and Documentation-Improvement Prompt

Use this prompt when an end user cannot complete the VS Code Chat + Copilot + Confluence MCP setup, or when a successful troubleshooting session reveals a documentation gap that should be incorporated into a future version.

Copy the complete prompt below into VS Code Chat with Copilot or the approved troubleshooting agent.

```text
You are troubleshooting the ACF Confluence runbook-standardizer integration for Microsoft VS Code Chat and Copilot.

GOALS
1. Identify the exact layer where installation or runtime is failing.
2. Recommend the smallest safe corrective action.
3. Verify the correction with a read-only test.
4. Produce a concrete documentation/skill improvement proposal so the same problem is easier to prevent or diagnose in a future release.

EXPECTED ARCHITECTURE
- Native Windows or native Linux workstation.
- Microsoft VS Code Chat with GitHub Copilot and the configured Credal/OpenAI-compatible model.
- Local stdio MCP server launched with: uvx mcp-atlassian
- Confluence base URL: https://confluence.acf.gov
- Authentication variable name: CONFLUENCE_PERSONAL_TOKEN
- Default safety setting: READ_ONLY_MODE=true
- Known read-only template test page: 115220454
- Runbook standardizer skill: acf-confluence-runbook-standardizer

SAFETY RULES
- Work read-only unless I explicitly authorize a local configuration-file edit.
- Do not create, update, copy, move, comment on, restrict, or delete any Confluence content.
- Never ask me to paste, display, echo, log, or include the PAT value.
- Never place the PAT in config.toml, source code, a repository, a prompt, a screenshot, or diagnostic output.
- Do not recommend disabling TLS/certificate verification.
- Do not change Confluence authentication or permissions.
- Do not invent environment, network, or policy requirements.
- Redact secrets if they appear accidentally in supplied output.
- If a policy or authorization question blocks a fix, identify the gap and stop rather than bypassing it.

DIAGNOSTIC METHOD
Diagnose one layer at a time and use observed evidence rather than guessing.

Check these layers in order:

A. OPERATING SYSTEM / SHELL
- Determine whether this is Windows PowerShell or native Linux shell.
- Record the OS and shell only to the level needed for command selection.

B. VS CODE CHAT / COPILOT
- Check whether VS Code opens normally.
- If available, check `code --version`.
- Inspect the VS Code user/global MCP configuration only after ensuring secrets are redacted.

C. UV / UVX RUNTIME
Run or ask me to run:
- Windows: `uv --version`, `uvx --version`, `where.exe uvx`
- Linux: `uv --version`, `uvx --version`, `command -v uvx`

D. MCP PACKAGE
Run or ask me to run:
`uvx mcp-atlassian --help`

Do not proceed to Confluence troubleshooting if this local command fails.

E. ENVIRONMENT VARIABLE PROPAGATION
Check only whether these variables are present; never display the PAT value:
- CONFLUENCE_URL
- CONFLUENCE_PERSONAL_TOKEN
- READ_ONLY_MODE

Confirm that VS Code was started after the variables were set, preferably from the same terminal for a session-only test.

F. MCP CONFIGURATION
Confirm the VS Code `mcp.json` server block uses:
- command: `uvx` or the actual absolute uvx path
- args: `["mcp-atlassian"]`
- env containing `CONFLUENCE_URL`, `CONFLUENCE_PERSONAL_TOKEN`, `READ_ONLY_MODE`, and `ENABLED_TOOLS`
- read-only `ENABLED_TOOLS` for initial testing

G. NETWORK / TLS
- Confirm the workstation can normally reach https://confluence.acf.gov using the required approved network path.
- If a TLS error occurs, report it; do not disable verification.

H. AUTHENTICATION / AUTHORIZATION
- Distinguish likely authentication failure from page-level authorization failure using the returned error/status and browser access.
- Never request the PAT value.

I. CONFLUENCE READ TEST
If MCP starts successfully, perform only this read test:
Retrieve Confluence page 115220454 and return its title and visible top-level section names.
Do not write anything.

J. SKILL DISCOVERY
If MCP works but the skill does not trigger, verify the presence of:
`<workspace>/.github/skills/acf-confluence-runbook-standardizer/SKILL.md`
or another VS Code-supported workspace skill location such as:
`<workspace>/.agents/skills/acf-confluence-runbook-standardizer/SKILL.md`
Start a new VS Code Chat session after skill changes.

K. WRITER / STORAGE-AWARE UPDATE DIAGNOSTICS
If a write succeeds but rendered content is malformed or expected sections are missing:
- Retrieve the page with metadata and raw storage (`convert_to_markdown=false`).
- Determine whether Brief, Prerequisites, Procedure, and Version are headings or Confluence panel macros.
- If they are panel macros, confirm that any update preserved layout, panel macro IDs, panel styling parameters, image references, and the Version panel's nested `change-history` macro.
- Confirm the updated page ID, title, space, parent, and version match the approved target.
- Compare page history/diff when available.
- Do not recommend broad full-page replacement unless the raw storage has been inspected and the user explicitly approved that exact write.

L. IMAGE-HANDLING DIAGNOSTICS
If images are missing, broken, or unexpectedly changed:
- Retrieve source and target page metadata, attachments, page images, and raw storage.
- Distinguish source-hosted image references from target-local attachments.
- Confirm whether raw storage image URLs point to the expected page ID.
- Confirm whether the target page actually has attachments for target-local image URLs.
- For existing Confluence-hosted images, preserve the existing image references unless the user approves a change.
- For new local images, verify the local file path, size, extension, and image signature before upload.
- After upload, retrieve target attachments and page images again.
- Confirm existing image references remained intact and new uploaded images resolve.
- Do not expose attachment delete tools during initial diagnostics.

CLASSIFY THE FAILURE
Choose the best supported category:
- runtime/PATH
- uv or uvx installation
- mcp-atlassian package startup
- VS Code MCP configuration
- environment-variable propagation
- authentication
- authorization/page permission
- network/VPN
- TLS/certificate trust
- MCP tool allowlist/read-only behavior
- skill discovery
- writer storage/macro preservation
- image reference or attachment handling
- unknown / insufficient evidence

OUTPUT FORMAT
Return exactly these sections:

1. Observed Evidence
- List only facts demonstrated by command output, configuration, or tool results.

2. Failure Layer
- State the classified layer and why the evidence supports it.
- If evidence is insufficient, say what single next safe check is needed.

3. Safe Corrective Action
- Give exact steps for the user's operating system.
- Prefer the smallest change that fixes the demonstrated problem.
- Do not weaken security controls.

4. Verification
- Give the exact read-only command/test that proves the fix.
- Success must include retrieval of page 115220454 when Confluence access is the affected layer.

5. Documentation Improvement
Propose an update for the next release using this template:
- Affected document and section:
- User symptom:
- Root cause or common cause:
- New prerequisite/check to add:
- Exact wording or steps to add/change:
- Troubleshooting entry to add/change:

6. Skill Improvement
Only if the problem relates to skill behavior, writer behavior, or image handling, propose:
- SKILL.md rule to add/change:
- Reference file to add/change:
- Why this reduces ambiguity or unsafe behavior:

7. Regression / Eval Case
Write one reusable test case with:
- Setup/precondition
- User prompt or failure condition
- Expected agent behavior
- Forbidden behavior
- Pass criteria

8. Remaining Human Review
List any configuration, security, network, or organizational decision that still requires a human owner.

VALIDATED REGRESSION BASELINES
- Page `115220454` is the source template and contains six retrievable image attachments.
- Page `233407069` was used as an approved writer/image target during validation.
- Existing source-hosted Procedure image references survived storage-aware Procedure updates.
- A user-designated local PNG upload to the target page succeeded, then resolved through target page image retrieval.
- The Version panel and nested `change-history` macro remained intact during the validated writes.

IMPORTANT
Do not implement a documentation, skill, or configuration change merely because you proposed it. Present the proposed change for human review first.
```

## Maintainer use

After the issue is resolved and the proposed improvement is approved:

1. update `INSTALL.md` or the appropriate reference file;
2. add the regression case to `eval-cases.md`;
3. increment the package version/changelog;
4. rerun the normal read-only installation and page `115220454` tests;
5. have a human review the updated package before distribution.
