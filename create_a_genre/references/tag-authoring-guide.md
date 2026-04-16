# Tag Authoring Guide

Use metadata tags conservatively and only with supported values.

For authored genres, the two most important controlled tags are:

- `genre`
- `age_rating`

## Genre Tag Values

When you add `metadata.tags` with `key: "genre"`, use only values from this permitted list:

- `Action`
- `Adventure`
- `Animation`
- `Comedy`
- `Crime`
- `Documentary`
- `Drama`
- `Family`
- `Fantasy`
- `History`
- `Horror`
- `Music`
- `Mystery`
- `Romance`
- `Science Fiction`
- `TV Movie`
- `Thriller`
- `War`
- `Western`
- `NSFW`

These values match the current product picker: TMDB movie genres plus the additional local tag `NSFW`.

Do not invent bespoke genre labels such as:

- `Legal Thriller`
- `Slice of Life`
- `Cozy Mystery`
- `Psychological Horror`

Instead, express those ideas using the permitted tags in combination.

Examples:

- `Legal Thriller` -> `Drama`, `Thriller`
- `Psychological Horror` -> `Horror`, `Thriller`
- `Romantic Fantasy` -> `Romance`, `Fantasy`
- `Historical Mystery` -> `History`, `Mystery`

## Age Rating Values

When you add `metadata.tags` with `key: "age_rating"`, use exactly one of these values:

- `U`
- `PG`
- `12`
- `12A`
- `15`
- `18`
- `R18`

Do not invent alternatives such as:

- `16`
- `M`
- `Adult`
- `Teen`

## Practical Rules

- Use `genre` tags for discovery/classification, not for prose.
- Use multiple permitted `genre` values when one invented label would otherwise be needed.
- Use at most one `age_rating` value.
- If unsure between two adjacent age ratings, choose the lower intended author target only if the authored material actually supports it.
- If no confident age rating is available at authoring time, prefer omitting it over inventing an unsupported value.