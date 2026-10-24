# 0007 — Four pilots, staggered, MNC first

**Status:** accepted, 2026-10-24.

## Context

Four businesses are ready to adopt and test: a pharmacy, a dry fruit retail
store, an IT consulting MNC, and a bakery restaurant. They are the most
important input to the architecture, because between them they demand
considerably more than the original module list covered — and they do not
demand it in the same order.

## Decision

**Go live one pilot per wave, in this order:**

| Wave | Pilot | Why here |
|---|---|---|
| 1 | **IT consulting MNC** | The only one needing neither Inventory nor PoS, and the one needing the least new library code |
| 2 | **Dry fruit retail** | Adds Inventory, Sales & Purchase, retail PoS and weight units |
| 3 | **Pharmacy** | Adds batch, expiry and FEFO on proven inventory, plus regulated items |
| 4 | **Bakery restaurant** | Adds manufacturing and restaurant-mode PoS — the most new ground |

**Employee and Timesheet move into wave 1**, ahead of Inventory and PoS,
because the MNC needs them and nothing else does.

## Consequences

- **Time to the first real user drops to about 31–39 weeks**, from 48–60 under
  the original order. A books-only go-live for the MNC is about half that.
- **Three new library families** the original plan did not have: batch and
  expiry with FEFO, weight-based units, and manufacturing. Plus a small
  regulated-item family for the pharmacy.
- Total effort rises to about 92–118 weeks, because the pilots added real
  scope. That is the honest trade: more work, a real user far sooner.

## Two warnings

**The requirements in ARCHITECTURE section 2 are inferred from the business
types, not gathered.** Each needs confirming with the business before its wave
starts. What holds regardless of the details is that the four are not
interchangeable.

**One developer cannot build the next wave and support four live pilots.** A
live pilot generates support, data corrections and change requests that compete
directly with building, and they win, because a business that has stopped
invoicing does not wait. Stagger the go-lives and reserve **20–30% of
capacity** for each pilot already live. If all four must go live close
together, that is a second person, not a longer timeline.
