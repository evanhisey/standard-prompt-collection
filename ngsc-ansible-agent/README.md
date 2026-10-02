# NGSC Ansible Automation VS Code Agent

This package provides a VS Code custom agent plus an Agent Skill for Ansible/AAP automation and GitLab CI/CD across the NGSC and AWS WorkSpaces connection models.

Supported host profiles:

- Legacy NGSC Windows: WinRM + Kerberos
- NGSC Linux: SSH
- AWS WorkSpaces Windows: SSH + PowerShell
- AWS WorkSpaces Linux: SSH

For Windows hosts in this environment, `ansible_shell_type: powershell` is authoritative: the host is SSH-only. WinRM is not a supported fallback for a PowerShell-tagged Windows host.

The CI/CD capability adds secure GitLab pipeline generation for Ansible validation, runtime smoke testing, software/RPM builds, package verification/publication, deployment, post-deployment smoke testing, promotion, and rollback.

## Package contents

```text
.github/
├── agents/
│   └── ngsc-ansible-automation.agent.md
└── skills/
    └── gitlab-ansible-cicd/
        └── SKILL.md
README.md
```

The agent remains the primary persona. The `gitlab-ansible-cicd` skill holds the detailed pipeline workflow so CI/CD rules do not turn the base agent into one large prompt. VS Code can load the skill automatically when a request involves GitLab pipelines, builds, smoke tests, package publication, or deployment.

The skill explicitly allows both automatic and manual invocation:

```yaml
user-invocable: true
disable-model-invocation: false
```

This means Copilot may load it automatically when its description matches the task, and a developer may force it with `/gitlab-ansible-cicd`. The parent agent also contains a mandatory routing matrix so CI/CD-related requests do not depend only on semantic skill discovery.

## Skill routing behavior

The base agent loads `gitlab-ansible-cicd` when a request involves any of these boundaries:

- GitLab CI configuration or troubleshooting;
- runners, CI images, runner tags, or runner/executor policy;
- lint, syntax, policy, or other CI validation;
- runtime or post-deployment smoke tests;
- RPM/software/container builds consumed by deployment;
- package verification, checksums/signatures, publication, or package repositories;
- build -> test -> publish -> deploy sequencing;
- deployment jobs, promotion, release, rollback, or protected-environment gates.

Examples:

| Request | Skill behavior |
|---|---|
| Create or modify an Ansible role | Normally do not load |
| Add GitLab CI validation for an Ansible role | Load |
| Modify a patching playbook | Normally do not load |
| Add a pre-production smoke test for the playbook | Load |
| Build an RPM that a deployment playbook installs | Load |
| Troubleshoot Windows SSH connectivity | Normally do not load |
| Troubleshoot Windows SSH in a GitLab smoke-test job | Load |

For manual testing or when you want to force the CI/CD workflow, invoke the skill directly in chat:

```text
/gitlab-ansible-cicd add an RPM build, verification, and pre-production smoke-test pipeline for this repository
```

The skill runs inline with the selected NGSC Ansible agent so it can use the same repository context and the agent's Windows/Linux transport rules.

## Recommended installation: project-specific

For an NGSC/WorkSpaces GitLab repository, install the complete package in the repository:

```text
<repo-root>/.github/agents/ngsc-ansible-automation.agent.md
<repo-root>/.github/skills/gitlab-ansible-cicd/SKILL.md
```

Commit both paths normally. VS Code uses `.github/agents` and `.github/skills` as workspace customization locations even when the source repository is hosted in **GitLab**; the repository does not need to be hosted on GitHub.

Project installation is the safest default because host transports, AAP inventory behavior, GitLab branches, runner tags, package repositories, smoke-test targets, and protected-environment requirements can vary by repository.

Open the repository in VS Code, open Chat, and select **NGSC Ansible Automation Engineer**. Use **Chat: Open Customizations** to confirm that both the agent and skill are discovered.

## Optional installation: user-wide agent

