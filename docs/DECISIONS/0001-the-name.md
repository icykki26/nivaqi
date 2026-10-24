# 0001 — The name

**Status:** accepted, 2026-10-24.

## Context

The product needed a name that works in both scripts, because the language's
central claim is that Tamil and its romanization are the same name written two
ways rather than one being a convenience spelling of the other.

## Decision

**நிவதி**, romanized `nivaqi`. It is an acronym of the Tamil for *enterprise
resource planning*:

| | Tamil | ezuqqu | |
|---|---|---|---|
| **ni** | நிறுவன | `niRuvaZa` | enterprise |
| **va** | வள | `vaLa` | resource |
| **qi** | திட்டமிடல் | `qittamital` | planning |

`eTamil_lang/scripts/transliterate.py` confirms நிவதி →
`nivaqi` exactly, and that neighbouring words give different ASCII
(நிவர்த்தி → `nivarqqi`, நிவடி → `nivati`, நிவேதி → `nivEqi`). The round trip
is unique.

## Consequences

- Both spellings are canonical. Neither is a transliteration of the other.
- Repository, package, domain and binary all use `nivaqi`.
- The name is self-explanatory to a Tamil speaker, which is worth more than it
  looks for a product aimed at Tamil-speaking businesses.
