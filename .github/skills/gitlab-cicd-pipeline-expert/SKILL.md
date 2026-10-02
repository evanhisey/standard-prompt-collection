---
name: gitlab-cicd-pipeline-expert
description: Design, generate, review, or troubleshoot GitLab CI/CD pipelines for ACF GitLab repositories, including GitLab 17-compatible branch routing, protected production controls, secure runner/private image handling, CI variables, validation, deployment gates, promotion, rollback, and GitLab 18 modernization planning.
---

# GitLab CI/CD Pipeline Expert

Use this skill when the user asks for a GitLab CI/CD pipeline, `.gitlab-ci.yml`, CI scripts, deployment gates, package publication, promotion, rollback, GitLab runner troubleshooting, private runner image handling, or secure CI/CD design for the ACF GitLab environment.

This skill is a better fit than a custom agent for normal pipeline work because it provides reusable domain rules while allowing the main coding agent to inspect and edit the target repository directly. Recommend a custom agent only when the user wants an isolated CI/CD reviewer persona with restricted tools or a long-running independent review separate from implementation.

## First response for new pipeline requests

For a new pipeline or major redesign, provide a design plan before editing unless the user explicitly asks to implement immediately.

The design plan must include:

1. Pipeline draft status and unresolved blockers.
2. Repository assumptions.
3. Routing and security model.
4. Proposed file tree.
5. Validation and smoke-test plan.
6. Open questions that block safe generation.

## Start from local evidence

Inspect the target repository before generating final files:

- existing `.gitlab-ci.yml` and included CI files;
- `README*`, `docs/`, project instructions, and existing CI documentation;
- application manifests, Ansible/Terraform files, package/build files, scripts, and tests;
- `ci/`, `scripts/`, `tests/`, Molecule configuration, lock/checksum files, and validation commands;
- branch/environment conventions and runner restrictions documented in the repo.

When present, these sibling AAP deployment files are useful examples for the local GitLab environment, not universal requirements:

- `../aap-deployment/aap-prompt-w-readme.template`
- `../aap-deployment/PHASE_1_DIAGNOSTIC_FINDINGS.md`
- `../aap-deployment/PHASE_2_3_IMPLEMENTATION.md`
- `../aap-deployment/.gitlab-ci.yml`

Generalize their GitLab compatibility, branch routing, private image, secret handling, and manual gate patterns. Do not copy AAP-specific deployment behavior into unrelated projects.

## Design contract to collect or infer

Collect or infer the following before final generation:

- project name and type;
- pipeline goal: validation, build, test, package, publish, deploy, promote, rollback, release, or mixed;
- GitLab compatibility baseline, defaulting to GitLab 17 unless evidence says otherwise;
- GitLab 18.11 or later modernization target when relevant;
- runner image, executor, and runner tags;
- production and pre-production branches and environments;
- merge-request policy;
- deployment targets;
- required CI/CD variables and File variables by name, type, scope, protected status, masking expectation, and owner;
- validation and smoke-test commands;
- artifact/package retention policy;
- manual gates and protected-environment expectations;
- unresolved runner, permission, credential, package, image, host, or approval blockers.

Default ACF GitLab branch/routing assumptions when not contradicted by local evidence:

- production branch: `main`
- pre-production branch: `pre-prod`
- production runner tag: `live-shared-services`
- pre-production runner tag: `live-shared-services-pre-production`
- environments: `production`, `pre-production`

## GitLab compatibility and routing rules

- Generate GitLab 17-compatible YAML unless the user confirms a newer baseline.
- Document GitLab 18.11 or later as the target end state when relevant.
- Do not use GitLab typed pipeline inputs for security-sensitive routing while GitLab 17 compatibility is required.
- Use `workflow: rules` to control allowed pipeline creation.
- Use `rules` instead of deprecated `only`/`except`.
- Fail closed with a final `when: never` where rules act as an allowlist.
- Keep branch-to-environment-to-runner mapping in trusted repository logic, preferably `workflow: rules: variables` when branch routing is required.
- Do not expose production environment, production runner selection, or production deployment decisions as user-editable manual/project/group variables.
- Use variable-expanded runner tags only when the installed GitLab/runner configuration supports them and the value comes from trusted workflow rules.
- Route merge-request pipelines to validation or pre-production runners only; merge-request code must never execute on production runners or receive production secrets.
- Production deployment requires protected production branch, protected runner, protected environment, and approvals outside YAML.
- Repository CI YAML is not an access-control boundary by itself.
- Keep deploy, rollback, destructive reset, package publication, and production promotion manual unless approved policy explicitly automates them.
- Do not use Git tags as the production promotion mechanism unless approved for the repository.

