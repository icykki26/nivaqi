# 0011 — SaaS architecture: shared schema, entitlements, pools, silos, and backup

**Status:** accepted, 2026-10-26, recorded on the owner's instruction.
Supersedes [0006](0006-multi-tenancy.md); amended 2026-10-26 by
[0013](0013-edge-placement.md), which adds the edge placement, and by
[0014](0014-databases-and-language.md), which fixes the databases and the
language. **The effort figure and every
capacity statement are estimates or unmeasured**, and are marked as such.

## Context

nivaqi is to be offered as a SaaS platform where scalability is critical. Some
modules are basic for every business and others are wanted by few customers,
and the question was how to serve a large customer base with minimum resources:
microservices, modules as separate products, a single ERP like Odoo, or
something else.

This also settles the reading flagged in 0006: the owner intends a SaaS
*product*, not only a multi-tenant data model.

## Decision

**A modular monolith: one codebase, one shared schema, modules sold as
entitlements, deployables split by load profile, and a separate database only
for customers who need one.**

1. **One shared schema, tenant on every row.** PostgreSQL row-level security is
   a second lock, set with `SET LOCAL` inside the kernel's transaction wrapper
   (`atiqqaLam/qaLap_parivarqqaZY.qmz`). No query reaches the database except
   through `vatikattal`.
2. **Modules are entitlements.** A control-plane table maps tenant → plan →
   modules. It is enforced once, in `aZumaqi`, at route entry. A small manifest
   declares module dependencies, so Inventory cannot be sold without
   Accounting. Switching a module on is a row, not a deployment, and data is
   never deleted when a module is switched off.
3. **Split by load profile, not by popularity.** A pool is an entry file that
   imports a subset of route files and is run as
   `etamil --server pool_core.qmz`: same binary, same database. Candidate pools:
   - **core** — partners, accounting, sales and inventory. These share
     transactions, so they are not split.
   - **PoS sync** — bursty at open and close of day.
   - **reports** — its database connection points at a read replica. eTamil's
     one-connection-per-process limit makes this natural.
   - **jobs** — payroll, year-end close, stock revaluation; one per tenant at
     a time.

   A light, rarely used module costs nothing sitting in the core pool.
4. **Pooled by default, siloed on request.** A customer who needs isolation
   gets its own database and a small process group, because one process is one
   database. Keep silos few: migrations run once per database.
5. **Stateless processes behind a load balancer.** State lives in PostgreSQL
   behind PgBouncer in transaction mode and in Redis for sessions and rate
   limits. Rate limits are per tenant, at the edge.
6. **Scale the database last, in order:** a larger machine; replicas for
   reports; partitioning large tables by tenant; and only much later Citus or
   sharding.

### Placements

The same code runs in three places. Which one a customer gets is a deployment
choice, not a product difference. The control plane records each tenant's
placement.

| Placement | Authority runs on | Defined in |
|---|---|---|
| Cloud pool | The shared schema described above | this decision |
| Cloud silo | A dedicated database and processes, in the cloud | this decision |
| **Edge** | The customer's premises, serving the LAN; the cloud is used for backup, updates and optional sync | [0013](0013-edge-placement.md) |

### Options not taken

| Option | Why not |
|---|---|
| Microservices per module | An invoice, its journal entry and its stock move must commit together (`examples/katY` does this in one `BEGIN`/`COMMIT`). Splitting them turns that into sagas, and the ledger being correct by construction is the product's edge. It would also be 18 deployables for one developer |
| Modules as separate products | Commercially yes, technically no. Inventory valuation posts to the ledger and sales needs partners; separate products would mean integrating one's own modules |
| Database per customer (Odoo-style) | Hundreds of tables and a migration run per customer; in eTamil it also means a process group per customer |
| Schema per tenant | Migrations multiply by tenant count and the catalog bloats; it does not solve the external-documents problem below |
| Partition per tenant | Partitions multiply as tables × tenants |

## What was measured, and what was not

An eTamil `--server` process importing nine library modules (ledger, GST,
costing, Ind AS 115, tax, timesheets, documents) was **ready in 1.4 s and held
10.7 MB resident, 6.7 MB private**. That is why pools and silos are affordable.

It is a first data point only: idle, on Windows, one route, no database
connection.

**Not measured:**

- Throughput per core with PostgreSQL under real handlers. This decides
  customers per node, so **no customer-count claim is made anywhere**.
- Memory under load with the full import set. Handlers compile with their
  file's imports, so memory may grow with routes × imports.
- Row-level security through eTamil's PostgreSQL driver, with `SET LOCAL` under
  PgBouncer transaction pooling.
