---
name: wagtales-create-narrative-card
description: Create a single Wagtales narrative-style card JSON object. Use for interviewing an author, shaping prose/voice requirements, and generating one import-ready narrative card with optional customDataShape filters.
---

# Wagtales Create Narrative Card

Use this skill when the user wants to create one custom Wagtales narrative-style card for later import into the app.

## When To Use This Skill

Use it when the user wants to:

- design a custom narrative voice or prose mode for a Wagtales genre
- turn a conversation about writing style into one valid narrative-style card
- author optional fields such as `title`, `description`, `representative_authors`, `technical_guide`, `example`, or `blacklist`
- add bespoke card-level custom fields governed by `customDataShape`
- use prompt-family filtering, runtime visibility filtering, or semantic query gating on authored custom fields

## Working Principles

- Treat the task as structured systems authoring, not generic prose generation.
- The output is one JSON object representing one card, never a deck wrapper or card array.
- `NarrativeStyleCard` is intentionally flexible: standard fields are optional except for `id`, and additional authored fields are allowed when they materially help the runtime express the author's intent.
- Use standard fields when they are the cleanest fit, but do not force every card to use every standard field.
- Prefer concise, high-leverage authored guidance over bloated manifesto text.
- Use `customDataShape` only for authored custom fields that need prompt routing, visibility gates, or semantic-query control.
- Every `customDataShape` entry must correspond to a real authored field on the same card using the exact same key.
- Built-in structural fields such as `id`, `title`, `description`, `theme`, `emoji`, `representative_authors`, `technical_guide`, `example`, and `blacklist` should normally stay directly authored rather than being treated as hidden control metadata.
- The card should give the runtime crisp stylistic guidance without becoming contradictory, redundant, or overconstrained.

## Workflow

### Phase 1

Interview the user to extract the intended narrative behavior.

Cover these areas as needed:

1. The desired voice, tone, and reading experience
2. What the narrator should emphasize or avoid
3. Whether the author wants exemplars via `representative_authors`
4. Whether the author wants direct craft instructions via `technical_guide`
5. Whether the author wants a short style sample via `example`
6. Whether the author wants explicit exclusions via `blacklist`
7. Whether custom authored fields would help, such as pacing directives, sensory emphasis, dialogue stance, interiority rules, mystery handling, or violence-description policy
8. Whether any custom field should be filtered with `promptIncludeMask`, `visibilityConditions`, or `semanticQueries`

If the user gives a vague brief, ask focused follow-ups or infer a coherent card shape from the stated goals.

### Phase 2

Produce the final card.

When doing so:

- Output valid JSON only.
- Output a single card object only.
- Do not output a `cards` array.
- Do not output `deckId`, `story_arc`, `narrative_style`, or any outer deck wrapper.
- Always include `id`.
- Include only the fields that help express the author's intent.
- Use the built-in narrative-style fields when they fit naturally.
- Additional custom fields are allowed and often desirable when they provide a clearer control surface than overloading the built-in fields.
- If you author custom fields that need runtime gating, add matching `customDataShape` entries under the same field keys.
- If you use `semanticQueries`, keep them scalar, concrete, and directly relevant to when the field should appear.
- Keep `example` short enough to function as a style reference rather than a miniature story.
- Keep `technical_guide` operational and actionable.
- Use `blacklist` for specific forbidden habits, clichés, tonal violations, or prose moves.
- Avoid filler, duplicated instructions, and mutually incompatible style directives.
- Do not include prose outside the JSON.

## Output Expectations

- The output must be one JSON object conforming to the narrative-style card contract in `references/narrative-style-card-template.json` and `references/narrative-style-card-authoring-guide.md`.
- `id` is required.
- All other standard narrative-style fields are optional.
- Arbitrary extra authored fields are allowed.
- `customDataShape` is optional and applies only to authored custom fields on this same card.
- Never emit deck-level structure.

## Quality Bar

The final card should:

- define a distinctive writing mode a user could deliberately choose
- be specific enough to shape output consistently across many turns
- avoid contradicting itself across visible and hidden instructions
- use optional fields intentionally rather than mechanically filling them all in
- include custom fields only when they add real control leverage
- make `customDataShape` rules legible and technically valid when present

## Resources

Read these bundled files as needed:

- `references/narrative-style-card-template.json`
- `references/narrative-style-card-authoring-guide.md`
- `references/examples.md`
- `references/checklist.md`
- `references/card-custom-data-shape-guide.md`
- `references/card-semantic-query-guide.md`