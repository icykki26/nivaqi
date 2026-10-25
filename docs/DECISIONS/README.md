# Decisions

One numbered file per architectural decision: what was decided, what was
rejected, and why — recorded so that a decision is not relitigated or
accidentally reversed.

The style follows `eTamil_lang/docs/ARCHITECTURE.md`, which keeps its reasoning
beside its decisions for the same reason.

| # | Decision | Status |
|---|---|---|
| [0001](0001-the-name.md) | The name — நிவதி, `nivaqi` | accepted |
| [0002](0002-compliance-scope.md) | Compliance scope — books, e-invoicing, e-way bills; returns summarised, not filed | accepted |
| [0003](0003-sales-and-purchase.md) | Sales & Purchase is a first-class module | accepted |
| [0004](0004-website-and-blog.md) | No CMS in eTamil | accepted |
| [0005](0005-pos-offline-first.md) | The PoS is offline-first | accepted |
| [0006](0006-multi-tenancy.md) | Multi-tenancy — data model now, SaaS product later | **superseded by 0011** |
| [0007](0007-pilots.md) | Four pilots, staggered, MNC first | accepted |
| [0008](0008-repositories-and-licence.md) | Repositories, visibility and licence | accepted, **one item is time-limited** |
| [0009](0009-where-this-document-lives.md) | Where the architecture document lives | accepted |
| [0010](0010-authorship.md) | Authorship | accepted |
| [0011](0011-saas-architecture.md) | SaaS architecture — shared schema, entitlements, pools, silos, and backup | accepted, amended by 0013; **estimates and unmeasured items marked** |
| [0012](0012-product-principles.md) | Product principles — modular like Odoo, simple like Sage 50 | accepted in direction; **targets are proposals** |
| [0013](0013-edge-placement.md) | Edge placement — local-first with a single authority | accepted, amended by 0014; **costs and capacities unmeasured** |
| [0014](0014-databases-and-language.md) | Databases by placement, and eTamil as the only backend language | accepted; **one reading to confirm** |

## The five that still want attention

**0008** notes that dual-licensing `nUlakam` is the owner's to do only while
there are no external contributors. That window closes on the first
contribution accepted without a contributor licence agreement.

**0011** carries an effort estimate for the control plane (4–6 weeks, a guess)
and a list of things not yet measured: throughput per core with PostgreSQL,
memory under load, and row-level security through the driver. No customer-count
claim is made until they are.

**0012** carries three numeric targets that are proposals only. The owner sets
the numbers.

**0013** leaves the money representation open, and its biggest cost, running a fleet of customer-hosted
instances with no updater, is not estimated.

**0014** reads "backend" as all server-side and operational code, with the
front end and infrastructure products outside the rule. It also leaves the
money representation (now forced, since SQLite is the edge database), how
UUIDv7 is generated, and whether the rule extends to development tooling.

## Writing a new one

Number it in sequence. Say what the context was, what was decided, why, and
what it costs. Record what was rejected — a decision whose alternatives are not
written down gets made again. When one is superseded, mark it so and link
forward rather than editing it.
