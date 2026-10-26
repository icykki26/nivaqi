# 0015 — Stored money is an integer count of the minor unit, on both engines

**Status:** accepted, 2026-10-26, recorded on the owner's instruction.
Settles the money representation left open in [0013](0013-edge-placement.md)
and [0014](0014-databases-and-language.md), and replaces ARCHITECTURE section
6.4, rule 6.

## Context

SQLite is the edge database and PostgreSQL the cloud database (0014). The
earlier rule stored money as `NUMERIC` on PostgreSQL and as text on SQLite, on
the grounds that text "round-trips losslessly". That is true of storage and
false of querying.

I first framed the problem as a `SUM` problem (0013). It is wider. SQLite
compares text lexicographically, so a text money column sorts, compares and
takes `MAX` wrongly. `examples/katY` declares its money column as `TEXT`, so
this is what the example would do.

Tested in SQLite 3.50.4 through Python, on four amounts (9.50, 10.00, 100.00,
2.25):

| | TEXT column | Integer minor units |
|---|---|---|
| `ORDER BY` | 10.00, 100.00, 2.25, 9.50 — wrong | correct |
| `WHERE a > 9.60` | nothing; 10.00 and 100.00 should qualify | correct |
| `MAX` | 9.50; should be 100.00 | correct |
| `SUM` over random ledgers | differed from the exact decimal sum in 44 of 1,500 small-ledger trials (0013) | debit equalled credit in all 1,800 trials; integer addition is exact by construction |

**Caveat:** this is SQLite through Python, not eTamil's bundled SQLite
(rusqlite 0.32, `bundled`) through its driver. Text comparison is standard
SQLite behaviour, so the ordering result is not in doubt, but the driver
round-trip is untested and is a spike item.

## Decision

**Every stored money amount is an integer count of the currency's minor unit** —
`BIGINT` on PostgreSQL, `INTEGER` on SQLite — never `NUMERIC`, never text,
never a float.

- **One representation on both engines.** Sums, ordering, comparison, `MAX` and
  indexes are exact and correct everywhere. A tenant's data moves between the
  pooled tier, a silo and an edge install unchanged, and the export files are
  identical whichever engine wrote them (0011, 0013).
- **In memory the ledger still works in exact decimals**, as every eTamil number
  does. The conversion happens once, in the persistence layer
  (`atiqqaLam/paqivEtu`), never in a module.
- **No implicit rounding at the boundary.** An amount with more precision than
  the currency's minor unit is refused, loudly, rather than stored rounded. The
  caller rounds explicitly, once, at the end of a calculation. This follows the
  project's rule that failures are loud (`eTamil_lang/docs/ARCHITECTURE.md`).
- **What counts as money:** ledger lines, document and line totals, payments,
  balances and valuation amounts.
- **Rates stay exact decimals stored as text**: tax and discount percentages and
  exchange rates. They are looked up by effective date and are never summed or
  ordered in SQL.
- **Range:** a signed 64-bit integer holds about 9.2 × 10¹⁸ minor units, which
  is about ₹9.2 × 10¹⁶. SQLite raises an error on integer overflow in `SUM`
  rather than wrapping, which is the failure to want.

## Alternatives not taken

| Alternative | Why not |
|---|---|
| `NUMERIC` on PostgreSQL and text on SQLite (the earlier rule 6) | Two representations, and the edge sorts and compares wrongly |
| A decimal aggregate and collation in the SQLite driver | Fixes sums and ordering for text columns only through a per-engine shim, and indexes still depend on the collation. Kept as a fallback for non-money decimals; the driver already enables rusqlite's `functions` feature, so it would be small |
| Floats | Not exact |
| A purpose-built database for money | Rejected under 0014's exception test: a storage engine is the huge case, and Layer 0 |

## What this adds

- **A minor-unit exponent per currency.** INR has 2, JPY 0, KWD 3.
  `nUlakam/paNam/kAcu.qmz` is paise-only: its comments say "two decimal places,
  exactly". I searched `kaNakkiyal/niRuvaZam.qmz` for minor-unit terms and found
  no per-currency table. So `paNam` needs a currency-aware sibling, which is a
  gap to fill in `eTamil_lang` under the 0014 policy. Currency exponents are
  static ISO 4217 data, loaded from `payiRci/` with their source.
- **Rounding rules written down:** half-up once at the end, as `kAcu.qmz`
  already does for percentages and splits (`விழுக்காடு_காசு`,
  `சமமாகப்_பிரி`), so a total matches the sum of its lines.
- **A column naming convention** that marks money columns. The name is the
  owner's to choose.

## Consequences

- The `examples/katY` and `examples/aluvalakam` schemas store money as text or
  inside JSON. They are not changed here and are not to be copied.
- The earlier `SUM` recommendation in 0013 is now a decision, and its "open"
  items are closed.
- Money columns are indexable and sortable, which `vatikattal` needs for list
  screens that filter and sort by amount.

## Not measured

- Integer columns round-trip through eTamil's driver as `INTEGER` and not as
  text. The driver sends decimals as text and relies on column affinity, which
  should coerce well-formed integer text, but I have not tested it.
- `SUM` through the driver returns an exact integer, and on PostgreSQL the
  `BIGINT` sum comes back without loss.

Both join the M0 spike (ARCHITECTURE section 15).

## Open

- **Quantities have the same problem.** Stock on hand, reorder rules and
  valuation need to be summed and compared in SQL, and the dry fruit store and
  the pharmacy sell by weight and by pack. The recommendation, not a decision,
  is the same approach: an integer in the unit category's reference unit at a
  defined scale (ARCHITECTURE section 6.4, rule 8). It must be settled before
  the inventory schema, not before M0.
- The money column naming convention.
- Whether the currency-aware money library extends `paNam/kAcu.qmz` or sits
  beside it.
