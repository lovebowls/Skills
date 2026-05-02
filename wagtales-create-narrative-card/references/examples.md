# Examples

These examples show the intended output shape for this skill: one narrative-style card object.

## Minimal Card

```json
{
  "id": "spare_clinical_voice",
  "technical_guide": "Use precise, low-affect narration. Favor observable facts before interpretation. Keep emotional spikes sudden and brief.",
  "blacklist": [
    "Sentimental moralizing",
    "Decorative metaphor unrelated to scene pressure"
  ]
}
```

## Standard-Field Heavy Card

```json
{
  "id": "lush_gothic_intimacy",
  "title": "Lush Gothic Intimacy",
  "description": "Ornate but controlled narration that treats desire, dread, and architecture as entangled forces.",
  "representative_authors": [
    "Shirley Jackson",
    "Angela Carter"
  ],
  "technical_guide": "Favor tactile and architectural imagery, but keep syntax controlled. Let desire and fear blur at the edges without losing scene clarity.",
  "example": "The corridor held its cold like a secret it had rehearsed. Even the candlelight seemed to move carefully, as if the house might resent being seen too clearly.",
  "blacklist": [
    "Campy horror phrasing",
    "Modern slang that punctures atmosphere"
  ]
}
```

## Card With CustomDataShape

```json
{
  "id": "investigative_low_heat",
  "title": "Investigative Low Heat",
  "technical_guide": "Keep the prose restrained, skeptical, and detail-led. Suspicion should accumulate through pattern recognition rather than melodrama.",
  "interiority_policy": "Reserve direct interior access for moments where evidence and intuition sharply diverge.",
  "dialogue_temperature": "measured, strategic, lightly adversarial",
  "customDataShape": {
    "interiority_policy": {
      "promptIncludeMask": 2,
      "visibilityConditions": {
        "currentTurn": {
          ">=": 1
        }
      }
    },
    "dialogue_temperature": {
      "promptIncludeMask": 66
    }
  }
}
```

## Card With Semantic Query Gate

```json
{
  "id": "romance_under_pressure",
  "technical_guide": "Keep attraction dangerous, intelligent, and partially suppressed by circumstance.",
  "romance_intensity": "let vulnerability emerge through tactical conversation rather than confession",
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