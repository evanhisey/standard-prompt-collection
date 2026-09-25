---
name: gitlab-ansible-cicd
description: >
  Design, review, create, modify, or troubleshoot GitLab CI/CD for Ansible/AAP
  repositories. Use this skill whenever work involves .gitlab-ci.yml, GitLab
  runners, Ansible CI validation, ansible-lint, yamllint, syntax checks, runtime
  smoke tests, RPM or software builds, package verification or publication,
  immutable build artifacts, deployment pipelines, post-deployment tests,
  promotion, release, or rollback workflows.
user-invocable: true
disable-model-invocation: false
---

# GitLab Ansible CI/CD Skill

Load this skill whenever the task crosses from Ansible implementation into CI/CD, build/package, runtime testing, artifact publication, or deployment-pipeline behavior. It may also be invoked explicitly as `/gitlab-ansible-cicd`.

Do not load this skill for ordinary Ansible code changes that have no CI/CD, package-build, smoke-test, publication, or pipeline impact.

## 1. Start from repository evidence

Inspect before designing:

- `.gitlab-ci.yml` and included CI files;
- `README*`, `docs/`, `AGENTS.md`, project instructions, and existing CI documentation;
- Ansible playbooks, roles, `requirements.yml`, Execution Environment definitions, and AAP launch/deploy scripts;
- `ci/`, `scripts/`, `tests/`, Molecule configuration, and validation commands;
- package/build files such as RPM `.spec` files, Makefiles, source archives, lock/checksum files, container build files, or language manifests;
- branch/environment conventions and any locally documented GitLab/runner restrictions.

When available, the following sibling AAP project files are useful implementation examples for this GitLab environment, not universal requirements:

- `../aap-deployment/aap-prompt-w-readme.template`
- `../aap-deployment/PHASE_1_DIAGNOSTIC_FINDINGS.md`
- `../aap-deployment/PHASE_2_3_IMPLEMENTATION.md`
- `../aap-deployment/.gitlab-ci.yml`

Generalize their GitLab patterns to the target repository. Do not copy AAP-specific deployment behavior into unrelated projects.

## 2. Build a pipeline design contract

Collect or infer:

- project name and type;
- pipeline goal: validate, build, test, package, publish, deploy, promote, rollback, release, or mixed;
- GitLab compatibility baseline;
- runner image, executor, and runner tags;
- production/pre-production branches and environments;
- merge-request policy;
- deployment targets;
- required CI/CD variables/File variables by name and scope only;
- validation and smoke-test commands;
- artifact/package retention policy;
- package repository/publishing destination;
- manual gates and protected-environment expectations;
- unresolved runner, permission, credential, package, image, host, or approval blockers.

For a new pipeline, produce the design plan before files unless the user explicitly asks for immediate implementation.

Use this plan structure:

1. Pipeline draft status and blockers.
2. Repository assumptions.
3. Routing/security model.
4. Proposed file tree.
5. Validation and smoke-test plan.
6. Open questions that actually block safe generation.

## 3. GitLab compatibility and security baseline

Unless repository evidence says otherwise:

- Generate GitLab 17-compatible YAML.
- Treat GitLab 18.11 or later as the documented modernization target from the supplied environment baseline when relevant.
- Do not use newer features for security-sensitive routing while GitLab 17 compatibility is required.
- Use `workflow: rules` to control allowed pipeline creation.
- Use `rules` rather than deprecated `only`/`except`.
- Fail closed. When a rule set is intended as an allowlist, end with `when: never`.
- Keep branch-to-environment-to-runner selection in trusted repository logic, preferably `workflow: rules: variables` when branch routing is required.
- Do not expose production environment, production runner selection, or production deployment decisions as user-editable manual/project/group variables.
- Use variable-expanded runner tags only when the installed runner/GitLab configuration supports them and the value comes from trusted workflow rules.
- Merge requests use a dedicated validation or pre-production runner and do not receive production secrets.
- Production deployment requires the protected production branch plus protected runner/environment/approval controls outside the YAML.
- Repository CI YAML is not an access-control boundary by itself.
- Keep deploy, rollback, destructive reset, package publication, and production promotion manual unless an approved policy explicitly automates them.
- Do not use Git tags as the production promotion mechanism unless approved for the repository.

