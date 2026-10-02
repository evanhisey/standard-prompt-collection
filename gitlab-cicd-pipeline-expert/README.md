# GitLab CI/CD Pipeline Expert Skill

Copy-ready VS Code/Copilot skill for designing, generating, reviewing, and troubleshooting GitLab CI/CD pipelines in the ACF GitLab environment.

## Contents

- `SKILL.md` - the runtime skill file.

## Use When

Use this skill for ACF GitLab CI/CD work that needs GitLab 17-compatible YAML, secure branch routing, protected production controls, runner/private image handling, CI variables, validation jobs, deployment gates, promotion, rollback, or GitLab 18 modernization planning.

## Install

Copy this whole directory into the target skill directory and keep the folder name unchanged.

Windows PowerShell:

```powershell
$source = "C:\Users\evan.hisey\Git_Repo\standard-prompt-collection\gitlab-cicd-pipeline-expert"
$target = "C:\path\to\target-repo\.agents\skills\gitlab-cicd-pipeline-expert"
New-Item -ItemType Directory -Force -Path (Split-Path $target) | Out-Null
Copy-Item -Recurse -Force -Path $source -Destination $target
```

Linux/macOS Bash:

```bash
source="$HOME/Git_Repo/standard-prompt-collection/gitlab-cicd-pipeline-expert"
target="/path/to/target-repo/.agents/skills/gitlab-cicd-pipeline-expert"
mkdir -p "$(dirname "$target")"
cp -R "$source" "$target"
```

## Notes

This package is intentionally stored as a top-level directory in this repository. It is not installed by placing it under this repository's `.github` directory.
