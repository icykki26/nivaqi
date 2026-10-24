# 0003 — Sales & Purchase is a first-class module

**Status:** accepted, 2026-10-24.

## Context

The original module list had Accounting, Partners, Inventory, Identity and PoS,
with no module owning the order-to-invoice chain.

## Decision

Sales & Purchase (`vaNikam`, வணிகம்) is a module in its own right, placed
after Inventory and before PoS.

## Rationale

Not that an ERP needs sales orders — that **the sales order has nowhere else to
live**:

- Put it in Accounting, and Accounting starts reserving stock, which breaks the
  one property that makes the ledger trustworthy.
- Put it in Inventory, and Inventory starts owning price lists, discounts and
  credit limits.
- The three-way match (order / receipt / bill) spans Partners, Inventory and
  Accounting and belongs to none of them.
- Gross margin — the number an owner looks at every morning — is computed from
  a line no other module owns.

## Consequences

- Nominally 6–8 weeks, but it removes the order chain from M3 and M4, so the
  net cost is nearer 3–4.
- Document lifecycles for all trade documents live in one place, on
  `qittam/nilYmARRam`.
