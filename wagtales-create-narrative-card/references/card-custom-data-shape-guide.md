# Card CustomDataShape Guide

Treat this file as the contract for field-level prompt routing and runtime visibility on authored custom fields inside a single narrative-style card object.

Use this when:

- authoring `customDataShape` on a narrative-style card
- generating card fields that should appear only for certain prompt targets
- adding runtime visibility gates to custom card fields

## Runtime Model

Two filter systems control authored custom fields:

1. `promptIncludeMask`
2. `visibilityConditions`

They are AND-ed.

A custom field is included only if:

- its `promptIncludeMask` allows the current prompt target
- its `visibilityConditions` pass against live game state

Invalid or unresolvable visibility logic fails closed and omits the field.

## Where Filters Live

Store filter metadata under the card's `customDataShape`, using the same key as the authored custom field.

Example:

```json
{
  "id": "gothic_pressure",
  "mystery_reveal_style": "withhold causal explanation until emotional stakes are attached",
  "customDataShape": {
    "mystery_reveal_style": {
      "promptIncludeMask": 2,
      "visibilityConditions": {
        "currentTurn": {
          ">=": 2
        }
      }
    }
  }
}
```

Filters target authored custom fields. Do not treat `customDataShape` as story content, and do not expect it to hide built-in fields such as `id`, `title`, `description`, `representative_authors`, `technical_guide`, `example`, or `blacklist`.

## Prompt Bitmask Reference

`promptIncludeMask` is a bitmask.

| Prompt Target | Value |
| --- | ---: |
| `narrativeTurn0` | `1` |
| `narrative` | `2` |
| `narrativeLastTurn` | `4` |
| `update` | `8` |
| `gameInit` | `16` |
| `choiceMetadata` | `32` |
| `dialog` | `64` |
| `actTransitionPruning` | `128` |

Mask calculation is additive.

Examples:

- narrative only: `2`
- narrative + dialog: `66`
- gameInit + update: `24`
- turn0 + narrative + lastTurn: `7`

Defaults:

- omit `promptIncludeMask` to allow all prompt families
- `0` is technically valid but blocks every prompt family

## Visibility Syntax

`visibilityConditions` is a map from state path to either:

- a direct scalar, treated as equality
- an operator object

Example:

```json
{
  "visibilityConditions": {
    "immutable.act": {
      ">=": 2
    },
    "location": {
      "!=": "safehouse"
    }
  }
}
```

All visibility conditions are AND-ed. There is no OR logic.

## Allowed Path Families

Supported path families:

- `player.*`
- `world.*`
- `location`
- `currentTurn`
- `immutable.act`
- `time.day`
- `time`
- supported `.count` paths

Use explicit full paths only.

## Authoring Rules

- Put filter settings under `customDataShape`, never inline in ordinary prose fields.
- Use the same key in `customDataShape` as the governed custom field.
- Omit `promptIncludeMask` if the field should be available everywhere.
- Use only explicit paths such as `player.x`, `world.npcs.id.field`, `location`, `time.day`, `time`, `currentTurn`, or `immutable.act`.
- Assume all visibility conditions are AND-ed.
- Prefer simple, explicit conditions.

## Common Errors

- adding a `customDataShape` entry for a field that is not actually authored on the card
- expecting `promptIncludeMask: 2` to include dialog prompts
- using shorthand paths such as `npcs.guard.trust`
- trying to use `customDataShape` to hide built-in card fields