# Genre init.json Authoring Guide

This guide is aimed at another language model generating Wagtales genre `init.json` files.

It focuses on practical authoring choices that make a genre both fun for players and useful to the runtime within the simplified contract now used by this prompt pack.

## What init.json Really Is

`init.json` is not just backstory. It is the initial playable state plus authored prompt-control data.

It should do four jobs well:

1. Establish a compelling premise immediately.
2. Create a playable opening situation with obvious actions.
3. Provide durable state that can evolve across turns.
4. Shape downstream AI behavior with precise, high-leverage instructions.

If a field does not improve one of those four jobs, consider omitting it.

## The Current Contract

The current authored shape is intentionally small:

- `currentTurn`
- `time`
- `location`
- `world.locations`
- `world.npcs`
- `metadata`

Within `metadata`, `world.locations.{id}`, and `world.npcs.{id}`, you may add extra authored properties when they are useful.

## Design For Play, Not Lore

Good genres are interactive engines.

They usually include:

- a clear player role
- a strong source of pressure, uncertainty, or desire
- one or more NPCs with clear functions
- a starting location that already contains tension or opportunity
- a loop that can keep generating good turns

Weak genres often fail because they are mostly premise and atmosphere.

Avoid these patterns:

- a beautiful setting with no immediate problem
- a protagonist with no leverage
- NPCs that are only decorative
- mechanics that depend on too many unstated assumptions
- custom metadata that sounds clever but does not affect prompt behavior

## Structure The Opening Turn Carefully

The opening state should make the first turn easy for the narrative engine.

Aim for:

- one current location that is already vivid and actionable
- at least one important NPC, unless deliberate isolation is core to the design
- a short list of implied next moves the player could plausibly take
- a clear reason the situation will change if the player does nothing

The player should be able to answer, within a few lines:

- Where am I?
- Who matters here?
- What is going wrong or about to happen?
- What could I try first?

## Use Metadata As Control Surfaces

The `metadata` block is not filler. In this simplified contract it should stay focused.

Key fields:

- `genre`: Leave blank unless a concrete value is explicitly required downstream.
- `title`: Should be immediately legible and marketable.
- `description`: Should communicate the fantasy and player role crisply.
- `blacklist`: Only use when specific content needs hard exclusion.
- `tags`: Optional, but useful when they improve discovery or classification.

If you add custom metadata fields, they should provide reusable prompt guidance, world rules, or phase-specific instructions.

## Build Stateful Worlds

A strong genre usually contains state that can change and matter.

Good examples:

- trust, suspicion, rank, debt, hunger, morale, danger level
- weather severity, contamination, crowd tension, fuel, supplies
- whether a route, location, or faction is accessible
- hidden facts that should emerge only when conditions are met

State should be legible and consequential.

Do not add numbers just to look systematic. Add them when they can plausibly influence narration, updates, or choices.

## When To Use Roll Placeholders

Use roll placeholders when authored randomness improves replayability and the result needs to be treated as a concrete fact inside the prompt.

Good use cases:

- choosing a destination, event type, weather pattern, or institutional mood
- generating a bounded year, time, shift, or scarcity level
- ensuring two prompt statements reuse the same resolved fact consistently

Poor use cases:

- randomizing trivial flavor that does not affect play
- replacing real design decisions with dice for no reason
- adding several unrelated rolls that make the premise incoherent

Best practice:

- use `id=` whenever the same random fact appears more than once
- use `choices=` for discrete authored categories
- use `min` and `max` for bounded numeric values
- write the surrounding instruction so the resolved result is treated as truth, not as a suggestion

## When To Use customDataShape

`customDataShape` is one of the most important advanced tools in this format.

It is supported on:

- `metadata`
- each location object
- each NPC object

Use it when a field should exist in the saved state but only become visible to certain prompt families or in certain runtime conditions.

This is useful for:

- hidden information that should emerge later
- update-only instructions that should not leak into narration
- location-specific guidance
- NPC-specific behavior that should only activate under explicit conditions
- world rules or author notes that should only appear in selected prompt families

This is not useful for:

- decorative filtering with no gameplay effect
- overly complex boolean logic that the current system does not support
- filtering fields that do not actually exist in the same object

Remember:

- prompt filtering and visibility conditions are ANDed together
- invalid visibility paths fail closed
- every filtered custom field still needs its actual value authored in the object

## Author NPCs For Function

NPCs should do work inside the scenario.

Good NPC roles include:

- gatekeeper
- ally under pressure
- unreliable guide
- rival
- authority figure
- witness
- caretaker
- tempter

For each important NPC, try to imply:

- what they want
- what they fear
- why they matter in the first few turns
- how they might change over time

If you add custom NPC properties that should only be visible in some prompt families or game states, define matching entries in that NPC's `customDataShape`.

## Author Locations For Action

A starting location should do more than look interesting.

It should support action through:

- meaningful constraints
- opportunities to investigate, negotiate, escape, protect, or acquire something
- connection to other future locations or states

A location description should help the AI infer good next moments, not merely paint scenery.

If you add custom location properties that should only surface under specific conditions, define matching entries in that location's `customDataShape`.

## Keep The JSON Lean

Do not over-author everything up front.

Prefer:

- one strong starting location over five shallow ones
- a few meaningful NPCs over a crowd of empty names
- a small number of high-value custom fields over sprawling metadata

The genre file only needs enough scaffolding to start well and stay coherent.

## A Good Internal Checklist

Before finalizing an `init.json`, check:

1. Is the premise instantly understandable?
2. Is turn 1 playable without inventing missing structure?
3. Does the world contain at least one interesting source of change?
4. Are roll placeholders used only where they improve replayability or consistency?
5. Are any filtered fields technically valid and genuinely useful?
6. Do the NPCs and locations create pressure, not just flavor?
7. Would a player seeing only the title and description want to click this?

If the answer to several of these is no, redesign the genre rather than padding it.