To make the Ansible agent available across repositories, copy the `.agent.md` file directly into the VS Code user `prompts` folder.

On Linux:

```text
~/.config/Code/User/prompts/ngsc-ansible-automation.agent.md
```

```bash
mkdir -p "$HOME/.config/Code/User/prompts"
cp "/path/to/standard-prompt-collection/ngsc-ansible-agent/.github/agents/ngsc-ansible-automation.agent.md" "$HOME/.config/Code/User/prompts/ngsc-ansible-automation.agent.md"
```

On Windows:

```text
%APPDATA%\Code\User\prompts\ngsc-ansible-automation.agent.md
```

```powershell
$sourceAgent = "C:\Path\To\standard-prompt-collection\ngsc-ansible-agent\.github\agents\ngsc-ansible-automation.agent.md"
$targetAgent = Join-Path $env:APPDATA "Code\User\prompts\ngsc-ansible-automation.agent.md"
New-Item -ItemType Directory -Force -Path (Split-Path $targetAgent) | Out-Null
Copy-Item -Force -Path $sourceAgent -Destination $targetAgent
```

Do not create a nested `agents/` directory inside the VS Code user `prompts` folder. User/global VS Code customization discovers `.agent.md`, `.prompt.md`, and `.instructions.md` files from that folder.

For CI/CD behavior, install the bundled skill in each repository or workspace where it is approved, for example:

```text
<repo-root>/.github/skills/gitlab-ansible-cicd/SKILL.md
```

or:

```text
<repo-root>/.agents/skills/gitlab-ansible-cicd/SKILL.md
```

Do not copy bundled skills into the VS Code user `prompts` folder. Skills belong in the target repository or workspace customization path.

A useful mixed deployment is:

- install the **agent user-wide** if you want the NGSC/WorkSpaces Ansible role everywhere;
- keep the **CI/CD skill project-specific** when GitLab runners, branches, environments, package services, or release policy differ between repositories.

## What the CI/CD capability does

The agent first classifies the repository's actual need instead of generating a large pipeline by default:

- validation only;
- Ansible runtime smoke testing;
- build/package verification;
- build + publish + deploy;
- deploy an existing immutable package;
- release/promotion/rollback.

For Ansible repositories it can generate or maintain GitLab jobs for:

- `yamllint`;
- `ansible-lint`;
- `ansible-playbook --syntax-check`;
- repository-owned shell/PowerShell validation;
- non-production connection and runtime smoke tests;
- RPM/software build and verification;
- package publication;
- deployment playbooks;
- post-deployment health checks.

The agent deliberately distinguishes **static validation** from **runtime smoke tests**. A lint or syntax job is not reported as a smoke test.

## Recommended Ansible CI container

For Ansible/AAP pipeline jobs, the recommended baseline container is:

```text
registry.management.acf.gov/oci/operations/op-tools:2.0
```

Use `2.0` as the minimum recommended baseline, or a newer ACF-approved compatible `op-tools` image when the repository/platform baseline has validated it. This keeps CI behavior aligned with the ACF AAP deployment environment. Preserve an approved newer repository pin instead of downgrading it.

A specialized build job may use another approved image when its toolchain requires it (for example RPM build tooling), but Ansible validation, smoke-test, and deployment jobs should return to the approved `op-tools` image unless the repository documents another requirement. Do not replace it with a generic public Ansible/Python image simply for convenience.

## GitLab routing baseline

Unless repository evidence defines approved alternatives, the supplied ACF GitLab baseline uses:

| Purpose | Default |
|---|---|
| YAML compatibility | GitLab 17 |
| Documented modernization target | GitLab 18.11 or later |
| Production branch | `main` |
| Pre-production branch | `pre-prod` |
| Production runner tag | `live-shared-services` |
| Pre-production runner tag | `live-shared-services-pre-production` |
| Production environment | `production` |
| Pre-production environment | `pre-production` |

These are defaults, not values the agent should force over local repository evidence.

