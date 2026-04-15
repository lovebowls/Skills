# Skill Template

Use this folder as the starting point for a new Claude-style skill.

## Steps

1. Copy this folder to a new skill-specific folder name.
2. Edit `SKILL.md`.
3. Add real files under `references/`.
4. Remove placeholder text.
5. Create a `deploy/` folder inside the skill.
6. Package the finished folder as an incremented ZIP for Claude upload.

## Deploy Naming

Use incrementing filenames:

```text
[name]_001.zip
[name]_002.zip
```

Example:

```text
my_skill_001.zip
```

## Build Command Template

Use a clean staging folder outside the skill folder.

```powershell
$src = "c:\path\to\My_Skill"
$stageRoot = "c:\temp\my_skill_upload_stage"
$stageSkill = Join-Path $stageRoot "my_skill"
$zipPath = "c:\path\to\My_Skill\deploy\my_skill_001.zip"

if (Test-Path $stageRoot) {
	Remove-Item $stageRoot -Recurse -Force -ErrorAction SilentlyContinue
}

New-Item -ItemType Directory -Path (Join-Path $stageSkill "references") -Force | Out-Null
Copy-Item (Join-Path $src "SKILL.md") $stageSkill -Force
Copy-Item (Join-Path $src "README.md") $stageSkill -Force
Copy-Item (Join-Path $src "references\*") (Join-Path $stageSkill "references") -Recurse -Force

if (Test-Path $zipPath) {
	Remove-Item $zipPath -Force -ErrorAction SilentlyContinue
}

Compress-Archive -Path $stageSkill -DestinationPath $zipPath
```