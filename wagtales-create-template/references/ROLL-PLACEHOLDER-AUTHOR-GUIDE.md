# Roll Placeholder Spec

Treat this file as the contract for authored randomization tokens inside prompt-facing Wagtales text.

Use this when:

- authoring `Authors_note`, `ai_instructions`, or other prompt-facing text
- instructing Claude to generate deterministic in-prompt randomization
- replacing legacy slot-based roll syntax

## Runtime Model

Roll placeholders are resolved server-side before prompt submission.

The model never performs the roll math and never sees the placeholder syntax when resolution succeeds. It sees only the resolved text value.

This gives deterministic randomization within a single prompt and keeps authored instructions literal.

## Supported Syntax

```text
[[ROLL:dN]]
[[ROLL:dN|id=some_name]]
[[ROLL:dN|min=X|max=Y]]
[[ROLL:dN|choices=a,b,c]]
[[ROLL:dN|choicesArray=["a","b","c"]]]
[[ROLL|choices=a,b,c]]
[[ROLL|choicesArray=["a","b","c"]]]
```

Examples:

```text
[[ROLL:d6]]
[[ROLL:d100|min=-1500|max=2000|id=arrival_year]]
[[ROLL:d4|choices=City,Town,Countryside,Wilderness|id=arrival_location_type]]
[[ROLL|choices=City,Town,Countryside,Wilderness]]
[[ROLL|choicesArray=["Anything, including commas","Another long value"]]]
```

## Option Semantics

### Raw roll

```text
[[ROLL:d6]]
```

Emits the rolled integer.

### Stable reused roll

```text
[[ROLL:d6|id=my_roll]]
```

Reusing the same `id` within one prompt reuses the same underlying roll. Reusing the same `id` with a different die size is invalid.

### Numeric range mapping

```text
[[ROLL:d100|min=-1500|max=2000|id=arrival_year]]
```

Rules:

- both `min` and `max` are required
- mapping is linear from `1..dN` into inclusive `min..max`
- result is rounded to the nearest integer
- `min` may be negative
- `min` may equal `max`
- `min` cannot be greater than `max`

### Ordered choice mapping

```text
[[ROLL:d4|choices=City,Town,Countryside,Wilderness|id=arrival_location_type]]
[[ROLL|choices=City,Town,Countryside,Wilderness|id=arrival_location_type]]
```

Rules:

- roll `1` selects the first choice, roll `2` the second, and so on
- with an explicit die, the number of choices must exactly match the die size
- without a die, the backend infers the die size from the number of choices
- choices are comma-separated and trimmed
- use `choices` only for simple values that do not contain commas

### Structured choice mapping

```text
[[ROLL|id=disaster_type|choicesArray=[
	"supervolcano: A Yellowstone or Campi Flegrei-scale eruption has occurred, producing ash fall across the hemisphere.",
	"solar_storm: A Carrington-class geomagnetic event has struck, disabling satellites and power systems.",
	"engineered_pandemic: A novel engineered pathogen has been confirmed spreading in multiple countries, with government responses lagging."
]]]
```

Rules:

- `choicesArray` must be valid JSON
- `choicesArray` must parse to a non-empty array
- every array entry must be a string
- with an explicit die, the array length must exactly match the die size
- without a die, the backend infers the die size from the array length
- commas inside a string are allowed
- embedded double quotes inside a string must be escaped as `\"`
- `choices` and `choicesArray` are mutually exclusive

## Typical Authoring Patterns

Raw count:

```text
The player may choose [[ROLL:d4|id=item_count]] carriable objects.
```

Mapped year:

```text
Set the arrival year to [[ROLL:d100|min=-1500|max=2000|id=arrival_year]].
```

Mapped category:

```text
Use this settlement type: [[ROLL:d4|choices=City,Town,Countryside,Wilderness|id=arrival_location_type]].
```

Mapped structured prose:

```text
Use this disaster seed: [[ROLL|id=disaster_type|choicesArray=["Anything, including commas","A longer authored value","A value with \"quotes\" inside"]]].
```

Reuse one roll in two forms:

```text
The destination category roll is [[ROLL:d4|id=destination_roll]].
Interpret that same roll as [[ROLL:d4|choices=City,Town,Countryside,Wilderness|id=destination_roll]].
```

Reuse one roll with a structured mapping:

```text
The severity bucket is [[ROLL|id=severity|choices=low,medium,high]].
Interpret that same roll as [[ROLL|id=severity|choicesArray=["Minor disruption","Major instability","Systemic collapse"]]].
```

## Legacy Syntax

The old slot-based syntax is unsupported.

Invalid examples:

```text
[[ROLL:d100:1]]
[[ROLL:d4:1]]
```

Do not author new content with slot numbers.

## Validation Rules

A placeholder is invalid if any of the following are true:

- the die spec is missing or malformed
- the die size is not a positive integer
- an unsupported option key is used
- the same option is provided more than once
- `min` is present without `max`
- `max` is present without `min`
- `min > max`
- `choices` is empty
- `choicesArray` is not valid JSON
- `choicesArray` is not a non-empty array of strings
- the number of `choices` does not equal the die size when a die is explicit
- the number of `choicesArray` entries does not equal the die size when a die is explicit
- `choices` is combined with `min` or `max`
- `choicesArray` is combined with `min` or `max`
- `choices` and `choicesArray` are both present
- the same `id` is reused with a different die size

Supported option keys are only:

- `id`
- `min`
- `max`
- `choices`
- `choicesArray`

## Failure Behavior

If a placeholder is invalid:

- the backend logs a warning
- the original placeholder text is preserved unchanged
- prompt compilation continues

Example:

```text
[[ROLL:d100:1]]
```

remains literal instead of being resolved.

## Authoring Rules For Claude

- Treat resolved values as facts, not suggestions.
- Write prompt text so the model uses the resolved value directly.
- Use `id` whenever multiple prompt fragments must stay consistent.
- Use `choices` for named categories instead of prose mapping instructions.
- Use `choicesArray` for values that may contain commas, embedded quotes, or multiline prose.
- Use `min` and `max` for bounded numeric values.
- Do not ask the model to perform roll arithmetic itself.
- Do not use legacy slot syntax.
