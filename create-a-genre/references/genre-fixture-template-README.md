# Simplified init.json Notes

This context pack is focused on authored `init.json` creation only.

## Core Structure

The simplified template includes:

- `currentTurn`
- `time`
- `location`
- `world.locations`
- `world.npcs`
- `metadata`

## Required Minimums

- Each authored location should normally include `id`, `name`, and `description`.
- Each authored NPC should include `id`, `name`, `description`, and `playable`.
- At least one NPC must be authored with `playable: true` or the genre will import but fail at game creation.
- Each playable NPC must include `tone` in `HHH:SSS:LLL` format.
- Non-player NPCs should usually set `playable: false` explicitly.
- `metadata` should include `genre`, `title`, `description`, and `blacklist`.
- If `metadata.tags` includes a `genre` tag, its values should come only from the permitted genre list in `tag-authoring-guide.md`.
- If `metadata.tags` includes `age_rating`, it should be exactly one of `U`, `PG`, `12`, `12A`, `15`, `18`, or `R18`.

## Flexible Authoring

The contract is intentionally open beyond those basics.

- Add extra location properties when they help the AI understand the place or track changing state.
- Add extra NPC properties when they clarify goals, pressure, relationships, or hidden information.
- Add extra metadata properties when they provide reusable prompt guidance or world rules.

## Player-Visible Fields

`world.locations.*.name`, `world.locations.*.description`, `world.npcs.*.name`, and `world.npcs.*.description` are player-visible.

Keep them limited to what the player can already perceive or reasonably know.

If information is hidden, private, systemic, or intended only for AI control, put it in a separate custom field instead of the visible description.

## Important: customDataShape

`customDataShape` is supported on:

- `metadata`
- each location object
- each NPC object

Use it when an authored property should only be visible to certain prompt families or only under certain game-state conditions.

Every filtered property still needs its actual authored value in the same object.