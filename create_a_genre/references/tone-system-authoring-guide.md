# Tone System Authoring Guide

Use `tone` to encode the emotional baseline of a playable NPC in compact `HHH:SSS:LLL` format.

`tone` is required for every NPC with `playable: true`.

Non-playable NPCs may also include `tone`, but it is optional and should only be authored when it gives useful emotional guidance.

## Format

- `HHH` = hue, the character's core emotional register
- `SSS` = saturation, the intensity of that emotion
- `LLL` = lightness, the brightness or darkness of the outlook

Use three digits for each component.

Examples:

- `000:100:025`
- `120:090:070`
- `180:050:050`
- `240:080:033`

## Hue Reference

- `000`: aggressive
- `030`: urgent
- `060`: passionate
- `090`: commanding
- `120`: optimistic
- `150`: analytical
- `180`: calm
- `210`: detached
- `240`: melancholy
- `270`: mysterious
- `300`: dramatic
- `330`: obsessive

## Saturation Guidance

- `020-040`: restrained, muted, emotionally controlled
- `050-070`: clear but moderate emotional intensity
- `080-100`: strong, vivid, forceful emotional presence

## Lightness Guidance

- `000-030`: dark, cynical, ominous, or harsh
- `031-069`: balanced, grounded, emotionally readable
- `070-100`: bright, hopeful, open, or idealistic

## How To Choose A Tone

Choose tone based on the character's role, emotional posture, and the genre's atmosphere.

- hard-boiled investigator: `240:080:030`
- idealistic rescuer: `120:090:070`
- severe commander: `090:070:040`
- cold bureaucrat: `210:040:045`
- haunted occultist: `270:060:030`
- driven surgeon under pressure: `030:085:045`

## Authoring Rules

- Every playable NPC must have `tone`.
- Tone should fit the character, not just the setting.
- Do not assign identical tones to every major NPC unless that sameness is deliberate.
- Use darker lightness values for grim or hostile playable characters.
- Use brighter lightness values for hopeful or open-hearted playable characters.
- High saturation suits characters who act intensely or wear emotion openly.
- Lower saturation suits guarded, stoic, or emotionally muted characters.

## Practical Checks

Before finalizing a playable NPC, ask:

1. Does the hue match the character's dominant emotional register?
2. Does saturation fit how intense or restrained they are?
3. Does lightness fit whether they feel hopeful, balanced, or dark?
4. Would this tone help downstream narration stay emotionally coherent?