Environment defaults from the supplied ACF GitLab baseline, only when not contradicted by local evidence:

- production branch: `main`
- pre-production branch: `pre-prod`
- production runner tag: `live-shared-services`
- pre-production runner tag: `live-shared-services-pre-production`
- environments: `production`, `pre-production`

## 4. Runner and private-image policy

### Recommended Ansible CI image

- For Ansible/AAP validation, smoke-test, deployment, and other Ansible-centric GitLab jobs, recommend `registry.management.acf.gov/oci/operations/op-tools:2.0` as the minimum baseline image.
- A newer compatible `op-tools` image may be used when the repository or ACF platform baseline has validated it. Prefer the newest approved compatible image rather than arbitrarily changing image families.
- This recommendation exists to preserve compatibility with the ACF AAP deployment tooling and runner environment.
- Do not replace the `op-tools` image with a generic public Ansible/Python image merely for convenience when the job is expected to mirror the ACF AAP execution environment.
- If an Ansible job requires tooling not present in the approved `op-tools` image, first determine whether the capability belongs in the image, a dedicated build job/image, or the repository. Do not silently install large toolchains at job runtime.
- Build/package jobs may use a different approved image when required by the build toolchain (for example RPM build tooling), but downstream Ansible validation/deployment jobs should return to the approved `op-tools` image unless local evidence requires otherwise.
- When a repository already pins an approved newer `op-tools` image, preserve that pin unless there is a documented compatibility reason to change it.

- Private image pull authentication is runner-admin configuration because the image must be pulled before job scripts execute.
- Do not manufacture registry authentication in `before_script`.
- Do not require a project `DOCKER_AUTH_CONFIG` only to pull the CI image.
- Kubernetes executor: prefer namespace-scoped Docker registry Secrets wired through runner `image_pull_secrets`.
- Docker executor: prefer a mode-0600 Docker `config.json` for the runner account.
- Runner `config.toml` environment configuration is a fallback only when executor-native auth is unavailable and approved.
- Pull credentials are read-only and narrowly scoped, with no production deployment privilege.
- Never print, artifact, commit, or decode registry credentials.
- Do not set `image:pull_policy` when the runner disallows pipeline-defined policies.
- When mutable image tags are unavoidable, document cache/refresh risk and assign runner operations as the owner of refresh/digest policy.
- Use `FF_KUBERNETES_HONOR_ENTRYPOINT=true` for Ansible Builder-derived images only when local runner evidence requires it.
- If the runtime UID lacks NSS/passwd identity and SSH or other tooling requires it, fail closed and report a runner-image problem.
- Treat Kubernetes admission warnings, pod RBAC gaps, unapproved helper/init registries, and runner security-context issues as runner-operation findings unless the repository owns them.
- Use `python3` for runner-side Python commands unless local evidence establishes another interpreter.
- Do not install expected CI tooling dynamically when the approved runner image is supposed to provide it. Validate versions and report missing tooling.

## 5. Secrets and variable policy

- Never hardcode credentials, private keys, tokens, live account IDs, customer hostnames, or secret values.
- Prefer environment-scoped File variables for structured/sensitive inputs that tools consume as files.
- For every required GitLab variable, document: name, type, scope, protected status, masking expectation, and owner.
- Validate required File variables as existing non-empty files without printing contents.
- When tools need a stable sensitive path, copy to a job-owned temporary file under restrictive `umask`, then remove it with a trap or `after_script`.
- Never persist sensitive files in artifacts, caches, test reports, logs, or generated documentation.
- Parse secret JSON at the native parser/tool boundary; do not round-trip it through unsafe shell serialization.
- Preserve multiline PEM/key bytes exactly. Do not normalize, fold, repair, or echo secret material.
- Use SHA-256 or stronger for integrity checks.
- Commit package coordinates, versions, and checksums when needed; do not commit vendor/generated binaries unless repository policy explicitly requires it.
- Use `CI_JOB_TOKEN` for automated package access when the target GitLab endpoint supports it and project access controls permit it.
- Keep package publication separate from deployment.

