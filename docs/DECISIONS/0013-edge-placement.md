# 0013 — Edge placement: local-first with a single authority

**Status:** accepted, 2026-10-26, recorded on the owner's instruction.
Amends [0011](0011-saas-architecture.md) by adding a third placement; the edge
database is settled by [0014](0014-databases-and-language.md). **Costs
and capacities here are estimates or unmeasured**, and are marked as such.

## Context

The owner wants to use minimum cloud for the maximum customer base, offering
the service at low cost or free to most customers. Customers may also be in
places with unstable internet. The question was why not design the whole
product like the PoS, to work offline (decision 0005).

## Decision

**Local-first, with one authority per company.**

The authority is the one database where a company's posted records are final.
It runs wherever the placement puts it. A device that cannot reach it queues
intents in an outbox, as the PoS does, and replays them when it can. There is
no multi-master merge.

**One code base, three placements.** Which one a customer gets is a deployment
choice, not a product difference.

| Placement | Authority runs on | Fits |
|---|---|---|
| **Edge** | The customer's premises (a PC, Raspberry Pi or Android device), serving the LAN. The internet is used only for encrypted backup, updates and optional sync or remote access | Single-premises small businesses: the pharmacy, the dry fruit store, the bakery |
| **Cloud pool** | Shared schema (0011) | Customers who want it hosted |
| **Cloud silo** | A dedicated database and processes in the cloud (0011) | The IT consulting MNC |

An edge install is a silo that lives at the customer's site, so it costs the
platform no compute. A tenant moves between placements with the tenant-scoped
export tool from 0011. The control plane records each tenant's placement.

### Why not everything offline

The PoS works offline because its conflicts are narrow and commercial. A shared
ERP has wider ones that cannot be merged automatically: gapless numbering,
stock that cannot go negative, closed periods, credit limits, e-invoice IRNs. A
general multi-master sync would be the hardest thing in the project.

### What makes single-authority sync tractable

The rules already adopted do most of the work (ARCHITECTURE section 6.4:
rules 2, 4, 7 and 9).

| Kind of data | Offline rule |
|---|---|
| Posted documents and journal entries | Immutable, so they never conflict. A correction is a reversing entry |
| Drafts | Owned by the device that created them until posted |
| Master data (partners, prices) | Last writer wins per field, with an audit trail |
| Identifiers | Globally unique (UUIDv7, rule 9), so rows from different devices do not collide |
| Numbering | A series per device, gapless within the series |
| Stock | Advisory offline, reconciled on sync, with a commercial rule for a line sold twice (0005) |
| Credit limits | Advisory offline, enforced on sync |
| Closing a period | Needs a fence: every registered device synced up to the cut-off |

### What stays online regardless

eCommerce, the portals and the website must be publicly reachable. E-invoicing
needs the IRP, so a B2B invoice raised offline is not final until it syncs; the
IRP state in 0002 already models that. The private AI and BI products see only
data that has synced.

### The kernel assumes no always-on cloud

Nothing in `atiqqaLam/` may require a connection to a cloud service to complete
a request. Anything that must reach the cloud goes through the outbox.

## What was checked

- **eTamil has no updater.** I searched the compiler source, packaging, the
  install script and the command reference for self-update or upgrade support
  and found none.
- **LAN serving works.** `--host 0.0.0.0` is documented.
- **The SQLite driver sets neither WAL nor a busy timeout explicitly.** I found
  no mention in `etamil_compiler/src/db/sqlite.rs`. The underlying library may
  apply defaults; I did not check.
- **Footprint.** An eTamil server with nine library imports held 10.7 MB
  resident (0011).

### SQLite and money

The ledger's balance check is an equality between two sums of money. SQLite
stores decimals as text in this design (ARCHITECTURE section 6.4, rule 6), so I
tested what `SUM` returns, in SQLite 3.50.4 through Python, over random
two-decimal amounts. Each trial summed the same amounts in two different orders:

