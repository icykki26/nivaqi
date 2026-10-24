# 0005 — The PoS is offline-first

**Status:** accepted, 2026-10-24.

## Context

A till that stops selling when the internet drops is not a till. Offline
support cannot be added afterwards: it decides the identifier scheme, the
conflict rules and the sync protocol.

## Decision

The PoS is offline-first, and it is a separate application rather than a screen
in the main one.

Because eTamil has a packaged Android arm64 build and SQLite is in the default
feature set, the terminal runs **a real eTamil process locally against a local
SQLite database, holding the same `nUlakam` rules as the server** — the same
tax arithmetic, the same price resolution, the same receipt numbering — and
synchronises through the outbox pattern `oruwkiNYppu/` already implements.

## Rationale

Most offline PoS implementations reimplement price and tax resolution in the
client and then drift from the server. That is the duplicated-rules problem
the architecture forbids everywhere else. Here it is avoided by construction,
and no other ERP can make the claim.

## Consequences

Three, to be accepted with open eyes:

- Receipt numbers are gapless **per terminal**, not globally.
- Stock at a terminal is advisory until it syncs.
- A line sold twice while two tills were offline needs a **commercial** rule —
  refund, backorder, or honour and go short. That is the owner's decision, not
  the developer's, and it is written into `muZYyam/RULES.md` **before wave 2
  starts**.

Cost: about +3–4 weeks on M6 against an online-only till. A rewrite if deferred.
