# Semantic Query Spec

Treat this file as a contract reference for authoring semantic-query-gated custom fields in Wagtales JSON.

Use this when:

- authoring `semanticQueries` under `customDataShape`
- instructing Claude to generate semantic-query-aware `init.json` content
- debugging why a custom field is missing from a prompt

## Runtime Model

A custom authored field may be gated by up to three systems:

1. `promptIncludeMask`
2. `visibilityConditions`
3. `semanticQueries`

All active gates are AND-ed.

A field is included only if:

- its `promptIncludeMask` allows the current prompt target
- its `visibilityConditions` pass against live game state
- every semantic query on that field passes against `session.semanticQueryAnswers`

Semantic evaluation fails closed. Missing answers, type mismatches, invalid config, or unknown ids all cause the field to be omitted.

## Where It Lives

Semantic queries live under the owning object's `customDataShape`, using the same key as the authored field they govern.

Supported hosts:

- `metadata.customDataShape`
- `world.npcs.{id}.customDataShape`
- `world.locations.{id}.customDataShape`

Semantic queries affect only authored custom fields. Do not use them as if they can hide built-in structural fields such as `name`, `description`, `tone`, `image`, or `playable`.

## Authored Shape

```json
{
  "customDataShape": {
    "ravenously_hungry": {
      "semanticQueries": [
        {
          "id": "ca7cd041",
          "question": "Is the player hungry?",
          "type": "boolean",
          "operator": "==",
          "expectedValue": true
        }
      ]
    }
  }
}
```

Each semantic query must contain:

- `id`: stable runtime key, max 100 chars
- `question`: plain-language scalar question, max 300 chars
- `type`: `boolean`, `number`, or `string`
- `operator`: comparison used during runtime filtering
- `expectedValue`: comparison target matching `type`

## Prompting Behavior

Semantic queries are asked only during update-script generation.

The backend injects each active query into the update prompt as a reduced structure:

```json
[
  {
    "id": "ca7cd041",
    "question": "Is the player hungry?",
    "type": "boolean",
    "acceptedResponseValue": true
  }
]
```

The model does not receive:

- `operator`
- `expectedValue`
- `hostPath`
- `fieldKey`

The backend performs the real comparison later.

## Query Collection Rule

When deciding whether to ask a semantic question during update generation:

- `visibilityConditions` are respected
- `promptIncludeMask` is ignored

This is intentional. A field may be excluded from the update prompt body but still contribute a semantic question whose answer controls later prompt inclusion.

## Runtime Storage And Script Op

Answers are stored at:

```json
{
  "session": {
    "semanticQueryAnswers": {
      "ca7cd041": false
    }
  }
}
```

The update script writes answers with:

```json
{
  "op": "semanticQuery.answer",
  "target": "ca7cd041",
  "value": false
}
```

Rules:

- `target` must match an authored semantic query id exactly
- `value` must match the authored `type`
- unknown ids fail
- type mismatches fail

## Multi-Query Semantics

`semanticQueries` is always an array.

If a field has multiple queries, they are AND-ed. There is no OR grouping, nesting, or composite expression syntax.

## Types And Operators

Supported types:

- `boolean`
- `number`
- `string`

Supported operators:

- `==`
- `!=`
- `>`
- `>=`
- `<`
- `<=`

Practical restrictions:

- `boolean`: use `==` or `!=`
- `string`: use `==` or `!=`
- `number`: may use all six operators
- `>`, `>=`, `<`, `<=` require `type: "number"`

## Validation Rules

Treat the query as invalid if any of the following are true:

- `id` is missing or empty
- `id` exceeds 100 chars
- `question` is missing or empty
- `question` exceeds 300 chars
- `type` is not `boolean`, `number`, or `string`
- `operator` is unsupported
- `expectedValue` does not match `type`
- a numeric-only operator is used with a non-number type

Runtime inclusion also fails closed when:

- no stored answer exists
- the stored answer type is wrong
- the emitted `semanticQuery.answer` id does not exist in authored data

## Authoring Rules For Claude

- Attach semantic queries only to authored custom fields.
- Put them under `customDataShape` using the same field key as the governed field.
- Use `semanticQueries` as an array even for one query.
- Prefer short opaque ids such as `ca7cd041`.
- Write questions that produce one scalar answer, not prose.
- Prefer boolean queries when possible.
- Use number queries for thresholds.
- Use string queries only for stable categorical labels.
- Do not invent arrays, objects, nested logic, or OR semantics.

## Minimal Patterns

Boolean gate:

```json
{
  "customDataShape": {
    "ravenously_hungry": {
      "semanticQueries": [
        {
          "id": "ca7cd041",
          "question": "Is the player hungry?",
          "type": "boolean",
          "operator": "==",
          "expectedValue": true
        }
      ]
    }
  }
}
```

Numeric threshold gate:

```json
{
  "customDataShape": {
    "village_alarm": {
      "semanticQueries": [
        {
          "id": "alarm_score",
          "question": "How alarmed are nearby villagers by the player on a scale from 0 to 10?",
          "type": "number",
          "operator": ">=",
          "expectedValue": 6
        }
      ]
    }
  }
}
```

String classification gate:

```json
{
  "customDataShape": {
    "merchant_friendly_offer": {
      "semanticQueries": [
        {
          "id": "merchant_mood",
          "question": "What one-word mood best describes the merchant's attitude to the player?",
          "type": "string",
          "operator": "==",
          "expectedValue": "welcoming"
        }
      ]
    }
  }
}
```

## Common Authoring Errors

- using descriptive ids instead of short opaque ids
- writing questions that expect narrative explanation instead of a scalar
- using numeric operators on `boolean` or `string`
- assuming `promptIncludeMask` stops the semantic question from being asked during update generation
- forgetting that multiple semantic queries on one field are all required