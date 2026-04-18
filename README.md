# Skills

This folder now uses Claude-style skill folders as the standard format for reusable workflows.

## Standard Structure

Each reusable workflow should live in its own folder:

```text
skill-name/
  SKILL.md
  references/
    ...supporting files...
```

For this repo, prefer filesystem-safe names that also satisfy Claude skill validation. Use lowercase letters, numbers, and hyphens for skill folder names and matching skill names, for example:

```text
create-a-genre/
create-a-genre_001.zip
```

This avoids upload issues caused by spaces or other path characters in some skill import flows.

## Required File

Each skill folder must contain `SKILL.md`.

`SKILL.md` must start with YAML frontmatter containing:

- `name`: the skill name
- `description`: what the skill does and when Claude should use it

The body of `SKILL.md` should contain:

- when to use the skill
- the workflow Claude should follow
- output expectations
- a resources section pointing to bundled reference files

## Resource Files

Put supporting material under `references/`.

Examples:

- templates
- schemas
- example files
- authoring guides
- style guides
- checklists

Keep `SKILL.md` focused and use `references/` for the heavier detail.

## Mapping From The Old Prompt-Pack Format

Old pattern:

```text
some-workflow/
  user-prompt.md
  README.md
  context/
```

New standard:

```text
some-workflow/
  SKILL.md
  references/
```

Mapping rules:

- `user-prompt.md` becomes the main body of `SKILL.md`
- `context/` becomes `references/`
- local workflow notes may stay in `README.md`, but `SKILL.md` is the source of truth

## Writing Good Skill Metadata

The `description` field is the trigger surface Claude uses to decide when to load the skill.

Write it so it includes:

- the domain
- the task type
- the file or artifact being produced
- important trigger phrases a user is likely to say

For example, mention concrete terms like `Wagtales`, `genre`, `init.json`, `customDataShape`, or `roll placeholders` instead of only abstract descriptions.

## Packaging For Claude Upload

To upload a skill to Claude:

1. Create a ZIP file of the skill folder.
2. The ZIP must contain the skill folder as its root.
3. `SKILL.md` must be inside that root folder.

Correct shape:

```text
create-a-genre_001.zip
  create-a-genre/
    SKILL.md
    references/
```

## Deploy Artifact Naming

Store upload artifacts in a skill-local `deploy/` folder.

Use incrementing filenames in this format:

```text
[name]_001.zip
[name]_002.zip
[name]_003.zip
```

Example:

```text
create-a-genre/
  deploy/
    create-a-genre_001.zip
    create-a-genre_002.zip
```

This keeps uploads reproducible and avoids overwriting a previously known-good package.

## ZIP Build Commands

Use a clean external staging folder so the ZIP does not accidentally include `deploy/`, temporary folders, or recursive artifacts.

Do not use `Compress-Archive` for Claude skill uploads.

Reason: the resulting ZIP may store entry names with Windows backslashes like `create-a-genre\SKILL.md`. Claude's skill updater can reject those with `Zip file contains path with invalid characters`.

Build the ZIP with normalized forward-slash entry names instead.

PowerShell template:

```powershell
$src = "c:\Wagtales\Skills\create-a-genre"
$stageRoot = "c:\Wagtales\temp\create-a-genre_upload_stage"
$stageSkill = Join-Path $stageRoot "create-a-genre"
$zipPath = "c:\Wagtales\Skills\create-a-genre\deploy\create-a-genre_001.zip"

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

Add-Type -AssemblyName System.IO.Compression
Add-Type -AssemblyName System.IO.Compression.FileSystem

$zip = [System.IO.Compression.ZipFile]::Open($zipPath, [System.IO.Compression.ZipArchiveMode]::Create)
try {
  Get-ChildItem $stageSkill -Recurse -File | ForEach-Object {
    $relativePath = [System.IO.Path]::GetRelativePath($stageRoot, $_.FullName).Replace('\\', '/')
    $entry = $zip.CreateEntry($relativePath, [System.IO.Compression.CompressionLevel]::Optimal)
    $entryStream = $entry.Open()
    $fileStream = [System.IO.File]::OpenRead($_.FullName)
    try {
      $fileStream.CopyTo($entryStream)
    }
    finally {
      $fileStream.Dispose()
      $entryStream.Dispose()
    }
  }
}
finally {
  $zip.Dispose()
}
```

After building, verify the ZIP contents before upload.

Correct internal entry names should look like this:

```text
create-a-genre/SKILL.md
create-a-genre/README.md
create-a-genre/references/init-schema.json
```

Avoid archives whose internal entry names contain backslashes:

```text
create-a-genre\SKILL.md
create-a-genre\references\init-schema.json
```

Incorrect shape:

```text
some_skill.zip
  SKILL.md
  references/
```

## Current Skills

- `create-a-genre`
- `template`

## Starter Template

Use `template` as the base when creating a new reusable workflow in this folder.

Recommended process:

1. Copy `template` to a new folder with the final skill name.
2. Replace the frontmatter and body in `SKILL.md`.
3. Add or replace files in `references/`.
4. Remove any example placeholders that no longer apply.
5. Zip the finished skill folder for Claude upload.