# Wagtales Create Template

This folder is now a Claude-style skill folder.

## Quick Download

If you just want the installable skill ZIP and do not want to use Git, download this file directly:

- [Download `wagtales-create-template_001.zip`](deploy/wagtales-create-template_001.zip)

On GitHub, open the link above and use the file download button.

Filesystem-safe packaging uses the folder name `wagtales-create-template` and the upload artifact `wagtales-create-template.zip`.

## Structure

- `SKILL.md` is the authoritative entrypoint.
- `references/` contains the bundled templates, schema, example JSON, and authoring guides.

## Intended Use

Use this skill when you want Claude to:

1. brainstorm Wagtales genre ideas
2. help choose a direction
3. generate a final `init.json` using the simplified contract
4. generate a single NPC object for `world.npcs`
5. generate a single location object for `world.locations`

## Supported Outputs

This skill can return either:

- a full Wagtales `init.json`
- one NPC object for insertion into `world.npcs`
- one location object for insertion into `world.locations`

If the user asks for an individual NPC or location, the skill should return just that object rather than forcing a full template.

## Upload Shape

For Claude custom skill upload, package this folder so the ZIP contains the folder itself as the root.

Important: the ZIP's internal entry names must use forward slashes, for example `wagtales-create-template/SKILL.md`.

If the archive stores Windows-style backslashes such as `wagtales-create-template\SKILL.md`, Claude may reject the upload with `Zip file contains path with invalid characters`.