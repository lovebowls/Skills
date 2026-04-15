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
- Each authored NPC should normally include `id` and `name`, and should usually include `description` when the character matters immediately.
- `metadata` should include `genre`, `title`, `description`, and `blacklist`.

## Flexible Authoring

The contract is intentionally open beyond those basics.

- Add extra location properties when they help the AI understand the place or track changing state.
- Add extra NPC properties when they clarify goals, pressure, relationships, or hidden information.
- Add extra metadata properties when they provide reusable prompt guidance or world rules.

## Important: customDataShape

`customDataShape` is supported on:

- `metadata`
- each location object
- each NPC object

Use it when an authored property should only be visible to certain prompt families or only under certain game-state conditions.

Every filtered property still needs its actual authored value in the same object.