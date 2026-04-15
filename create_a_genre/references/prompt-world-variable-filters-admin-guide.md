# Prompt And World Variable Filters Admin Guide

This guide documents the filtering system used to control when custom authored fields are visible to AI prompts.

Use this when:

- authoring genre metadata with custom prompt filters
- authoring custom NPC or location fields with runtime visibility rules
- instructing another AI to generate `init.json` content that already includes valid filters

This document reflects the current implementation in shared types, frontend authoring UI, and middle-layer runtime filtering.

## Mental Model

There are two separate filters:

1. `promptIncludeMask`
   Controls which prompt families are allowed to receive a field.
2. `visibilityConditions`
   Controls whether the field is included at runtime based on current game state.

Both filters apply with logical AND.

A field is included only if:

- its `promptIncludeMask` allows the current prompt target, and
- its `visibilityConditions` pass against the active game state

If a visibility rule is invalid or cannot be resolved, the system fails closed and omits the field.

## Where Filters Live

### Genre-level metadata custom fields

Store settings under:

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

### Entity-level custom fields on locations and NPCs

Store settings on the owning entity's `customDataShape`.

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

Important:

- Filters only affect custom authored fields.
- Core structural fields such as `id`, `name`, and `description` are not the main target for this system.
- `customDataShape` itself is internal metadata and must not be written as ordinary story content.

## Prompt Bitmask Reference

`promptIncludeMask` is a bitmask. Each prompt target has a power-of-two value.

| Prompt Target | Value | Meaning |
| --- | ---: | --- |
| `narrativeTurn0` | `1` | Opening/prologue narrative generation |
| `narrative` | `2` | Standard turn narrative generation |
| `narrativeLastTurn` | `4` | Ending/epilogue narrative generation |
| `update` | `8` | Update-script generation |
| `gameInit` | `16` | Initial generation prompt |
| `choiceMetadata` | `32` | Choice analysis/enrichment |
| `dialog` | `64` | NPC and narrator dialog prompts |
| `actTransitionPruning` | `128` | Pre-transition pruning/consolidation |

### How To Calculate A Mask

Add the values for every prompt family you want.

Examples:

- Narrative only: `2`
- Narrative plus dialog: `2 + 64 = 66`
- Game init plus update: `16 + 8 = 24`
- Prologue plus standard narrative plus epilogue: `1 + 2 + 4 = 7`

### Important Defaults

- If `promptIncludeMask` is omitted, the field is treated as available to all prompt families.
- A mask of `0` is technically valid in backend schema but functionally means no prompt family is allowed. The frontend authoring UI treats this as an error state for normal authoring.
- The shared type surface includes `choiceMetadata = 32`, but the current frontend selector does not expose that option in the prompt target picker.

## Visibility Conditions Reference

`visibilityConditions` is a map keyed by state path.

Each entry may be either:

- a direct scalar value, which behaves like equality
- an operator object

Examples:

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

The three conditions above are all required. V1 is AND-only.

## Allowed Visibility Paths

The runtime supports these path families:

- `player.*`
- `world.*`
- `location`
- `currentTurn`
- `immutable.act`
- `time.day`
- `time`
- supported `.count` paths

### Notes On Path Rules

- Visibility paths must be explicit. Shorthand forms such as `npcs.sable.mood` or `sable.mood` are not valid here.
- `location` means the current location id stored on the root game state.
- `time` is evaluated as minutes after midnight.
- `.count` is supported for:
  - `world.locations.count`
  - `world.npcs.count`
  - custom collection fields on a specific NPC or location, for example `world.locations.harbor.crates.count`

### Examples Of Valid Paths

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

## Operator Rules

The system does not allow every operator on every type.

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

### Existence Semantics

Use:

```json
{
  "world.npcs.sable.secret": {
    "exists": true
  }
}
```

This passes when the resolved value is neither `undefined` nor `null`.

The frontend UI may display `!exists` as an authoring affordance, but the stored JSON shape should still be written using `exists: false` when authoring directly in data.

Example:

```json
{
  "world.locations.harbor.smuggling_cache": {
    "exists": false
  }
}
```

## Evaluation Semantics

The runtime evaluates every `visibilityConditions` entry against the active game state.

Rules:

