# Create a Genre

This folder is now a Claude-style skill folder.

Filesystem-safe packaging uses the folder name `create_a_genre` and the upload artifact `create_a_genre.zip`.

## Structure

- `SKILL.md` is the authoritative entrypoint.
- `references/` contains the bundled templates, schema, example JSON, and authoring guides.

## Intended Use

Use this skill when you want Claude to:

1. brainstorm five candidate Wagtales genre ideas
2. help choose a direction
3. generate a final `init.json` using the simplified contract

## Upload Shape

For Claude custom skill upload, package this folder so the ZIP contains the folder itself as the root.

Important: the ZIP's internal entry names must use forward slashes, for example `create_a_genre/SKILL.md`.

If the archive stores Windows-style backslashes such as `create_a_genre\SKILL.md`, Claude may reject the upload with `Zip file contains path with invalid characters`.