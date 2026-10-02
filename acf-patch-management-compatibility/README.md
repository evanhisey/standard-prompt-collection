# ACF Patch Management Compatibility Skill

Copy-ready VS Code/Copilot skill for generating, reviewing, and updating Ansible automation that must remain compatible with the `acf.patch_management` collection.

## Contents

- `SKILL.md` - the runtime skill file.

## Use When

Use this skill for `non-repo-linux-apps`, Splunk/non-repo application roles, optional Linux app integration, canonical patch facts, AAP inventory contracts, and CI validation that must preserve `acf.patch_management` compatibility.

## Install

Copy this whole directory into the target skill directory and keep the folder name unchanged.

Windows PowerShell:

```powershell
$source = "C:\Users\evan.hisey\Git_Repo\standard-prompt-collection\acf-patch-management-compatibility"
$target = "C:\path\to\target-repo\.agents\skills\acf-patch-management-compatibility"
New-Item -ItemType Directory -Force -Path (Split-Path $target) | Out-Null
Copy-Item -Recurse -Force -Path $source -Destination $target
```

Linux/macOS Bash:

```bash
source="$HOME/Git_Repo/standard-prompt-collection/acf-patch-management-compatibility"
target="/path/to/target-repo/.agents/skills/acf-patch-management-compatibility"
mkdir -p "$(dirname "$target")"
cp -R "$source" "$target"
```

## Notes

This package is intentionally stored as a top-level directory in this repository. It is not installed by placing it under this repository's `.github` directory.
