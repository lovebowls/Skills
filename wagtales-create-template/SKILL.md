---
name: wagtales-create-template
description: Create Wagtales genre init.json templates. Use for brainstorming, designing, validating, or generating Wagtales templates with customDataShape filters or roll placeholders.
---

# Wagtales Create Template

Use this skill for creating new Wagtales genre `init.json` files.

## When To Use This Skill

Use it when the user wants to:

- brainstorm new Wagtales story ideas
- turn a chosen idea into a valid `init.json`
- design metadata, NPC, or location properties with `customDataShape`
- use prompt-family filtering or runtime visibility filtering
- use roll placeholders inside authored prompt-facing text
- create a genre that fits the simplified Wagtales `init.json` contract

## Working Principles

- Treat this as a hybrid creative design and software authoring task.
- Treat the available authoring features as a small programming language for story systems: state variables, gated fields, semantic checks, and roll-based resolution are tools for building bespoke gameplay logic.
- The output should be expressive, dramatically interesting, and technically precise.
- Think like a senior systems designer authoring content for a runtime, not like a novelist writing a synopsis.
- Every authored field should justify its existence by improving replayability, clarity, statefulness, pacing, or AI behavior.
- Prefer strong gameplay branching, dynamic state, and controllable variation over passive lore.
- Use roll placeholders and `customDataShape` only where they create real leverage.
- Compose a genre-specific control surface from the primitives available; do not default to the same metadata fields, location pattern, NPC pattern, or progression structure used by bundled examples.
- Keep the setting legible to a player on turn 1.
- Make the premise commercially attractive: immediately graspable, emotionally legible, and rich in emergent situations.

## Workflow

### Phase 1

First, give exactly 3 candidate genre ideas to brainstorm.

For each idea provide:

1. Title
2. One-sentence hook
3. Why it works well for Wagtales mechanically
4. The main gameplay loop
5. What kind of state, filters, or roll-driven variation it could exploit
6. Risks or design traps to avoid

Constraints for the three ideas:

- Make them materially different from one another.
- All should strongly exploit prompt/world variable filters, roll placeholders and semantic questions.
- At least 1 should be grounded and realistic rather than fantastical.

Do not write JSON yet.
Do not choose for the user.
Do not collapse the ideas into vague blurbs.

### Phase 2

After the user has finalized their choice, produce the final `init.json`.

When doing so:

- Output valid JSON only.
- Use the bundled template and schema as a contract, not as a content outline.
- Prefer minimal but high-leverage authored data rather than bloated filler.
- Ensure the opening state creates immediate play.
- Build a structure that fits the chosen premise rather than echoing the layout of `sample-good-init-aevum.json`.
- Ensure every NPC includes `id`, `name`, `description`, and `playable`.
- Ensure at least one NPC in `world.npcs` is marked `playable: true` so the genre can actually be started.
- Ensure every playable NPC includes a `tone` value in `HHH:SSS:LLL` format.
- Non-playable NPCs may include `tone`, but do not need it unless it adds useful emotional guidance.
- Treat NPC and location `name` and `description` as player-visible UI content.
- Keep those visible fields superficial and in-world: they should describe only what the player would already know on sight or at the current moment.
- Do not hide motives, secrets, solution logic, persuasion hooks, engine instructions, or future reveals inside visible `name` or `description` fields.
- Put hidden, engine-only, or prompt-control information into separate custom fields, and use `customDataShape` when that information should be restricted to specific prompt families or runtime conditions.
- Use only permitted metadata tag values for `genre` and `age_rating`.
- Do not invent compound or bespoke genre labels when they can be expressed as a combination of permitted tags.
- Include only fields that have a clear runtime purpose.
- Use second-person or other narration-control metadata only when it meaningfully constrains the downstream model.
- If using `customDataShape`, ensure every filtered field has a matching concrete authored field.
- If using roll placeholders, use only the supported syntax from the bundled guide.
- If using semantic queries, visibility filters, or rolls, use them to express premise-specific logic, thresholds, uncertainty, and revelation timing rather than reproducing the same mechanisms or field layouts as the example.
- Make location, NPC, and metadata text specific enough to guide the AI strongly.
- Avoid generic placeholders, TODO text, and vague abstractions.
- Do not include explanatory prose outside the JSON.

## Quality Bar

The final genre should:

- have a strong turn-1 premise
- suggest a sustainable arc across many turns
- create opportunities for meaningful choices
- support changing state over time
- give the AI crisp instructions without overconstraining it
- be distinctive enough that a player could choose it from a list immediately
- feel authored from first principles for its own premise, not like a reskinned variant of the sample genre

## Resources

Read these bundled files as needed:

- `references/init-template.json` for the base authored shape
- `references/init-schema.json` for the structural contract
- `references/sample-good-init-aevum.json` for one worked example of the feature set, not a structural template to imitate
- `references/genre-init-authoring-guide.md` for general init.json design guidance
- `references/ROLL-PLACEHOLDER-AUTHOR-GUIDE.md` for supported roll syntax
- `references/prompt-world-variable-filters-admin-guide.md` for `customDataShape`, `promptIncludeMask`, and visibility rules
- `references/semantic-query-author-guide.md` for `semanticQueries`, query shape, and genre-safe host scope
- `references/tone-system-authoring-guide.md` for assigning NPC tone values in `HHH:SSS:LLL` format
- `references/tag-authoring-guide.md` for the allowed `genre` and `age_rating` tag values used by the product
- `references/genre-fixture-template-README.md` for simplified contract notes