## Runner and private image policy

- Private image pull authentication is runner administration because images are pulled before job scripts run.
- Do not manufacture registry authentication in `before_script`.
- Do not require a project `DOCKER_AUTH_CONFIG` only to pull the CI image.
- Kubernetes executor: prefer namespace-scoped Docker registry Secrets wired through runner `image_pull_secrets`.
- Docker executor: prefer a mode-0600 Docker `config.json` for the runner account.
- Runner `config.toml` environment configuration is a fallback only when executor-native auth is unavailable and approved.
- Pull credentials must be read-only and narrowly scoped, with no production deployment privilege.
- Never print, artifact, commit, or decode registry credentials.
- Do not set `image:pull_policy` when runner policy rejects pipeline-defined pull policies.
- If mutable image tags are unavoidable, document cache/refresh risk and assign runner operations as owner of refresh/digest policy.
- Use `FF_KUBERNETES_HONOR_ENTRYPOINT=true` for Ansible Builder-derived images only when local evidence requires it.
- If the runtime UID lacks NSS/passwd identity and SSH or other tooling requires it, fail closed and report a runner-image problem.
- Treat Kubernetes admission warnings, pod RBAC gaps, unapproved helper/init registries, and runner security-context issues as runner-operation findings unless the repository owns them.
- Use `python3` for runner-side Python commands unless local evidence establishes another interpreter.
- Do not install expected CI tooling dynamically when the approved runner image is supposed to provide it; validate versions and report missing tooling.

## Secrets and variables

- Never hardcode credentials, private keys, tokens, live account IDs, customer hostnames, or secret values.
- Prefer environment-scoped File variables for structured/sensitive inputs consumed as files.
- Validate File variables as existing non-empty files without printing contents.
- Copy sensitive files to job-owned temporary paths with restrictive `umask` when tooling needs a stable path, then remove them with `trap` or `after_script`.
- Never persist sensitive files in artifacts, caches, test reports, logs, or generated documentation.
- Parse secret JSON at the native parser/tool boundary; do not round-trip it through unsafe shell serialization.
- Preserve multiline PEM/key bytes exactly. Do not normalize, fold, repair, or echo secret material.
- Use SHA-256 or stronger for integrity checks. Do not use MD5 or SHA-1.
- Commit package coordinates, versions, and checksums when needed; do not commit vendor/generated binaries unless repository policy explicitly requires it.
- Use `CI_JOB_TOKEN` for automated package access when the target GitLab endpoint supports it and project access controls permit it.
- Keep package publication separate from deployment.

## Pipeline generation guidance

Prefer the smallest pipeline that satisfies the stated goal and security contract.

For Ansible repositories, include relevant validation such as:

- `yamllint`
- `ansible-lint`
- `ansible-playbook --syntax-check`
- smoke tests using a dedicated validation/pre-production target where approved

For Terraform repositories, include `terraform fmt`, `terraform validate`, provider lock handling, non-secret plan artifacts where approved, and manual protected apply.

For container repositories, validate Dockerfile/Containerfile, build with the approved engine, avoid leaking registry auth, and push only from protected trusted branches.

For script repositories, include shellcheck, PowerShell checks, Python syntax/type/test commands, or other language-native checks available in the repo.

For documentation repositories, include markdown linting and link checks when practical. Do not add deployment jobs unless the publish target is defined.

Use small reusable scripts under `ci/` for complex validation or deployment boundaries instead of long embedded YAML.

Treat every generated artifact as a producer/consumer contract. Validate generated shell files, environment files, JSON, package metadata, and downstream parser expectations before sourcing or consuming them.

## Troubleshooting classification

When reviewing failures, separate:

- fatal CI findings;
- non-fatal dependency deprecations;
- runner-operation warnings;
- external platform/policy warnings.

Do not attribute failure to warnings without checking the failing command exit code, fatal summary, file/line findings, and final job status.

## Output after implementation

After approved implementation, report:

1. Implementation status and unresolved blockers.
2. Pipeline architecture overview.
3. Files created or changed.
4. Secret/variable and runner requirements.
5. Validation commands run and results.
6. Commands not run and why.
7. Human review items before merge or deployment.