- All conditions must pass.
- Invalid paths fail closed.
- Unsupported operators fail closed.
- Missing game state fails closed for any field that requires visibility evaluation.
- Omitted `visibilityConditions` means no runtime restriction.

This means direct authoring should stay conservative. If you are not sure a path will exist at runtime, prefer `exists` checks or omit the condition.

## Authoring Patterns

### 1. Narrative-only metadata field

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

### 2. Dialog-only NPC custom field

```json
{
  "world": {
    "npcs": {
      "sable": {
        "id": "sable",
        "name": "Sable",
        "rumor_style": "Answers indirectly, as if testing whether the player deserves the truth.",
        "customDataShape": {
          "rumor_style": {
            "promptIncludeMask": 64
          }
        }
      }
    }
  }
}
```

### 3. Field shown only after day 3

```json
{
  "customDataShape": {
    "storm_memory": {
      "visibilityConditions": {
        "time.day": {
          ">=": 3
        }
      }
    }
  }
}
```

### 4. Field shown only in one location

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

### 5. Field shown when a world property exists

```json
{
  "customDataShape": {
    "hidden_passage_hint": {
      "visibilityConditions": {
        "world.locations.catacombs.hidden_passage": {
          "exists": true
        }
      }
    }
  }
}
```

### 6. Combined prompt and runtime filtering

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

## What Another AI Should And Should Not Do

### Safe authoring rules

If you are instructing another AI to generate filter-aware genre content, give it these rules:

1. Put filtering settings inside `customDataShape`, never mixed into ordinary prose fields.
2. Use `promptIncludeMask` only when a field should be limited to specific prompt families.
3. Omit `promptIncludeMask` when the field should be available everywhere.
4. Use only explicit visibility paths such as `player.x`, `world.npcs.id.field`, `location`, `time.day`, `time`, `currentTurn`, or `immutable.act`.
5. Use only supported operators for the resolved value type.
6. Assume all visibility conditions are AND-ed together.
7. Prefer `exists` checks when runtime shape may vary.
8. Do not use shorthand paths.
9. Do not invent OR groups, nested boolean logic, or expression strings.
10. Do not try to filter built-in entity fields with `customDataShape`.

### AI-ready instruction block

Use the block below when prompting another AI:

```text
When generating Wagtales genre JSON, you may attach prompt/world filters to custom metadata, NPC, or location fields.

Rules:
- Store filter settings under customDataShape using the same key as the authored field.
- promptIncludeMask is a bitmask. Use these values: narrativeTurn0=1, narrative=2, narrativeLastTurn=4, update=8, gameInit=16, choiceMetadata=32, dialog=64, actTransitionPruning=128.
- Omit promptIncludeMask if the field should be available to all prompt families.
- visibilityConditions is a map of explicit state paths to either a scalar equality value or an operator object.
- Allowed path families: player.*, world.*, location, currentTurn, immutable.act, time.day, time, and supported .count paths.
- Use only supported operators: number => exists, ==, !=, >, >=, <, <=; string/boolean => exists, ==, !=; array/object => exists; location => ==, !=; time => >, <.
- All visibilityConditions are AND-ed.
- Invalid or unresolvable visibility conditions fail closed, so do not invent paths or operators.
- Do not use shorthand paths like npcs.id.field or bare entity ids.
- Do not invent OR groups or expression strings.

When you output a filtered field, include both the field value and its matching customDataShape entry.
```

## Common Mistakes

- Using `promptIncludeMask: 2` and assuming it also affects dialog prompts. It does not.
- Using shorthand visibility paths such as `npcs.sable.trust` instead of `world.npcs.sable.trust`.
- Using `>` on a string field.
- Using `==` on an object or array field.
- Writing `!exists` directly into saved JSON instead of `"exists": false`.
- Expecting OR logic. There is no OR grouping in V1.
- Adding filters to standard entity fields and expecting them to hide built-in fields.

## Practical Recommendation

For most genre authoring, use this hierarchy:

1. Omit filtering unless a field clearly should be restricted.
2. Use `promptIncludeMask` first when the distinction is about prompt family.
3. Add `visibilityConditions` only when the distinction truly depends on live state.
4. Keep conditions simple and explicit so another AI can reproduce them reliably.