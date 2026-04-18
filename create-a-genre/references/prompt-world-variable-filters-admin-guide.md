# Prompt And Visibility Filter Spec

Treat this file as the contract for field-level prompt routing and runtime visibility in Wagtales JSON.

Use this when:

- authoring `promptIncludeMask` or `visibilityConditions`
- generating `customDataShape` entries for metadata, NPC, or location fields
- debugging why an authored custom field does or does not appear in a prompt

## Runtime Model

Two filter systems control custom authored fields:

1. `promptIncludeMask`
2. `visibilityConditions`

They are AND-ed.

A field is included only if:

- its `promptIncludeMask` allows the current prompt target
- its `visibilityConditions` pass against live game state

Invalid or unresolvable visibility logic fails closed and omits the field.

## Where Filters Live

Store filter metadata under the owning object's `customDataShape`, using the same key as the authored field.

Metadata example:

```json
{
  "metadata": {
    "weather_pressure": 1008,
    "customDataShape": {
      "weather_pressure": {
        "promptIncludeMask": 2,
        "visibilityConditions": {
          "time.day": {
            ">=": 2
          }
        }
      }
    }
  }
}
```

Location example:

```json
{
  "world": {
    "locations": {
      "harbor": {
        "id": "harbor",
        "name": "Harbor",
        "fog_density": 7,
        "customDataShape": {
          "fog_density": {
            "promptIncludeMask": 2,
            "visibilityConditions": {
              "time.day": {
                ">=": 2
              }
            }
          }
        }
      }
    }
  }
}
```

Filters target authored custom fields. Do not treat `customDataShape` as story content, and do not expect it to hide built-in structural fields such as `id`, `name`, or `description`.

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
- `0` is technically valid but functionally blocks every prompt family
- `choiceMetadata = 32` exists in shared types even if some UI surfaces do not expose it

## Visibility Syntax

`visibilityConditions` is a map from state path to either:

- a direct scalar, treated as equality
- an operator object

Example:

```json
{
  "visibilityConditions": {
    "player.rank": "captain",
    "time.day": {
      ">=": 3
    },
    "world.npcs.sable.alert": true
  }
}
```

All visibility conditions are AND-ed. There is no OR logic or expression language.

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

Notes:

- use explicit full paths only
- `location` means the current location id at the root game state
- `time` means minutes after midnight
- supported `.count` examples include `world.locations.count`, `world.npcs.count`, and custom collection fields such as `world.locations.harbor.crates.count`

Valid examples:

- `player.health`
- `player.flags.has_key`
- `world.npcs.sable.trust`
- `world.locations.harbor.fog_density`
- `world.locations.count`
- `world.npcs.count`
- `location`
- `currentTurn`
- `immutable.act`
- `time.day`
- `time`

## Operator Matrix

| Target Kind | Allowed Operators |
| --- | --- |
| `number` | `exists`, `==`, `!=`, `>`, `>=`, `<`, `<=` |
| `string` | `exists`, `==`, `!=` |
| `boolean` | `exists`, `==`, `!=` |
| `array` | `exists` |
| `object` | `exists` |
| `unknown/unresolvable` | `exists` |
| `location` | `==`, `!=` |
| `currentTurn` | `==`, `!=`, `>`, `>=`, `<`, `<=` |
| `immutable.act` | `==`, `!=`, `>`, `>=`, `<`, `<=` |
| `time.day` | `==`, `!=`, `>`, `>=`, `<`, `<=` |
| `time` | `>`, `<` |

Existence semantics:

```json
{
  "world.npcs.sable.secret": {
    "exists": true
  }
}
```

This passes when the resolved value is neither `undefined` nor `null`.

When authoring stored JSON directly, use `"exists": false` rather than `!exists`.

## Evaluation Rules

- every condition must pass
- invalid paths fail closed
- unsupported operators fail closed
- missing game state fails closed for fields that require visibility evaluation
- omitting `visibilityConditions` means no runtime visibility restriction

Prefer simple explicit conditions. If path existence is uncertain, use `exists` or omit the condition.

## Minimal Patterns

Narrative-only metadata field:

```json
{
  "metadata": {
    "ominous_undertone": "The town behaves as if it has already survived one disaster too many.",
    "customDataShape": {
      "ominous_undertone": {
        "promptIncludeMask": 2
      }
    }
  }
}
```

Dialog-only NPC field:

```json
{
  "customDataShape": {
    "rumor_style": {
      "promptIncludeMask": 64
    }
  }
}
```

Location-gated field:

```json
{
  "customDataShape": {
    "dockside_smell": {
      "visibilityConditions": {
        "location": {
          "==": "harbor"
        }
      }
    }
  }
}
```

Combined prompt and runtime filter:

```json
{
  "customDataShape": {
    "combat_readiness": {
      "promptIncludeMask": 10,
      "visibilityConditions": {
        "player.level": {
          ">=": 5
        },
        "location": {
          "!=": "safehouse"
        }
      }
    }
  }
}
```

`10` means `narrative (2) + update (8)`.

## Authoring Rules For Claude

- Put filter settings under `customDataShape`, never inline inside ordinary prose fields.
- Use the same field key in `customDataShape` as the governed authored field.
- Omit `promptIncludeMask` if the field should be available everywhere.
- Use only explicit paths such as `player.x`, `world.npcs.id.field`, `location`, `time.day`, `time`, `currentTurn`, or `immutable.act`.
- Use only operators valid for the resolved value type.
- Assume all visibility conditions are AND-ed.
- Prefer `exists` when runtime shape may vary.
- Do not invent shorthand paths, OR groups, or expression strings.

## Common Authoring Errors

- assuming `promptIncludeMask: 2` also includes dialog prompts
- using shorthand paths such as `npcs.sable.trust`
- using `>` on a string field
- using `==` on objects or arrays
- writing `!exists` directly into stored JSON
- expecting OR logic or nested boolean expressions