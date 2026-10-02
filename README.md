# Standard Prompt Collection

This repository is a source collection for reusable VS Code Copilot agents and skills used by NGSC automation, CI/CD, patch-management, and Confluence runbook workflows.

The directories in this repository are package sources. Do not treat this repository's `.github` directory as the installation target. Copy the package you need into the target workspace or user-level customization location required by your VS Code/Copilot build.

## Directory Overview

| Directory | Type | Purpose |
|---|---|---|
| `gitlab-cicd-pipeline-expert/` | Skill | ACF GitLab CI/CD pipeline design, review, troubleshooting, secure branch routing, runner/private image policy, validation, deployment gates, promotion, and rollback. |
| `cicd-pipeline/` | Skill | Platform-neutral GitHub Actions and GitLab CI/CD pipeline design, review, troubleshooting, secrets, artifacts, Podman, SELinux, FIPS-safe scripting, and validation. |
| `acf-patch-management-compatibility/` | Skill | Ansible compatibility guidance for `acf.patch_management`, non-repository Linux applications, canonical patch facts, optional integration boundaries, and CI validation. |
| `ngsc-ansible-agent/` | Agent and skill package | NGSC/WorkSpaces Ansible automation agent with a bundled GitLab Ansible CI/CD skill. |
| `acf-jira-mcp-agent/` | Agent, skill, and MCP package | ACF Jira MCP agent with a bundled Jira workflow skill, read-only issue retrieval, approved writer-test operations, and MCP setup material. |
| `acf-confluence-runbook-standardizer-v0.2.0/` | Skill and MCP package | ACF Confluence runbook standardizer skill with Credal, Confluence MCP, writer-test, and image-handling setup material. |
| `old-style-prompt-files/` | Archive | Legacy prompt files retained for reference after conversion to newer skill or agent packaging. |

## Basic Skill Installation

For simple skill packages, copy the entire top-level skill directory into the target workspace skill directory and keep the folder name unchanged.

Simple skill packages in this repository:

- `gitlab-cicd-pipeline-expert/`
- `cicd-pipeline/`
- `acf-patch-management-compatibility/`

Recommended workspace destination:

```text
<target-repo>/.agents/skills/<skill-name>/
```

Some VS Code/Copilot builds also support repository customization locations such as:

```text
<target-repo>/.github/skills/<skill-name>/
```

Use the location approved for the target repository or VS Code build. The source copy in this repository remains a top-level package directory either way.

### Windows PowerShell

```powershell
$source = "C:\Users\evan.hisey\Git_Repo\standard-prompt-collection\gitlab-cicd-pipeline-expert"
$target = "C:\path\to\target-repo\.agents\skills\gitlab-cicd-pipeline-expert"
New-Item -ItemType Directory -Force -Path (Split-Path $target) | Out-Null
Copy-Item -Recurse -Force -Path $source -Destination $target
```

### Linux/macOS Bash

```bash
source="$HOME/Git_Repo/standard-prompt-collection/gitlab-cicd-pipeline-expert"
target="/path/to/target-repo/.agents/skills/gitlab-cicd-pipeline-expert"
mkdir -p "$(dirname "$target")"
cp -R "$source" "$target"
```

Replace `gitlab-cicd-pipeline-expert` with the skill directory you want to install.

## Package-Specific Instructions

Some packages need more than a simple skill copy.

- For `acf-confluence-runbook-standardizer-v0.2.0/`, follow `acf-confluence-runbook-standardizer-v0.2.0/INSTALL.md`. That package includes Credal model setup, Confluence MCP profiles, read-only validation, writer-test guidance, and image-handling profiles.
- For `acf-jira-mcp-agent/`, follow `acf-jira-mcp-agent/INSTALL.md`. That package includes a custom Jira agent, a bundled Jira workflow skill, Credal model setup, Jira MCP profiles, read-only validation, and controlled writer-test guidance.
- For `ngsc-ansible-agent/`, follow `ngsc-ansible-agent/README.md`. That package includes both a VS Code custom agent and a related skill, so install both package parts together unless you have a reviewed reason to split them.
- For each simple skill package, read its local `README.md` before copying it into another repository.

## Validation After Install

After copying a skill or agent package:

1. Open the target repository in VS Code.
2. Open Copilot Chat.
3. Use the VS Code customization view or command available in your build to confirm the skill or agent is discovered.
4. Start a small request that matches the package description.
5. Confirm the agent loads the expected skill before using it for production changes.

Do not paste credentials, tokens, private keys, or live secrets into prompts, package files, screenshots, or troubleshooting output.
