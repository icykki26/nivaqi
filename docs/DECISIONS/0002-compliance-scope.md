# 0002 — Compliance scope

**Status:** accepted, 2026-10-24.

## Context

"Accounting" in India is a regulatory commitment whose size depends entirely on
where the line is drawn. Drawing it late is how ERP projects disappear.

## Decision

The line is *does it change the document lifecycle, or is it a report?*

**In scope:**

- Books of account, double entry, the three statements.
- **E-invoicing (IRN and QR).** An invoice is not final until the IRP responds,
  and cancellation has a fixed window — that is a lifecycle change and cannot
  be retrofitted into the state machine cheaply.
- **E-way bills.** Nearly free: `cuwkam/` already rounds validity up, because
  rounding down expires a bill while the lorry is moving.
- **The MCA audit trail (edit log).** Satisfied by the immutable-journal and
  stamped-row rules in ARCHITECTURE section 6.4.
- **TDS computation**, via `nErativari/mUlavari`.
- **GSTR-1 and 3B figures**, summarised from the ledger and exported.

**Out of scope:**

- Filing any return. Filing is a certified product with someone else's
  liability attached.
- GSP integration.
- TDS returns.

## Consequences

- M3 drops from 10–12 weeks to about 8–9.
- The invoice lifecycle carries an IRP state from the first design, not later.
- **No threshold is baked into code.** The e-invoicing turnover threshold has
  moved several times; it lives in an effective-dated table, like every other
  rate (ARCHITECTURE section 6.4, rule 3). Check the current number at
  implementation time, not from a document.
