---
name: acf-patch-management-compatibility
description: Generate, review, or update Ansible automation for non-repository Linux applications so it remains compatible with the acf.patch_management collection, including canonical patch facts, optional integration boundaries, Splunk/non-repo app roles, AAP inventory contracts, and CI validation.
---

# ACF Patch Management Compatibility Skill

Use this skill when generating, reviewing, or updating automation in `non-repo-linux-apps` or related repositories that must remain compatible with the `acf.patch_management` collection.

This is best as a skill rather than a custom agent because it provides reusable compatibility contracts while letting the main agent inspect and edit the target Ansible repositories. Recommend a custom agent only when the user wants an isolated compatibility reviewer that returns a separate finding report without making edits.

## Initial repository reads

Before editing, inspect the relevant local files when available. If a reference repository or file is unavailable, continue using the compatibility contract in this skill and state that repository-specific verification could not be performed.

For `acf-patch-management`, read when present:

- `acf-patch-management/README.md`
- `acf-patch-management/playbooks/example_os_patch.yml`
- `acf-patch-management/roles/patch_common/defaults/main.yml`
- `acf-patch-management/roles/patch_common/tasks/init_facts.yml`
- `acf-patch-management/roles/patch_common/tasks/map_legacy_facts.yml`
- `acf-patch-management/roles/linux_patching/defaults/main.yml`
- `acf-patch-management/roles/linux_patching/tasks/main.yml`
- `acf-patch-management/roles/windows_patching/defaults/main.yml`
- `acf-patch-management/roles/windows_patching/tasks/third_party_stub.yml`
- `acf-patch-management/tests/smoke.yml`
- `acf-patch-management/ci/smoke_validate.py`

For Splunk Forwarder or third-party package work, also read when present:

- `3rd-party-updates/roles/splunk_forwarder/defaults/main.yml`
- `3rd-party-updates/roles/splunk_forwarder/tasks/main.yml`
- `non-repo-linux-apps/README.md`
- `non-repo-linux-apps/galaxy.yml`
- `non-repo-linux-apps/requirements.yml`
- `non-repo-linux-apps/roles/splunk_forwarder/defaults/main.yml`
- `non-repo-linux-apps/roles/splunk_forwarder/tasks/main.yml`

## Compatibility contract

Preserve these `acf.patch_management` boundaries:

- `acf.patch_management` owns OS patching, canonical patch facts, and legacy fact mapping.
- Calling repositories own inventory selection, AAP job templates, connection setup, AWS WorkSpaces control-plane behavior, report templates, and final `set_stats` output.
- `non-repo-linux-apps` must not generate inventory files or override upstream AAP connection variables.
- Linux automation should assume SSH with caller-provided credentials and `become` where package installation requires privilege.
- Do not add secrets, access keys, passwords, account IDs, role ARNs, or host-specific values.
- Use existing variable names or documented AAP credential mechanisms.
- Do not make `acf.patch_management` depend unconditionally on optional private application collections.
- Optional integrations belong in caller requirements or explicit role variables.

## Canonical patch facts

Do not rename or repurpose these `acf.patch_management.patch_common` facts:

```yaml
patch_platform: unknown
patch_inventory_valid: true
patch_host_eligible: true
patch_skip_reason: ""
patch_connection_status: succeeded
patch_status: pending
patch_error_records: []
patch_updates_changed: false
patch_update_count: 0
patch_rebooted: false
patch_post_patch_connection_ok: false
patch_windows_attempted_kbs: []
patch_windows_installed_kbs: []
patch_windows_failed_kbs: []
patch_windows_rejected_kbs: []
patch_windows_microsoft_addition_kbs: []
patch_microsoft_additions_status: not_applicable
patch_third_party_status: not_applicable
```

If Linux non-repo application roles need to report into a patching workflow, publish app-specific aggregate facts in the application collection, then map them at an explicit integration point. Do not silently mutate canonical patch facts inside a standalone app role unless the integration task is intentionally part of `acf.patch_management` or a caller playbook.

Existing legacy mappings are controlled by:

- `patch_common_map_simplified_facts`
- `patch_common_map_acf_facts`

Let `roles/patch_common/tasks/map_legacy_facts.yml` remain the owner of those mappings unless the user explicitly requests a patch-management change.

## Optional third-party integration model

