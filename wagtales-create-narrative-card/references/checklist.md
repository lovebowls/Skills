# Checklist

Use this checklist before returning the final output.

- The artifact is a single JSON object.
- The output is not wrapped in `narrative_style`, `story_arc`, `deckId`, or `cards`.
- The object includes `id`.
- Standard fields are present only when useful.
- Any extra custom fields materially help express the author's intent.
- Every `customDataShape` key matches a real authored field on the card.
- Any `semanticQueries` are scalar, valid, and attached only to authored custom fields.
- `technical_guide`, `example`, and `blacklist` do not contradict one another.
- The card describes a coherent, distinctive writing mode.