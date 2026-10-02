# CI/CD Pipeline Skill

Copy-ready VS Code/Copilot skill for secure GitHub Actions and GitLab CI/CD pipeline design, generation, review, and troubleshooting.

## Contents

- `SKILL.md` - the runtime skill file.

## Use When

Use this skill for platform-neutral CI/CD work involving GitHub Actions or GitLab CI/CD, self-hosted Fedora/RHEL runners, Podman container builds, SELinux volume handling, FIPS-safe scripting, secrets, artifacts, validation, or pipeline documentation.

For ACF GitLab-specific branch routing, protected production controls, and runner policy work, prefer the `gitlab-cicd-pipeline-expert` skill.

## Install

Copy this whole directory into the target skill directory and keep the folder name unchanged.

Windows PowerShell:

```powershell
$source = "C:\Users\evan.hisey\Git_Repo\standard-prompt-collection\cicd-pipeline"
$target = "C:\path\to\target-repo\.agents\skills\cicd-pipeline"
New-Item -ItemType Directory -Force -Path (Split-Path $target) | Out-Null
Copy-Item -Recurse -Force -Path $source -Destination $target
```

Linux/macOS Bash:

```bash
source="$HOME/Git_Repo/standard-prompt-collection/cicd-pipeline"
target="/path/to/target-repo/.agents/skills/cicd-pipeline"
mkdir -p "$(dirname "$target")"
cp -R "$source" "$target"
```

## Notes

This package is intentionally stored as a top-level directory in this repository. It is not installed by placing it under this repository's `.github` directory.
