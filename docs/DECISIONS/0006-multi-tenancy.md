# 0006 — Multi-tenancy

**Status:** superseded by [0011](0011-saas-architecture.md), 2026-10-26.
The reading below was settled: the owner intends a SaaS *product*. The text
that follows is kept as it was written.

## Context

The original question was whether nivaqi would ever be multi-tenant SaaS. The
answer given was "accepted". Decision 0007 then settled four pilots across four
unrelated legal entities, which forces the question regardless: four unrelated
businesses on one system is tenancy, not multi-company.

## Decision

**The data model is multi-tenant from M0. The SaaS product surface is
deferred.**

In now, because retrofitting it is a migration:

- Every row carries a tenant as well as a company, written by the kernel and
  never by a module.
- No query reaches the database except through `atiqqaLam/vatikattal`, so the
  tenant predicate is applied in one place and cannot be forgotten.
- No cross-tenant sequences, and no process-wide cache keyed without a tenant.

Deferred, because it is a product and four pilots do not need it:

- Self-service provisioning.
- Per-tenant branding.
- Subscription billing.
- Written isolation guarantees of the kind a paying tenant expects.

## The reading to confirm

*Resolved 2026-10-26: the second reading. See 0011.*

"Multi-tenant SaaS — accepted" is recorded here as *multi-tenant data model
now, SaaS product surface later*. The alternative reading is *build the SaaS
product*, which grows the kernel by provisioning and billing and lengthens
wave 1.

Correcting this today costs nothing. Correcting it after wave 1 is a migration
across every table.

## Consequences

- Multi-company and multi-tenant scoping are the same mechanism, so doing both
  from M0 costs almost nothing over doing multi-company alone.
- **One constraint to carry:** eTamil holds one database connection at a time
  and cannot name a handle, so database-per-tenant is not expressible today.
  If tenant isolation ever has to be physical, that is a language request, and
  it wants filing early rather than discovered at module nine.
