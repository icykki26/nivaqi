# nivaqi — நிவதி

English · [தமிழ்](README.ta.md)

**An ERP written in [eTamil](https://github.com/Maruff/eTamil_lang).**

நிவதி is an acronym of the Tamil for *enterprise resource planning*:
**நி**றுவன (`niRuvaZa`, enterprise) · **வ**ள (`vaLa`, resource) ·
**தி**ட்டமிடல் (`qittamital`, planning). It romanizes to `nivaqi` on-scheme,
and the round trip is exact.

---

## Status

**Nothing is built yet.** This repository currently holds the architecture and the
eighteen decisions it rests on. Code starts with the kernel, which is an eTamil
library and so lives in `nUlakam`; nivaqi holds only the application built from it.

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

## Where code lives

| Kind of code | Where |
|---|---|
| eTamil libraries: the business rules, the kernel, the domain families | `nUlakam`, in [`Maruff/eTamil_lang`](https://github.com/Maruff/eTamil_lang) |
| JavaScript and TypeScript libraries: the form and list framework | [`Maruff/eTamil-ciRal`](https://github.com/Maruff/eTamil-ciRal) |
| The application built from them | **here** |

nivaqi holds very little code: routes, the mapping to the database, configuration,
migrations, seed data, and screens composed from the ciRal framework. It holds no
business rule and no helper that two modules share. Anything like that moves at once
to `nUlakam` or to ciRal ([decision 0017](docs/DECISIONS/0017-where-code-lives.md)).

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
| Interface | React + Next.js + TypeScript, presentation only; the framework is in [eTamil-ciRal](https://github.com/Maruff/eTamil-ciRal) ([decision 0016](docs/DECISIONS/0016-front-end.md)) |
| Database | PostgreSQL in the cloud; SQLite on the edge and in the PoS |
| Backend language | **eTamil by default**; exceptions are recorded ([decision 0014](docs/DECISIONS/0014-databases-and-language.md)) |
| Point of sale | A separate eTamil process on Android, offline-first, over local SQLite |

## Documentation

Everything a person reads to use or extend nivaqi inside a code folder is written in
English and in Tamil: `X.md` and its twin `X.ta.md`. An automated check enforces it
(`karuvi/qamiz_iNY_caripArppu.qmz`, run in CI). This README's Tamil twin is
[README.ta.md](README.ta.md). Code comments follow the eTamil convention (English
inside `__…__`, Tamil in plain text). `docs/` — the architecture, the decisions and
the capability list — is English only, by decision. The Tamil text is a first draft;
the owner will review it.

## Licence and authorship

AGPL-3.0-or-later — see [LICENSE](LICENSE).

Copyright (C) 2026 Isha Maruff and Esan Maruff. See [AUTHORS](AUTHORS).
