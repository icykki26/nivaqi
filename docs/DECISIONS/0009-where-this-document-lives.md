# 0009 — Where the architecture document lives

**Status:** accepted, 2026-10-24.

## Context

The document was drafted in `eTamil_lang/docs/nivaqi/NIVAQI.md`, where it was a
statement about what eTamil can support. Once there is code, it becomes the
application's own architecture document.

## Decision

It lives in the nivaqi repository, at `docs/ARCHITECTURE.md` —
`github.com/icykki26/nivaqi`, and `D:\src\nivaqi` locally. The owner pushes.

`eTamil_lang/docs/INDEX.md` keeps a one-line pointer to it, recording that
nivaqi is the Phase 4 pilot the roadmap has been waiting for. The copy drafted
there is removed, because two copies is the rot the eTamil documentation index
already warns about.

## Consequences

- The document is maintained beside the code it describes.
- Changes to it are reviewed with the code that motivated them.
- `eTamil_lang` keeps a pointer, not a copy.