Merge requests are validation-only by default and must not use production runners or production secrets. Production deployment/promotion relies on protected branches, protected environments, protected runners, and approvals in addition to `.gitlab-ci.yml` rules.

## RPM and software-build workflows

When a deployment playbook depends on software that must be built first, the agent models the build as a producer/consumer workflow rather than hiding it inside the deploy job.

Preferred shape:

```text
validate
  -> build
  -> package_verify
  -> smoke
  -> publish
  -> deploy
  -> post_deploy_smoke
```

For RPMs, the agent can account for `.spec` validation, reproducible/isolated build tooling, `rpmlint` where available, RPM metadata checks, checksum/signature policy, install/upgrade smoke testing, package publication, and deployment of the exact immutable package version.

Generated RPMs should normally **not** be committed to the source Git repository. Publish the validated package to the repository/service intended to distribute packages, then make the deployment playbook consume that exact version.

GitLab's Generic Package Registry can store an RPM file as a generic binary, but it is not a native YUM/DNF RPM repository. If hosts install packages through repository metadata, use the approved internal RPM/YUM repository or the project's established repodata workflow.

## Fact-gathering efficiency

The agent treats Ansible fact gathering as opt-in based on actual need rather than an automatic default:

- use `gather_facts: false` when a play or smoke test does not consume host facts;
- when facts are required, request only the smallest supported subset;
- for POSIX hosts, a distribution-only pattern is:

```yaml
gather_facts: true
gather_subset:
  - '!all'
  - '!min'
  - distribution
```

The subset can be described compactly as `!all,!min,distribution`, but generated playbook YAML uses the explicit list form because `gather_subset` is a list of subset selectors. Full fact gathering should not be used merely to determine the Windows connection transport; that comes from the AAP inventory/connection contract. CI and smoke-test plays follow the same minimal-facts rule.

## Smoke-test model

The agent uses three separate validation layers:

1. **Static validation** - lint, syntax, schemas, policy checks.
2. **Build/package verification** - artifact metadata, payload, checksum/signature, expected output.
3. **Runtime smoke testing** - minimal execution against an approved disposable or pre-production target.

Connection smoke tests respect the host profile:

- Linux SSH: `ansible.builtin.ping` plus minimal read-only checks as required.
- Legacy NGSC Windows Kerberos/WinRM: `ansible.windows.win_ping` through the Kerberos-capable path.
- WorkSpaces Windows SSH/PowerShell: `ansible.windows.win_ping` plus a minimal PowerShell/module assertion when required.

Post-deployment smoke tests should verify the actual deployed version/service/configuration without exposing secrets.

## Windows connection normalization

The agent includes a reusable normalization pattern for roles/playbooks that must operate across Windows SSH/PowerShell and WinRM hosts.

In this environment, `ansible_shell_type: powershell` on a Windows host is the authoritative SSH selector. The agent must normalize the effective connection before the first remote task:

- PowerShell-tagged Windows path: `ansible_connection: ssh`, `ansible_shell_type: powershell`, normally port 22. SSH is the only supported remote transport for these hosts.
- The agent must not probe, generate, or fall back to WinRM for a PowerShell-tagged Windows host, even if stale or conflicting WinRM variables are present.
- Legacy NGSC WinRM/Kerberos is a separate profile only for Windows hosts without `ansible_shell_type: powershell` and identified by repository/AAP context as legacy NGSC Windows.
- Legacy NGSC Windows uses the established Kerberos-over-HTTPS model when local evidence does not define a more specific approved value.
- Domain credentials are normalized in a separate `no_log: true` task and must come from the approved AAP/secret source.
- Debug output reports only non-secret connection-selection information.

For this environment, the agent **does** treat `ansible_shell_type: powershell` on Windows as definitive evidence of the SSH transport contract. Static host/group connection settings should remain in inventory/group variables where possible; task-level `set_fact` is reserved for roles that genuinely need runtime normalization across SSH-only PowerShell hosts and separate legacy WinRM/Kerberos hosts.