Current optional third-party integration is Windows-specific:

```yaml
patch_windows_third_party_enabled: false
patch_windows_third_party_roles: []
windows_patching_third_party_enabled: "{{ patch_windows_third_party_enabled }}"
windows_patching_third_party_roles: "{{ patch_windows_third_party_roles }}"
```

For Linux non-repo application integration, follow the same optional/dynamic pattern if integration into `acf.patch_management` is requested. Use Linux-specific variable names such as:

```yaml
patch_linux_non_repo_apps_enabled: false
patch_linux_non_repo_app_roles: []
linux_patching_non_repo_apps_enabled: "{{ patch_linux_non_repo_apps_enabled }}"
linux_patching_non_repo_app_roles: "{{ patch_linux_non_repo_app_roles }}"
```

Map from application aggregate facts, for example:

- `acf_non_repo_linux_update_status`
- `acf_non_repo_linux_update_changed`
- `acf_non_repo_linux_update_error_records`
- `acf_non_repo_linux_update_results`

Recommended canonical mappings for an explicit Linux integration task:

- `patch_third_party_status`: app aggregate status, or `skipped_disabled` when disabled.
- `patch_updates_changed`: existing value OR app aggregate changed flag.
- `patch_error_records`: append app errors with `phase: non_repo_linux_apps`.

Do not add a Linux non-repo integration task to `acf.patch_management` unless the user explicitly asks for that repository to be changed.

## Linux application role requirements

For roles in `non-repo-linux-apps`:

- Use fully qualified collection names for modules.
- Prefer `ansible.builtin.dnf` or `ansible.builtin.yum` for RPM installs; support the target package manager explicitly.
- Use `ansible.builtin.package_facts` for installed package detection when possible.
- Use `ansible.builtin.command` only for version commands with no purpose-built module, with `changed_when: false` and clear `failed_when` behavior.
- For controller-side S3 artifact access, use `amazon.aws.s3_object` for listing and `amazon.aws.aws_s3` for downloads, matching the existing Splunk model.
- Delegate AWS artifact operations to `localhost` and consume inherited runtime IAM credentials.
- Do not assume roles or embed credentials unless the repository already documents that model.
- Validate supported OS families before package changes. Current Splunk RPM targeting is RHEL 9 and Amazon Linux.
- Keep package GPG checks enabled by default. Only disable GPG checks through a documented variable and approved exception.
- Publish role aggregate facts under the `acf_non_repo_linux_*` namespace, not `acf_third_party_*` or `simplified_patch_*`.

## Collection and CI requirements

For `non-repo-linux-apps` collection work:

- Keep collection metadata in `galaxy.yml` aligned with `acf.non_repo_linux_apps`.
- Keep dependency declarations in `requirements.yml`; do not rely on caller repositories to install required public collections for this collection's own CI.
- Use `registry.management.acf.gov/oci/operations/op-tools:2.0` in GitLab CI unless local evidence shows a newer approved image.
- Do not replace the CI image with upstream `python:*` images or install ad hoc `ansible-core` with pip unless the user explicitly requests it.
- Build with `ansible-galaxy collection build --force` and syntax-check standalone playbooks.
- Add or update smoke tests for structure, required variables, optional integration defaults, and prohibited secret/role-assumption strings.

## Validation commands

Run the safest available checks from the relevant repository root:

```bash
ansible-galaxy collection install -r requirements.yml --collections-path .ansible/collections
ansible-galaxy collection build --force
ansible-playbook --syntax-check -i localhost, playbooks/splunk_forwarder.yml
ansible-playbook -i localhost, -c local tests/smoke.yml
```

For `acf-patch-management` changes, also run when available:

```bash
ansible-playbook --syntax-check playbooks/example_os_patch.yml -e patch_platform=linux
ansible-playbook --syntax-check playbooks/example_os_patch.yml -e patch_platform=windows
python ci/smoke_validate.py
ansible-lint .
```

If local Ansible tooling is unavailable, state that validation is blocked and name the command that failed. Do not execute playbooks against live NGSC or WorkSpaces hosts as validation.

## Response after changes

When responding after changes, include:

1. Files changed and why.
2. Compatibility points preserved from `acf.patch_management`.
3. Validation commands actually run and results.
4. Commands that could not run and the reason.
5. Human review items before merge or AAP deployment.
