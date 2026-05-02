# Wagtales Create Narrative Card

This folder is a Claude-style skill for generating a single Wagtales narrative-style card as JSON.

Filesystem-safe packaging uses the folder name `wagtales-create-narrative-card` and the upload artifact `wagtales-create-narrative-card.zip`.

## Structure

- `SKILL.md` is the authoritative entrypoint.
- `references/` contains the narrative-card template, authoring guidance, examples, and copied card-specific filter references.

## Intended Use

Use this skill when you want Claude to:

1. interview an author about the desired narrative voice, prose behavior, exclusions, and special runtime control fields
2. translate those requirements into one valid narrative-style card object
3. return JSON that can be pasted into `narrative_style.cards[]` in a genre `deck.json`

## Output Contract

The generated artifact is a single JSON object representing one `NarrativeStyleCard`.

Important:

- Do not output the outer `narrative_style` deck object.
- Do not output a `cards` array.
- Do not wrap the card inside a larger `deck.json` structure.
- The user can import or paste the single object into the app afterward.

## Upload Shape

For Claude custom skill upload, package this folder so the ZIP contains the folder itself as the root.

Important: the ZIP's internal entry names must use forward slashes, for example `wagtales-create-narrative-card/SKILL.md`.

If the archive stores Windows-style backslashes such as `wagtales-create-narrative-card\SKILL.md`, Claude may reject the upload with `Zip file contains path with invalid characters`.