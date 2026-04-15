# Roll Placeholder Author Guide

## Purpose

This guide explains how genre and preset authors can use roll placeholders in authored text such as `Authors_note`, `ai_instructions`, and other prompt-facing metadata.

Roll placeholders are resolved server-side before the prompt is sent to the model. The model does not see placeholder syntax and does not perform the roll math itself. It only sees the final resolved values.

This makes authored randomization deterministic within a single prompt and removes the need to rely on the model to interpret roll tables or formulas correctly.

## Where You Can Use Roll Placeholders

Use roll placeholders anywhere authored text is inserted into prompts, including:

- `metadata.Authors_note`
- `world.locations.{id}.ai_instructions`
- other authored metadata or scenario instructions that become part of prompt text

If a piece of text is compiled into a prompt, roll placeholders inside that text will be resolved automatically.

## Basic Syntax

The new syntax is:

```text
[[ROLL:dN]]
[[ROLL:dN|id=some_name]]
[[ROLL:dN|min=X|max=Y]]
[[ROLL:dN|choices=a,b,c]]
[[ROLL|choices=a,b,c]]
```

Examples:

```text
[[ROLL:d6]]
[[ROLL:d100|min=-1500|max=2000|id=arrival_year]]
[[ROLL:d4|choices=City,Town,Countryside,Wilderness|id=arrival_location_type]]
[[ROLL|choices=City,Town,Countryside,Wilderness]]
```

## Copy-Paste Templates

Use these as starter patterns in authored prompt text.

### Raw roll template

```text
[[ROLL:d6]]
```

### Reusable roll ID template

```text
[[ROLL:d6|id=my_roll]]
```

### Numeric range template

```text
[[ROLL:d100|min=-1500|max=2000|id=arrival_year]]
```

### Ordered choices template

```text
[[ROLL:d4|choices=City,Town,Countryside,Wilderness|id=arrival_location_type]]
```

### Ordered choices shorthand template

```text
[[ROLL|choices=City,Town,Countryside,Wilderness|id=arrival_location_type]]
```

### Reuse one roll in two forms template

```text
[[ROLL:d4|id=destination_roll]]
[[ROLL:d4|choices=City,Town,Countryside,Wilderness|id=destination_roll]]
```

## Supported Features

### 1. Raw Die Roll

Use this when you want the rolled number directly.

```text
Choose [[ROLL:d4]] carriable objects.
```

Possible output seen by the model:

```text
Choose 3 carriable objects.
```

### 2. Reusable Roll IDs

Use `id=` when the same underlying roll must be reused multiple times in one prompt.

```text
The transport year is [[ROLL:d100|min=-1500|max=2000|id=arrival_year]].
Record the same year in any state updates: [[ROLL:d100|min=-1500|max=2000|id=arrival_year]].
```

Both placeholders will resolve from the same underlying `d100` roll.

Important:

- Reusing the same `id` with the same die is allowed.
- Reusing the same `id` with a different die is invalid and will fail.

### 3. Numeric Range Mapping

Use `min` and `max` when you want a roll mapped to an inclusive numeric range.

```text
pick a number between 1500BC and 2000AD -> [[ROLL:d100|min=-1500|max=2000|id=arrival_year]]
```

Resolution rule:

- The raw roll is mapped linearly from `1..dN` into `min..max`
- The mapped result is rounded to the nearest integer
- `min` and `max` are both inclusive endpoints

Authoring notes:

- You must provide both `min` and `max`
- `min` may be negative
- `min` may equal `max`
- `min` cannot be greater than `max`

### 4. Ordered Choice Mapping

Use `choices=` when each die face should map to a specific label.

```text
[[ROLL:d4|choices=City,Town,Countryside,Wilderness|id=arrival_location_type]]
```

Shorthand is also supported when the die size can be inferred from the number of choices:

```text
[[ROLL|choices=City,Town,Countryside,Wilderness|id=arrival_location_type]]
```

Resolution rule:

- Roll `1` selects the first choice
- Roll `2` selects the second choice
- and so on

Authoring notes:

- If you supply an explicit die, the number of choices must exactly match the die size
- If you omit the die, the backend infers the die size from the number of choices
- Choices are comma-separated
- Choices are trimmed for surrounding spaces

