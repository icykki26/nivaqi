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
| [0006](0006-multi-tenancy.md) | Multi-tenancy — data model now, SaaS product later | accepted, **one reading to confirm** |
| [0007](0007-pilots.md) | Four pilots, staggered, MNC first | accepted |
| [0008](0008-repositories-and-licence.md) | Repositories, visibility and licence | accepted, **one item is time-limited** |
| [0009](0009-where-this-document-lives.md) | Where the architecture document lives | accepted |
| [0010](0010-authorship.md) | Authorship | accepted |

## The two that still want attention

**0006** records a reading of "multi-tenant SaaS — accepted" as *data model
now, product surface later*. Correcting that reading is free today and a
migration after wave 1.

**0008** notes that dual-licensing `nUlakam` is the owner's to do only while
there are no external contributors. That window closes on the first
contribution accepted without a contributor licence agreement.

## Writing a new one

Number it in sequence. Say what the context was, what was decided, why, and
what it costs. Record what was rejected — a decision whose alternatives are not
written down gets made again. When one is superseded, mark it so and link
forward rather than editing it.