## 6. Select the smallest correct pipeline

### Validation-only Ansible repository

Typical stages:

`validate`

Applicable jobs:

- `yamllint`
- `ansible-lint`
- `ansible-playbook --syntax-check`
- shell/PowerShell syntax/static analysis for repository-owned scripts
- policy/schema checks

Do not call these runtime smoke tests.

### Ansible repository with runtime smoke tests

Typical stages:

`validate -> smoke`

Static validation runs first. The smoke stage uses a dedicated disposable/validation/pre-production target and performs only the minimal behavior required to prove the change.

### Build/package plus Ansible deployment

Typical stages:

`validate -> build -> package_verify -> smoke -> publish -> deploy -> post_deploy_smoke`

Use `needs` where appropriate without obscuring the security or producer-consumer boundary.

Build once. Downstream stages consume the same immutable artifact/version.

### Deployment-only repository

Typical stages:

`validate -> deploy -> post_deploy_smoke`

Require an already published immutable artifact coordinate. Do not silently add a hidden rebuild.

## 7. Ansible validation requirements

For Ansible repositories use, when supported by the repository:

- Fully qualified collection names.
- Explicit `become` behavior.
- YAML booleans as booleans.
- `yamllint`.
- `ansible-lint`.
- `ansible-playbook --syntax-check`.
- Existing repository unit/integration tests.

Scope linters to repository-owned files. Exclude downloaded Galaxy content, generated dependency trees, caches, build outputs, and tool-managed directories.

Avoid live production inventory for CI validation.

## 8. Smoke-test design

A smoke test is a minimal runtime proof, not a lint job.

### Fact-gathering efficiency in CI and smoke tests

- Set `gather_facts: false` for validation, connection, and smoke-test plays that do not consume discovered host facts.
- If a smoke test requires facts, collect only the smallest supported subset. For POSIX targets, a distribution-only example is:

```yaml
gather_facts: true
gather_subset:
  - '!all'
  - '!min'
  - distribution
```

- Do not enable full fact gathering simply to determine Windows SSH versus WinRM. Connection transport comes from the AAP inventory/repository contract; a Windows host with `ansible_shell_type: powershell` is SSH-only in this environment.
- If facts become necessary only after an initial connection check, prefer an explicit narrow `ansible.builtin.setup` task rather than gathering all facts at play start.
- Keep target-platform differences in mind; only request subset names supported by the target fact module and Execution Environment.

### Smoke-test properties

- non-production by default;
- least privilege;
- deterministic and time-bounded;
- verifies one or more observable outcomes;
- fails when the expected outcome is absent;
- produces sanitized diagnostics;
- cleans temporary resources when safe;
- uses repository-owned scripts/playbooks for complex logic rather than long inline YAML;
- never receives production credentials in merge requests.

### Ansible connection smoke tests

Use the environment's actual connection profile:

- Linux SSH: `ansible.builtin.ping`, then a minimal read-only/non-destructive module check if needed.
- Legacy NGSC Windows Kerberos/WinRM: `ansible.windows.win_ping` through the established Kerberos-capable Execution Environment.
- Windows WorkSpaces SSH/PowerShell: `ansible.windows.win_ping`, plus a minimal PowerShell/module assertion when needed.
- If a Windows target has `ansible_shell_type: powershell`, treat it as SSH-only for smoke testing. Do not generate or attempt a WinRM fallback/test path for that host.

Do not change transport merely to make CI easier.

### Playbook/role smoke tests

Prefer a dedicated smoke playbook or script, for example:

- `tests/smoke/smoke.yml`
- `ci/smoke/run.sh`
- `ci/smoke/run.ps1`

Use Molecule when the repository already uses it or its target model is accurate. Do not force Linux container-based Molecule onto Windows, domain, systemd-heavy, kernel-dependent, or host-integration tests that require a real VM/host.

### Post-deployment smoke tests

Verify the deployed outcome, such as:

- exact package/application version;
- service enabled/running state;
- expected file/config state without dumping secrets;
- expected local health endpoint/port where approved;
- idempotent second-run behavior when practical;
- Windows service/PowerShell health checks appropriate to the target.

