# Skill Template

Use this folder as the starting point for a new Claude-style skill.

## Steps

1. Copy this folder to a new skill-specific folder name.
2. Edit `SKILL.md`.
3. Add real files under `references/`.
4. Remove placeholder text.
5. Create a `deploy/` folder inside the skill.
6. Package the finished folder as a newly incremented ZIP for Claude upload.

## Deploy Naming

Every deploy must create a new incremented filename. Do not overwrite the previous ZIP.

Use incrementing filenames:

```text
[name]_001.zip
[name]_002.zip
[name]_003.zip
```

Example:

```text
my_skill_001.zip
my_skill_002.zip
```

If `my_skill_001.zip` already exists, the next deploy is `my_skill_002.zip`. Each rebuild increments again, even if the change is small.

## Build Command Template

Use a clean staging folder outside the skill folder. Set the ZIP filename to the next unused deploy number before building.

```powershell
$src = "c:\path\to\My_Skill"
$stageRoot = "c:\temp\my_skill_upload_stage"
$stageSkill = Join-Path $stageRoot "my_skill"
$version = "002"
$zipPath = "c:\path\to\My_Skill\deploy\my_skill_$version.zip"

if (Test-Path $stageRoot) {
	Remove-Item $stageRoot -Recurse -Force -ErrorAction SilentlyContinue
}

New-Item -ItemType Directory -Path (Join-Path $stageSkill "references") -Force | Out-Null
Copy-Item (Join-Path $src "SKILL.md") $stageSkill -Force
Copy-Item (Join-Path $src "README.md") $stageSkill -Force
Copy-Item (Join-Path $src "references\*") (Join-Path $stageSkill "references") -Recurse -Force

Compress-Archive -Path $stageSkill -DestinationPath $zipPath
```

Before running the build, check the `deploy/` folder and bump `$version` to the next number. Example: if `_004.zip` is the latest file, build `_005.zip` next.