| Ledger | Trials | `SUM(text)`: debit ≠ credit | `SUM(text)`: differs from the exact decimal sum | `SUM(integer paise)`: debit ≠ credit |
|---|---|---|---|---|
| 200 lines, ₹1–5,000 | 1,500 | 0 | **44** | 0 |
| 5,000 lines, ₹1–2,00,000 | 150 | 0 | 0 | 0 |
| 5,000 lines, up to ₹50 crore | 150 | 0 | 0 | 0 |

`SUM` returns a binary float. In about 3% of the small-ledger trials (44 of
1,500) it differed from the exact decimal sum. The debit-equals-credit check
never failed in any of the 1,800 trials, but I cannot promise that it cannot.
Integer paise were exact throughout. I have not investigated why the two
larger runs showed no differences.

**Caveats.** This is SQLite through Python, not eTamil's bundled SQLite build
and not through its driver. Older SQLite versions sum less carefully. How the
driver reads a float result back into a decimal is untested.

**Recommendation, not a decision:** on any engine that may be an edge database,
sum money in integer paise (`nUlakam/paNam/kAcu.qmz` is paise-exact money) or
aggregate in eTamil, and never `SUM` a text column. Rates and quantities need
more than two decimals and stay exact decimals, with arithmetic in eTamil only.
Rule 6 in ARCHITECTURE section 6.4 is **not changed** until this is decided.

## Not measured

- **Concurrent writers.** Several terminals writing through one eTamil process
  to one SQLite file, and whether `database is locked` errors appear.
- **The SQLite version bundled with eTamil**, and `SUM` through its driver.
- **The sync protocol**, which does not exist yet beyond the PoS outbox pattern.
- **Backup size** per customer, and the **free-tier cloud cost** (backup
  storage, update hosting, sync relay, licensing). No figure is given.

These join the M0 spike (ARCHITECTURE section 15).

## Costs and risks

- **Running many customer-hosted instances is the biggest new cost, and it
  falls on the developer, not the cloud bill.** There is no updater, so version
  skew, migrations on machines nobody can reach, and sync-protocol compatibility
  are all new work. Needed: an updater, staged rollout, and schema and protocol
  versioning (expand/contract, as in 0011). **Not estimated.**
- **The backup becomes the product.** If a customer's device dies, the
  off-site encrypted backup is all they have. It must be restored regularly in
  testing. eTamil has XChaCha20-Poly1305 over Argon2id for the encryption.
- **Data on customer devices:** theft, encryption at rest, and the customer
  holding personal data they are responsible for.
- **Two database engines** (SQLite on the edge, PostgreSQL in the cloud) mean
  two test targets and a common subset of SQL. Accepted in 0014.
- **Free users cost support.**

## Consequences

- Wave 1 is unchanged: the MNC is a cloud silo.
- Edge must be ready by wave 2, the dry fruit store, which is also when the PoS
  sync mechanics are built. The edge authority generalises them.
- The sync protocol is an extension of the PoS outbox, not a new mechanism.
- A premises machine can serve several terminals, but that depends on the
  untested concurrent-writer behaviour above.

## Not taken

- **Pure peer-to-peer or multi-master sync:** the hardest problem in the
  project, for little gain over a single authority.
- **Cloud only:** it rules out the free tier and unstable-internet customers.
- **A client-side offline cache over a cloud authority:** it helps one user for
  a short outage, but not a premises with several users and a long one.

## Open

- **The edge database:** settled as SQLite in [0014](0014-databases-and-language.md).
- **Money representation.** SQLite is the edge database, so this must be
  decided before the first schema (above).
- **What "free" means:** limits on users, documents and storage. A business
  decision for the owner.
- **The updater's design,** and which hardware is supported (PC, Raspberry Pi,
  Android).
- **How long an edge install may run without contacting the cloud,** for
  licensing and for the period-close fence.