## AAP and Windows SSH baseline

AAP 2.6 / `ansible-core` 2.16 is treated as a valid project baseline. The agent intentionally permits the repository-tested Windows-over-SSH pattern on 2.16 when the required `ansible.windows` / `community.windows` content and SSH dependencies are present.

The agent distinguishes project-tested capability on 2.16 from the later upstream/vendor official Windows OpenSSH support boundary. It does not rewrite a working 2.16 WorkSpaces SSH implementation merely because official support arrived later.

## Secrets and runner administration

The agent does not ask users to paste secrets and does not generate real secret values.

For GitLab CI/CD it also separates repository-owned configuration from runner/admin controls. In particular:

- private CI-image pull authentication belongs to runner/executor configuration, not `before_script`;
- merge-request pipelines do not receive production secrets;
- structured secrets should use appropriately scoped protected/File variables when that matches the repository contract;
- secret-bearing files are never artifacts or caches;
- protected branches/environments, runner protections, approvals, and package-repository permissions must be configured in GitLab/infrastructure and cannot be guaranteed by YAML alone.

## Report/document behavior

The agent does not create a new Markdown report for every request. Plans, findings, validation results, and implementation summaries are returned in chat by default.

If the repository already has a canonical engineering report or CI document, the agent updates it in place. If persistent general reporting is explicitly required and no convention exists, the fallback is:

```text
AGENT_REPORT.md
```

For substantial CI/CD documentation, a single stable path such as `docs/CI_CD.md` may be created once when justified and then maintained in place. Timestamped/query-specific report files are not generated unless explicitly requested.

## Human review and execution safety

Generated code is not production approval. Review changes before merge and before executing against managed hosts.

The agent may run safe local/static validation when available. It must not use live NGSC or WorkSpaces inventory as a validation shortcut and must not publish packages, trigger deployments, or run destructive jobs merely to prove generated pipeline code works.

## References

- VS Code custom agents: https://code.visualstudio.com/docs/agent-customization/custom-agents
- VS Code Agent Skills: https://code.visualstudio.com/docs/agent-customization/agent-skills
- GitLab CI/CD YAML: https://docs.gitlab.com/ci/yaml/
- GitLab job controls: https://docs.gitlab.com/ci/jobs/job_control/
- GitLab Generic Package Registry: https://docs.gitlab.com/user/packages/generic_packages/
- GitLab package registry formats: https://docs.gitlab.com/administration/packages/
- Ansible Windows SSH setup: https://docs.ansible.com/projects/ansible-core/2.16/os_guide/windows_setup.html#windows-ssh-setup
- Ansible Kerberos authentication: https://docs.ansible.com/projects/ansible/latest/os_guide/windows_winrm_kerberos.html

## Technical documentation behavior

The agent treats repository documentation as technical specification rather than educational/project-management material.

- Plans, assumptions, open questions, and implementation summaries remain in chat unless persistence is explicitly required.
- New Markdown files are created only when the information must persist, no canonical document can hold it, and the new file has a distinct technical responsibility.
- Prefer existing `README.md`, canonical `DESIGN-PLAN.md`, and canonical `IMPLEMENTATION-SUMMARY.md`. Use a dedicated `docs/CI_CD.md` only when CI/CD requires a separate specification or the repository already uses that path.
- Do not generate `00-START-HERE.md`, information-gathering checklists, reference-location/navigation files, deliverables/status documents, audience guides, or per-query reports unless explicitly requested or already authoritative.
- Specifications should state exact paths, roles/jobs, variables/defaults, dependencies, connection contracts, pipeline/artifact contracts, hard blockers, validation commands, and objectively testable acceptance criteria.
- Unknown implementation-critical values are not guessed. Use repository/ACF defaults when established; otherwise record a concise `BLOCKED:` constraint in the canonical specification.

This policy is intended to prevent documentation sprawl and to keep smaller/faster models from converting planning scaffolding into repository artifacts.
