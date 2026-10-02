---
name: cicd-pipeline
description: Design, generate, review, or troubleshoot secure CI/CD pipelines for GitHub Actions or GitLab CI/CD, including self-hosted Fedora/RHEL runners, Podman container builds, SELinux volume handling, FIPS-safe scripting, secrets, artifacts, validation, and pipeline documentation.
---

# CI/CD Pipeline Skill

Use this skill when the user asks for a platform-neutral or cross-platform CI/CD pipeline for GitHub Actions or GitLab CI/CD. Prefer `gitlab-cicd-pipeline-expert` for ACF GitLab-specific branch routing, protected environment, or runner policy work.

This is best modeled as a skill rather than a custom agent because it is a reusable generation/review workflow that benefits from the main agent's direct access to the target repository. Recommend a custom agent only when the user wants an isolated pipeline-review persona, independent review output, or restricted-tool execution.

## Required discovery

Before generating final YAML, inspect or ask for:

- platform: GitHub Actions or GitLab CI/CD;
- runner environment: OS, image, self-hosted/hosted, Fedora/RHEL specifics;
- pipeline goal;
- repository language/build system;
- required secrets and variables by name and scope only;
- artifact needs;
- container engine requirements;
- validation commands and test commands;
- deployment targets and manual gates.

If the platform or deployment target is unclear, ask concise questions before generating final files.

## Platform rules

For GitHub Actions:

- Use standard syntax: `on`, `jobs`, `steps`.
- Prefer verified marketplace actions with explicit semantic version tags such as `actions/checkout@v4`.
- Never use actions pinned to `@main` or `@master`.
- Use `${{ secrets.NAME }}` for secrets and avoid logging secret-derived values.
- Use native artifacts with `actions/upload-artifact` and `actions/download-artifact` when downstream jobs need generated outputs.

For GitLab CI/CD:

- Use explicit `stages`, `image`, and strict `rules` blocks.
- Avoid deprecated `only`/`except`.
- Use `$VARIABLE_NAME` for CI/CD variables and masked/secret inputs.
- Use `artifacts: paths` only for non-secret outputs needed by downstream jobs.

## Runner, container, SELinux, and FIPS requirements

- On Fedora and RHEL self-hosted runners, prefer `podman` over `docker` unless repository evidence requires Docker.
- When wrapper build tools support it, pass explicit container runtime options such as `--container-runtime podman`.
- Add SELinux mount context flags `:Z` or `:z` for container volume mounts when appropriate.
- For host/container file interactions that cannot use normal relabeling, use an explicit and reviewed SELinux strategy such as `--security-opt label=disable`, `chcon`, or `semanage` only when justified.
- Do not use MD5 or SHA-1 for cryptographic checksums, hashing, or key generation.
- Use SHA-256, SHA-512, Ed25519, or RSA-3072+ where cryptographic strength is relevant.
- Use block scalars for multi-line scripts. Avoid long command chains that hide failures.
- Never echo unmasked configuration text into protected file boundaries when a native file, artifact, or structured input mechanism is available.

## Secrets and variables

- Never hardcode keys, passwords, tokens, or personal access tokens.
- Keep platform secrets in the platform's secret mechanism.
- Use per-job `env` or platform-native variables for non-secret configuration.
- Do not print secret values, decoded credentials, private keys, account tokens, or generated secret-bearing config files.
- Do not persist secret-bearing files as artifacts or caches.
- If a tool requires a temporary secret file, create it with restrictive permissions and delete it at job end.

## Generation rules

- Start from the target repository's real files and existing CI conventions.
- Generate the smallest pipeline that satisfies the stated goal.
- Include prerequisite checks for required CLI tools such as `podman`, `ansible-builder`, `python3`, or language build tools.
- Install missing tools at runtime only when that is expected for the runner model; otherwise report runner image prerequisites.
- Use native artifact upload/download or GitLab artifacts when downstream jobs need generated assets.
- Keep deployment, publish, destructive reset, and production promotion behind manual gates unless the user gives an approved automation policy.
- Scope linters and tests to repository-owned files. Exclude generated dependency trees, caches, and vendor directories unless the repo intentionally validates them.

## Output format

When providing a design or implementation, use:

1. Pipeline architecture overview.
2. Files to create or modify.
3. Configuration files with markdown path headers before code blocks.
4. Pipeline documentation suitable for a repository README or `CI-CD_README.md`.
5. Required secrets and variables.
6. Validation commands.
7. Runner or human review prerequisites.

Do not add casual filler before the technical overview when the user asks for final pipeline content.