- UUIDv7 generation (below) and how fast the export tool runs when written
  in eTamil (0014).
- A static Linux build, which is what would be deployed.
- **Integer money through the driver.** Stored money is integer minor units
  (0015); the check is that it round-trips as `INTEGER` and sums exactly.
- **Concurrent terminals** writing to one SQLite file through one eTamil
  process (0013).

These join the M0 performance spike (ARCHITECTURE section 15).

## Backup and restore

Without a database per customer, a customer cannot be backed up with `pg_dump`.
Two problems, with different answers.

**1. Disaster recovery for the whole platform.** Continuous WAL archiving plus
periodic base backups, giving point-in-time recovery of the entire database. It
covers a lost server, corruption or a bad deploy, and restores every tenant to
the same moment. Backups are encrypted, kept in another region, and **restored
on a schedule**; an untested backup is not a backup.

**2. Restoring one customer.** Restore a point-in-time copy to a scratch
instance, extract that tenant's rows, and load them into production. A
**tenant-scoped export tool** does the extraction: every row carries a tenant,
so it walks the tables in foreign-key order and copies `WHERE tenant = X`. One
tool serves four needs: per-tenant restore, a nightly per-tenant backup, the
customer's own data export (the tenant lifecycle needs one), and moving a tenant
from the pooled tier to its own database. It is an eTamil program in `karuvi/`, exchanging
files between engines (decision 0014); backend code is eTamil by default.

**Rolling a tenant back is not safe for a ledger.** Some things have already
left the system and cannot be unsent: e-invoice IRNs, e-way bills, UPI payments,
emails, sent outbox rows. Restoring a tenant to 10:00 leaves an invoice filed at
11:00 with the tax authority and absent from the books. So:

- **The default restore goes into a side copy**, to recover specific records or
  to compare, without overwriting production.
- An overwrite happens only with the tenant frozen read-only, and is then
  reconciled against the outbox.
- The audit trail is append-only and survives a restore. A restore is itself an
  audited event.
- Gapless numbering sequences go back with the rows they number. Numbers
  already issued externally have the same problem as the documents.

**Identifiers must not collide across databases**, or a row cannot move between
the pooled tier, a silo and a restored copy. Primary keys are globally unique
(UUIDv7) or composite `(tenant, id)`. This is rule 9 in ARCHITECTURE section
6.4, and it must be settled before any schema exists.

**Backup is a plan feature.** The pooled tier gets platform point-in-time
recovery plus tenant export and the side-copy restore. The silo tier gets its
own database, so its own `pg_dump` and point-in-time recovery, with its own
recovery targets. A customer who needs independent restore guarantees by
contract is a silo customer.

Deleted tenants persist in backups until those age out. The retention policy
must say so explicitly; it matters for deletion requests under personal-data
law, which should be checked before launch.

## New scope this decision adds

- **Control plane:** tenant registry, entitlements, provisioning and metering.
  Billing goes through a payment provider rather than being built. **Estimate:
  4–6 weeks for a minimal version, on top of wave 1. This is a guess, not a
  measurement**, and the effort totals in decision 0007 do not include it.
- **Tenant lifecycle:** export, suspend and delete.
- **Shared object storage.** Attachments (`atiqqaLam/kOppiNYppu`) and generated
  documents are written to local disk today, which fails on two or more nodes.
  An S3-style client is needed. The HMAC primitive for request signing exists
  in the host; the client does not.
- **Expand/contract migrations**, so old and new pools can share a database
  during a rollout.

## Consequences

- Wave 1 grows by the control plane, so time to the first real user is longer
  than the 31–39 weeks in 0007 by about the estimate above.
- The four pilots become the first tenants. The IT consulting MNC is the likely
  first silo.
- The language limitation recorded in 0006 (one database connection at a time,
  no named handle) is no longer a blocker, because one process per database is
  the design. If physical isolation ever needs more than that, it is still a
  language request.

## When to change shape

Start with one pool, the control plane and row-level security. Do not build
pools speculatively.

- **Split a pool** when interactive latency is hurt by jobs, reports or sync.
- **Silo a tenant** when a customer requires it contractually.
- **Revisit the database** when replicas and partitioning are no longer enough.

## Open

- Tamil names for the control plane, entitlements and tenant. Not guessed here;
  the owner chooses.
- Recovery-point and recovery-time targets per tier. They depend on
  infrastructure not yet chosen.
- Hosting region and provider.
- The billing provider.
- The object-storage client is written in eTamil (decision 0014); its design
  is open.
- Legal review of personal-data obligations before launch.