## Full Authoring Examples

### Example: Random item count

```text
The player may choose [[ROLL:d4|id=item_count]] carriable objects.
```

### Example: Random year in a historical range

```text
When ready to transport the player, determine the year like this:
pick a number between 1500BC and 2000AD -> [[ROLL:d100|min=-1500|max=2000|id=arrival_year]]
```

### Example: Random settlement category

```text
Use this settlement type when building the destination:
[[ROLL:d4|choices=City,Town,Countryside,Wilderness|id=arrival_location_type]]
```

### Example: Reusing one roll in multiple ways

```text
The destination category roll is [[ROLL:d4|id=destination_roll]].
Interpret that same roll as [[ROLL:d4|choices=City,Town,Countryside,Wilderness|id=destination_roll]].
```

In this example:

- both placeholders use the same raw `d4` result
- the first emits the raw number
- the second emits the mapped label

## Invalid Legacy Syntax

The old slot-based syntax is no longer supported.

Invalid:

```text
[[ROLL:d100:1]]
[[ROLL:d4:1]]
```

Do not author new content using slot numbers.

There is no backward-compatibility layer that resolves legacy slot syntax into values.

Current runtime behavior:

- invalid or legacy placeholders are left in the prompt as literal text
- prompt compilation continues
- the AI receives the unresolved placeholder and may still infer the intended meaning from the surrounding text

That means legacy syntax is tolerated, but not supported.

## Validation Rules

The resolver treats the placeholder as invalid if any of the following are true:

- the die spec is missing or malformed
- the die size is not a positive integer
- an unsupported option key is used
- the same option is provided more than once
- `min` is present without `max`
- `max` is present without `min`
- `min > max`
- `choices` is empty
- the number of `choices` does not equal the die size
- `choices` is combined with `min` or `max`
- the same `id` is reused with a different die size

Supported option keys are only:

- `id`
- `min`
- `max`
- `choices`

If a placeholder is invalid:

- the backend logs a warning
- the original placeholder text is preserved unchanged
- the request does not fail just because of that placeholder

Example:

```text
[[ROLL:d100:1]]
```

will stay as:

```text
[[ROLL:d100:1]]
```

instead of causing prompt compilation to abort.

## Authoring Best Practices

- Treat resolved values as facts, not suggestions.
- Write instructions so the model uses the resolved value directly.
- Prefer explicit wording such as "set the year to ..." rather than asking the model to perform a calculation.
- Use `id=` whenever two or more parts of the prompt must stay consistent.
- Use `choices=` for named categories rather than describing a mapping in prose.
- Use `min` and `max` for bounded numeric values.

Better:

```text
Set the arrival year to [[ROLL:d100|min=-1500|max=2000|id=arrival_year]].
```

Worse:

```text
Roll a d100 and calculate a year between 1500BC and 2000AD.
```

## Recommended Migration Pattern

If you have old content like this:

```text
pick a number between 1500BC and 2000AD -> -1500 + (([[ROLL:d100:1]] - 1) / 99 * 3500), round to nearest integer
[[ROLL:d4:1]] -> [City, Town, Countryside, Wilderness]
```

Replace it with this:

```text
pick a number between 1500BC and 2000AD -> [[ROLL:d100|min=-1500|max=2000|id=arrival_year]]
[[ROLL:d4|choices=City,Town,Countryside,Wilderness|id=arrival_location_type]]
```

## Mental Model For Authors

Think of roll placeholders as a text macro system with built-in randomization:

- the backend rolls
- the backend resolves the placeholder into plain text
- the model sees only the finished text

If you need the same random fact more than once, give it an `id`.

If you need a number in a range, use `min` and `max`.

If you need a named category, use `choices`.

## Quick Reference

```text
Raw roll:
[[ROLL:d6]]

Reusable raw roll:
[[ROLL:d6|id=foo]]

Mapped numeric range:
[[ROLL:d100|min=-1500|max=2000|id=arrival_year]]

Mapped named choice:
[[ROLL:d4|choices=City,Town,Countryside,Wilderness|id=arrival_location_type]]
```