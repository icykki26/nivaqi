# nivaqi — நிவதி

**An ERP written in [eTamil](https://github.com/Maruff/eTamil_lang).**

நிவதி is an acronym of the Tamil for *enterprise resource planning*:
**நி**றுவன (`niRuvaZa`, enterprise) · **வ**ள (`vaLa`, resource) ·
**தி**ட்டமிடல் (`qittamital`, planning). It romanizes to `nivaqi` on-scheme,
and the round trip is exact.

---

## Status

**Nothing is built yet.** This repository currently holds the founding
architecture and the ten decisions it rests on. Code starts with the kernel.

| | |
|---|---|
| Architecture | [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) |
| Decisions | [docs/DECISIONS/](docs/DECISIONS/) |
| Capabilities to build in eTamil | [docs/CAPABILITIES.md](docs/CAPABILITIES.md) |
| Language and libraries | [Maruff/eTamil_lang](https://github.com/Maruff/eTamil_lang) — eTamil 1.4.2 |

## The idea

Most ERPs bolt a ledger onto a transaction system. eTamil's standard library,
`nUlakam`, already contains a double-entry ledger, GST, accounting periods and
close, the three statements, costing, Ind AS 115 revenue recognition, direct
tax, project scheduling and timesheets — **written in eTamil, before any ERP
needed them.**

So nivaqi inverts the usual shape. Every business rule lives in `nUlakam`,
where it is readable, testable without a server, and reusable by the next
module. A nivaqi module does six things and no others: read the request,
authorise it, load records, call `nUlakam`, persist the result, shape the
response.

The measured precedent is `eTamil_lang/examples/aluvalakam`: 223 lines of
application code against 3,029 for the same domain logic written conventionally.

## The pilots

Four businesses are ready to adopt and test, and they drive the build order:

| Wave | Pilot | What it adds |
|---|---|---|
| 1 | IT consulting MNC | Kernel, identity, partners, accounting, employee, timesheet, project billing |
| 2 | Dry fruit retail | Inventory, sales & purchase, offline PoS, weight-based units |
| 3 | Pharmacy | Batch, expiry and FEFO; regulated items |
| 4 | Bakery restaurant | Manufacturing and recipes; restaurant-mode PoS |

The MNC is first because it is the only one needing neither inventory nor a
till, which makes it the shortest path from zero to a real business running on
nivaqi.

## Shape

| Layer | Written in |
|---|---|
| Business rules | **eTamil**, in `nUlakam` |
| Routes, persistence, orchestration | **eTamil**, here |
| Interface | React + Vite + TypeScript, presentation only |
| Database | PostgreSQL in the cloud; SQLite on the edge and in the PoS |
| Backend language | **eTamil by default**; exceptions are recorded ([decision 0014](docs/DECISIONS/0014-databases-and-language.md)) |
| Point of sale | A separate eTamil process on Android, offline-first, over local SQLite |

## Licence and authorship

AGPL-3.0-or-later — see [LICENSE](LICENSE).

Copyright (C) 2026 Isha Maruff and Esan Maruff. See [AUTHORS](AUTHORS).
