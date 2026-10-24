# 0008 — Repositories, visibility and licence

**Status:** accepted, 2026-10-24. **One item is time-limited.**

## Context

The original assumption was nivaqi private over public AGPL libraries, which
raised an awkward question: requirement 3 pushes all domain logic into
`nUlakam`, so the commercial value would sit in the public repository and the
private one would hold routing and screens.

## Decision

- **All nivaqi repositories are public.** `icykki26/nivaqi`.
- **Licence: AGPL-3.0-or-later**, matching `eTamil_lang`. nivaqi imports
  `nUlakam` at runtime, both are distributed, and everything is public — so
  AGPL is both consistent and the choice that raises no question.
- **Three products stay private:** AI, BI, and the Customizing Studio.

This resolves the original tension cleanly: there is no private work over AGPL
libraries in nivaqi itself.

## What remains open

**The three private products reintroduce the question.** They will build on
AGPL `nUlakam` and AGPL nivaqi. Whether a private work that imports AGPL
libraries at runtime is itself subject to AGPL is a real legal question that
wants an opinion rather than a developer's guess — **before those products
start, not after.**

## The time-limited item

`eTamil_lang/AUTHORS` records a **sole copyright holder and no contributors
yet**, and states that copyright is not assigned to the project — a contributor
keeps their own and licenses it under AGPL by sign-off.

So **dual-licensing `nUlakam` is entirely the owner's to do today.** That
option closes the moment the first external contribution lands without a
contributor licence agreement in place.

If a commercial licence on the libraries might ever be wanted — and with three
private products planned on top of them, it might — **decide on a CLA before
accepting a contribution.**
