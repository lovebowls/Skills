# Narrative Style Card Authoring Guide

Use this guide when authoring a single Wagtales `NarrativeStyleCard` object.

## Contract

The output artifact for this skill is one JSON object, not a deck wrapper.

Valid shape:

```json
{
  "id": "card_id",
  "title": "Optional title",
  "description": "Optional description",
  "representative_authors": ["Optional", "examples"],
  "technical_guide": "Optional craft guidance",
  "example": "Optional short style sample",
  "blacklist": ["Optional exclusions"],
  "custom_field": "Allowed",
  "customDataShape": {
    "custom_field": {
      "promptIncludeMask": 2
    }
  }
}
```

Invalid shapes for this skill:

- a `cards` array
- an object with `deckId`
- an outer `narrative_style` wrapper
- a full `deck.json` object with `story_arc` and `narrative_style`

## Required And Optional Fields

- `id` is required.
- `theme` is optional.
- `title` is optional.
- `description` is optional.
- `emoji` is optional.
- `representative_authors` is optional.
- `technical_guide` is optional.
- `example` is optional.
- `blacklist` is optional.
- `customDataShape` is optional.
- Additional authored fields are allowed.

## Field Intent

### id

Use a stable, filesystem-safe, human-readable identifier.

Good:

- `noir_precision`
- `lush_gothic_intimacy`
- `clinical_conspiracy_voice`

Avoid:

- spaces
- punctuation-heavy ids
- generic ids like `style1`

### title

Use when the card benefits from a readable label in editing workflows.

### description

Use as the short player- or editor-facing summary of what the style does.

### representative_authors

Use when a few anchor influences sharpen the intended output. Prefer authors whose prose behavior is legible and distinct.

Do not use the list as a substitute for actual guidance.

### technical_guide

Use for explicit craft instructions. This is the best place to specify things like:

- sentence rhythm
- narrative distance
- sensory density
- handling of exposition
- dialogue strategy
- figurative-language tolerance
- how conflict or dread should surface

### example

Use for a short style sample, usually one compact paragraph.

The sample should demonstrate the intended voice. It should not become long lore, character backstory, or a scene that constrains future content too tightly.

### blacklist

Use for specific prose moves, clichés, tonal mistakes, or structural habits the style should avoid.

Prefer explicit entries like:

- `Jokey quips during moments of danger`
- `Explaining subtext the reader can already infer`
- `Purple metaphor chains for ordinary actions`

## Flexible Custom Fields

Narrative-style cards are deliberately flexible. Additional fields are allowed when they better capture the author's intent.

Useful examples:

- `pacing_bias`
- `dialogue_temperature`
- `interiority_policy`
- `violence_rendering`
- `romance_intensity`
- `mystery_reveal_style`
- `description_density`
- `humor_policy`

Add custom fields only when they create a clearer control surface than cramming everything into `technical_guide`.

## customDataShape Rules

`customDataShape` applies only to authored custom fields on the same card.

Rules:

- Every `customDataShape` key must match a real authored field on the card.
- Do not create orphan `customDataShape` entries.
- Use it when a custom field should be prompt-targeted, visibility-gated, or semantic-query-gated.
- Do not rely on it to hide built-in structural card fields.

See `card-custom-data-shape-guide.md` and `card-semantic-query-guide.md`.

## Authoring Heuristics

- Prefer one strong coherent mode over a bag of disconnected adjectives.
- Keep the card internally consistent.
- If the user asks for paradoxical constraints, resolve them into an operational compromise rather than preserving raw contradiction.
- Use built-in fields for obvious standard concerns and custom fields for specialty controls.
- Keep the object compact enough that a human author could still review it easily.

## Suggested Process

1. Identify the dominant reading experience.
2. Decide which standard fields actually help.
3. Add custom fields only for missing control dimensions.
4. Add `customDataShape` only if some custom fields need runtime gating.
5. Return one JSON object only.