## 9. RPM build/package workflow

If deployment depends on a new RPM, inspect `.spec`, sources, macros, target RHEL release, signing policy, and repository publication model first.

Prefer an isolated/reproducible build toolchain already approved by the project; use `mock` as a recommendation when appropriate rather than silently imposing it.

A robust flow is:

1. Validate spec/sources/version.
2. Build SRPM/RPM as required.
3. Run `rpmlint` when available/approved.
4. Verify metadata with `rpm -qp` and inspect payload as needed.
5. Verify checksum/signature contract.
6. Smoke-test install/upgrade/uninstall in an environment capable of exercising the package.
7. Publish the exact validated package to the approved package repository/storage.
8. Record immutable package coordinate/version/checksum for downstream deployment.
9. Run the Ansible deployment playbook using that exact package version.
10. Run post-deployment smoke tests.

Do not rebuild the RPM during deployment.

Do not commit generated RPMs to the Git source repository unless the project intentionally versions binaries in Git.

GitLab's Generic Package Registry can store an `.rpm` as a generic binary, but it is not a native RPM/YUM repository. If the playbook installs through `dnf`/`yum` repository metadata, use the approved RPM/YUM repository service or the repository's established repodata workflow.

## 10. Producer/consumer contracts

Treat every generated artifact as a contract between stages or languages.

Examples:

- environment file produced by Python and sourced by shell;
- JSON produced by a build script and parsed by Ansible;
- RPM metadata passed into a deployment playbook;
- checksum file consumed by download/install logic.

Validate the exact serialization/format before the consumer uses it. `sh -n` validates shell syntax but does not prove that a generated environment file is semantically safe to source.

Avoid shell chaining that hides the failing command. Prefer readable block scalars and strict shell mode such as `set -eu` where appropriate.

## 11. Artifact and package rules

- Artifact only non-secret outputs needed by downstream jobs or human review.
- Set retention intentionally.
- Never use broad caches for paths that can contain secrets.
- Keep build artifacts distinct from package publication.
- If a downstream job can fetch a package from the approved package repository, prefer that immutable package coordinate over passing a long-lived CI artifact.
- Never describe GitLab Generic Packages as a YUM repository.

## 12. Deploy, promotion, and rollback

- Deployment jobs are manual by default.
- Production deployment is blocking/manual unless approved automation policy says otherwise.
- Use protected environments and GitLab approvals where available and configured.
- Use `manual_confirmation` for sensitive manual actions when the deployed GitLab version supports it.
- Destructive reset jobs are non-production only by default and require exact repository-controlled guards, not user-provided variables.
- Add a rollback job only when an approved rollback command/process exists. Otherwise document the missing rollback contract and fail closed rather than inventing one.
- Package publication and production promotion should be independently reviewable gates.

## 13. Troubleshooting classification

When a job fails, identify the failing command and exit status before blaming warnings.

Classify findings as:

1. fatal CI/repository finding;
2. non-fatal dependency/deprecation warning;
3. external runner/platform policy finding.

Point to the owning layer: repository, runner administration, GitLab project settings, registry/package service, AAP, or target infrastructure.

## 14. Documentation behavior

Update an existing README/CI document where possible. Do not create a new report Markdown file per request.

If a dedicated CI/CD document is needed and no stable document exists, use one canonical path such as `docs/CI_CD.md` and update it in place.

Document:

- GitLab compatibility baseline;
- branch/environment/runner mapping;
- protected branch/environment/approval prerequisites;
- runner image/executor prerequisites;
- required CI variables by metadata only, never values;
- validation and smoke-test commands;
- artifact/package retention and publication model;
- deployment and rollback gates;
- unresolved operational blockers and their owner.

## 15. Implementation output

After implementation, report:

1. Implementation status and unresolved blockers.
2. Pipeline architecture and stage flow.
3. Files created/modified.
4. Validation/smoke tests actually run and results.
5. Variables/runner/project settings that an administrator must configure.
6. Human-review gates before merge, package publication, deployment, or production promotion.

Never claim a command, smoke test, package publication, or deployment ran if it did not.
