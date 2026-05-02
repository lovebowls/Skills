# Card Semantic Query Guide

Treat this file as a contract reference for semantic-query-gated custom fields on a single narrative-style card object.

Use this when:

- authoring `semanticQueries` under card `customDataShape`
- generating card fields that should appear only when semantic answers pass

## Runtime Model

A custom authored card field may be gated by up to three systems:

1. `promptIncludeMask`
2. `visibilityConditions`
3. `semanticQueries`

All active gates are AND-ed.

A field is included only if:

- its `promptIncludeMask` allows the current prompt target
- its `visibilityConditions` pass against live game state
- every semantic query on that field passes against `session.semanticQueryAnswers`

Semantic evaluation fails closed.

## Where It Lives

Semantic queries live under the card's `customDataShape`, using the same key as the authored custom field they govern.

Example:

```json
{
  "id": "romantic_heat",
  "romance_intensity": "allow tenderness to stay charged and a little dangerous",
  "customDataShape": {
    "romance_intensity": {
      "semanticQueries": [
        {
          "id": "romance_open",
          "question": "Is romantic tension currently active between the focal characters?",
          "type": "boolean",
          "operator": "==",
          "expectedValue": true
        }
      ]
    }
  }
}
```

Semantic queries affect only authored custom fields. Do not use them as if they can hide built-in structural fields.

## Query Shape

Each semantic query must contain:

- `id`
- `question`
- `type`
- `operator`
- `expectedValue`

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

Numeric comparison operators require `type: "number"`.

## Authoring Rules

- Attach semantic queries only to authored custom fields.
- Use `semanticQueries` as an array even for one query.
- Prefer short stable ids.
- Write questions that produce one scalar answer, not prose.
- Prefer boolean queries when possible.
- Use number queries for thresholds.
- Use string queries only for stable categorical labels.

## Common Errors

- adding semantic queries to a field that is not authored on the card
- writing open-ended questions that do not yield a scalar answer
- using numeric operators with non-number query types