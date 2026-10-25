# 0012 — Product principles: modular like Odoo, simple like Sage 50

**Status:** accepted in direction, 2026-10-26. **Every numeric target below is
a proposal**, not a decision. The owner sets the numbers.

## Context

The owner's direction: *nivaqi should be modular like Odoo but simpler like
Sage 50 (Peachtree).*

This is read as simplicity in **what a customer sees and configures**, with the
module scope unchanged. If the intent was a smaller feature set, this decision
is wrong and should be superseded, not patched.

It matters now because the first UI shell and the kernel set the product's
personality. Odoo's cost to a small business is its configuration surface, not
its module count.

Sage 50 is characterised here from memory: a home page organised by task area
(Customers & Sales, Vendors & Purchases, Inventory & Services, Employees &
Payroll, Banking, Company), with every transaction posting through the ledger.
Check that against the product before copying any specific.

## Decision

| | Take | Leave |
|---|---|---|
| **Odoo** | Modules a customer switches on; one integrated ledger and partner list; the order → delivery → invoice chain | The configuration surface, workflow editors in core, a marketplace of third-party apps |
| **Sage 50** | Accounting as the spine; a home page organised by task; strong defaults; guided setup; one way to do each job | The single-company, desktop, accounting-only limits |

Five rules:

1. **Accounting is the spine, not one module among peers.** Every module posts
   through the ledger (ARCHITECTURE section 6.4, rule 1), and accounting is
   reachable from every section of the home page.
2. **Screens are named for tasks.** "Enter a bill", "Receive a payment" and
   "Run payroll" are screens; "Account moves" is not. A module is a section of
   the home page, and a module the tenant has not bought is absent from it.
3. **Few settings, strong defaults, one way to do each job.** Each module has a
   settings budget and every setting justifies itself. Chart-of-accounts
   templates per business type carry the defaults. There is no workflow editor
   or form designer in core; customisation is the private Customizing Studio
   (decision 0008). No field is required that the ledger does not need.
4. **Guided setup.** Company, financial year, GSTIN and opening balances lead
   to a first invoice without reading documentation.
5. **No platform vocabulary reaches the customer.** A customer sees a company,
   never a tenant, a pool or a silo.

### Proposed targets

**All three are proposals.** None is decided, and none has been measured.

| Target | Proposed | How it would be measured |
|---|---|---|
| Time to first invoice, from sign-up, on a clean company with a business-type template | ≤ 10 minutes | Someone who has not seen the product, timed |
| Settings visible by default, per module | ≤ 10; the rest behind "Advanced" | Counted from the module's `README.md`, which lists every setting |
| Sections on the home page | ≤ 7 | Counted |

Templates at launch follow the pilots (decision 0007): pharmacy, retail,
services and bakery restaurant. That count is derived from the pilots, not a
target.

## What this does not mean

- **Not a smaller feature set.** The module plan is unchanged.
- **Not an extension ecosystem.** eTamil has no namespaces and no private
  helpers, so outside code cannot be sandboxed in the same process. "Modular"
  means what a customer turns on. Customisation is the Studio.
- **Not two editions.** Depth is an entitlement, not a second build.

## Consequences

- **The first pilot is the least Sage-50-like.** The IT consulting MNC needs
  multi-entity, segregation of duties and SSO, which is the Odoo end of the
  range, and it goes live first (decision 0007). If wave 1 is built for it, the
  UI shell sets the tone. Build the shell and the setup wizard as a
  small-business experience first (books and invoicing, which the staged MNC
  go-live already allows) and treat the MNC's depth as an entitlement.
- **These principles are only truly tested at wave 2**, on the dry fruit store.
  Run the first-invoice measurement there.
- **Templates need rate data.** `nUlakam` seeds no rate
  (`kaNakkiyal/vari_vikiqam`: every lookup takes a date, and none is seeded), so
  "GST preconfigured" means nivaqi loads effective-dated rate tables from
  `payiRci/`, each with an effective date and a source. Someone owns keeping
  them current.
- **Platform vocabulary stays in code and operations documents.** User-facing
  strings say "company". Add this to the i18n review.
- **The kernel and the library families are unchanged.** Simpler for customers
  is not smaller for developers.
- **Each module's `README.md` lists its settings**, because a budget needs a
  list to count.
- **The entitlement manifest** in the SaaS architecture decision (not yet
  written) decides which home-page sections a company sees.

## Not taken

- **Configurability first**, as Odoo does: flexibility paid for in settings.
- **Two editions** (Essentials and Enterprise as separate builds): two products
  to maintain, where entitlements give the same progressive depth.
- **A smaller feature set**: not taken, on the reading above.

## Open

- The Sage 50 characterisation is from memory and needs checking.
- The three numeric targets are the owner's to set.
- Whether an unentitled module is hidden or shown as an upsell. This decision
  says hidden; it is a recommendation, not a settled point.
