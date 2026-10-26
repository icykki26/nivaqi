# nivaqi — an ERP written in eTamil

**Status: the founding architecture, with its decisions taken.** Written
2026-10-24 against eTamil 1.4.2 and `nUlakam` at `eTamil_lang@7df1fbd`. No
nivaqi code exists yet. The ten decisions in [DECISIONS/](DECISIONS/) were
settled by the owner on 2026-10-24 and are recorded there individually; this
document is written as though they hold, because they do.

It exists so that the first line of nivaqi code is written against a decided
architecture rather than a guess, and so that the decisions can be argued with
before they become expensive. When this document and the code disagree, the
code wins — fix the document.

---

## 1. What nivaqi is

An ERP in the Odoo sense — a set of business modules sharing one database, one
identity, one ledger and one interface — whose domain logic is written in
eTamil and lives in `nUlakam`.

### The name

**நிவதி** — `nivaqi` under the ezuqqu romanization, which
`scripts/transliterate.py` confirms is an exact and unique round trip.

It is an acronym of the Tamil for *enterprise resource planning*:

| | Tamil | ezuqqu | |
|---|---|---|---|
| **ni** | நிறுவன | `niRuvaZa` | enterprise |
| **va** | வள | `vaLa` | resource |
| **qi** | திட்டமிடல் | `qittamital` | planning |

The name says what the product is, in Tamil, and romanizes on-scheme. Both
spellings are canonical and neither is a transliteration of the other for
convenience — they are the same name written two ways, which is the language's
own central claim.

### Why this is worth doing

1. **The accounting is already there, and it is the hard part.** Most ERPs bolt
   a ledger onto a transaction system. `nUlakam/kaNakkiyal/` is a double-entry
   ledger, a chart of accounts, GST on transactions, periods, period close,
   allocation and ageing, three statements, depreciation, payroll arithmetic
   and multi-entity currency — written before any ERP needed it.
   `examples/katY` already turns an order into a journal entry rather than a
   row to reconcile later. That is the inversion most ERP projects never
   achieve.
2. **The research claim gets its pilot.** `eTamil_lang/docs/ROADMAP.md` records
   Phase 4 as "released; no pilot deployed". nivaqi is the pilot — and with
   four businesses waiting (section 2), it is four pilots.
3. **The library-first constraint is measurable.** `examples/aluvalakam`
   measured it once: 223 application lines against BeakPMO's 3,029 for the same
   domain logic. nivaqi tests whether that ratio survives eighteen library families.

### What nivaqi is not

| Not | Why |
|---|---|
| An Odoo clone | Odoo's value is a dynamic ORM, a view metadata engine and a module loader. Reimplementing those is years of work producing no business capability. Section 6.3 argues this specifically |
| A framework | nivaqi is an application. Reusable parts go to `nUlakam`, where they are readable eTamil, not into a nivaqi "core" only nivaqi can use |
| A CMS | Decision 4. A blog has no business rule, so it can contribute nothing to `nUlakam` |
| A filing agent | Decision 2. nivaqi computes GST and TDS and exports the figures. It does not file returns |

### Product principles

*Modular like Odoo, simple like Sage 50.* Full reasoning, and what was
rejected, is in [decision 0012](DECISIONS/0012-product-principles.md). It is
read as simplicity in what a customer sees and configures; the module scope is
unchanged.

1. **Accounting is the spine.** Every module posts through the ledger, and
   accounting is reachable from every section of the home page.
2. **Screens are named for tasks**, not tables. A module is a section of the
   home page; a module the company has not bought is absent from it.
3. **Few settings, strong defaults, one way to do each job.** Each module's
   `README.md` lists its settings. Templates per business type carry the
   defaults. No workflow editor in core; customisation is the private Studio.
4. **Guided setup** from sign-up to a first invoice.
5. **No platform vocabulary reaches the customer.** A company, never a tenant,
   pool or silo.

**Proposed targets — proposals, not decisions, and none measured:** first
invoice within 10 minutes of sign-up; at most 10 settings visible by default
per module; at most 7 home-page sections.

The consequence to plan for: the first pilot, the IT consulting MNC, is the
least Sage-50-like. Build the shell and setup wizard as a small-business
experience first and treat the MNC's depth as an entitlement.

---

## 2. The four pilots

Four businesses are ready to adopt and test. They are the single most important
input to this architecture, because between them they demand considerably more
than the original module list covered, and because they do not demand it in the
same order.

> **These requirements are inferred from the business types, not gathered.**
> Each needs confirming with the business before its wave starts. The point of
> the table is that the four are *not* interchangeable, which is a planning
> fact that holds regardless of the details.

| Pilot | Needs | Does **not** need | New ground |
|---|---|---|---|
| **IT consulting MNC** | Partners, accounting, projects, timesheets, T&M and fixed-price billing, revenue recognition, multi-entity and multi-currency, export invoicing, payroll at scale, strict access control and SSO | Inventory, PoS, manufacturing | **Almost none** — `varuvAy/` (Ind AS 115), `vaLam/nEraqqAL` (timesheets), `vaLam/paqivu` (project cost to ledger), `qittam/` (projects) and RS256 SSO all exist |
| **Dry fruit retail** | Inventory, retail PoS, **weight-based UoM and price per kg**, loose sale from bulk, variable-weight barcodes, grades, shrinkage | Projects, manufacturing (mostly) | Weight UoM, scale/barcode handling, repacking bulk into retail packs |
| **Pharmacy** | Everything the retail store needs, plus **batch and expiry on every line, FEFO picking**, near-expiry reporting, supplier returns of expired stock, drug schedules (H, H1, X) with prescription capture, drug licence numbers, substitutes | Manufacturing | Batch/expiry, FEFO, a regulated-item register |
| **Bakery restaurant** | **Recipes and production orders**, yield and wastage, recipe costing, short shelf life, daily production planning, **restaurant PoS** with tables, KOT and split bills, plus a retail counter | Projects | Manufacturing, restaurant-mode PoS |

### What this changes

**Three things, and all three are consequential.**

1. **The IT consulting MNC can go live without Inventory or PoS.** It is the
   only one of the four that can, it needs the least new library code, and it
   exercises the parts of `nUlakam` that are already strongest. It should be
   pilot one — which means **Employee and Timesheet move from the third wave to
   the first**, ahead of Inventory and PoS.
2. **Three new library families are needed** that the original plan did not
   have: batch/expiry with FEFO, weight-based units, and manufacturing.
   Section 8 places them.
3. **Four unrelated legal entities on one system is multi-tenancy**, not
   multi-company, and the owner intends a SaaS product. See
   [decision 0011](DECISIONS/0011-saas-architecture.md) and section 6.7.

### A warning about running four at once

One developer cannot build wave *n+1* and support four live pilots. A live
pilot generates support, data corrections and change requests that compete
directly with building, and they win, because a business that has stopped
invoicing does not wait.

**Stagger the go-lives, one per wave, and budget 20–30% of capacity for
supporting each pilot already live.** The effort table in section 12 assumes
this; without it, the schedule is fiction. If all four must go live close
together, that is a second person, not a longer timeline.

---

## 3. The three constraints, made checkable

### 3.1 Minimal code

**Rule.** A nivaqi module file may do six things and no others: read the
request, authorise it, load records, call `nUlakam`, persist the result, shape
the response.

Anything that is a *rule* — arithmetic, a validation, a state transition, a
rounding decision, an eligibility test, a tax treatment — belongs in `nUlakam`,
where it is testable without a server and reusable by the next module. If a
handler contains a business rule, the review outcome is "move it", not "add a
comment".

| Limit | Value | Why |
|---|---|---|
| A route handler | ≤ 40 code lines | Beyond that it is orchestrating too much, or holding a rule |
| A module source file | ≤ 500 code lines | `aluvalakam_kYyALi.qmz` is 1,031 and already at the edge of readable |
| Application code as a share of module total | ≤ 25% | `aluvalakam` ran at about 15% (223 of 1,516). 25% is the alarm, not the target |
| Business rules in SQL | zero | A rule in a `CASE` expression is invisible to the tests and unavailable to the next module |
| Business rules in React | zero | Same, plus the client is not a trust boundary |

Measured per module and reported in its README. A breach is not blocked, but it
is explained in that module's `DECISIONS.md`.

### 3.2 Detailed documentation

`nUlakam` already holds the convention: **153 library modules, 23 `uqavi/`
directories, 937 runnable worked examples** — roughly one executable example
per public function, each with a bilingual comment saying what the function
answers and often what it is *not* (`alaku_pawkaLippu.qmz`: "this is not
profit; the fixed cost has not been met yet").

nivaqi inherits that unchanged and adds a document set per module
(section 11).

### 3.3 Libraries under `nUlakam`

Every module either adds a `nUlakam` family or extends one. Section 8 lists
what each adds.

**The cross-repository consequence.** nivaqi is `icykki26/nivaqi`; `nUlakam` is
in `Maruff/eTamil_lang`. Module work therefore splits across two repositories
on every feature:

```
eTamil_lang   nUlakam/carakku/*.qmz            the rules
nivaqi        carakku/carakku_kYyALi.qmz       the routes
```

Policy:

- The library change lands in `eTamil_lang` **first**, with its `uqavi/`
  examples and `cOqaZY` tests, and is released.
- nivaqi pins an eTamil version in `.env` and in CI, and bumps it
  deliberately. It never tracks `main`.
- For the inner loop, `ETAMIL_PATH` points at a local `eTamil_lang` checkout.
  CI never does this — CI installs the pinned release, which is what catches
  "it works here because my `nUlakam` is ahead".
- A nivaqi PR needing an unreleased library function links the `eTamil_lang`
  PR and does not merge first.

This is friction, and it is the correct friction: it is what stops nivaqi
quietly growing a private copy of the standard library.

---

## 4. What already exists

### 4.1 Libraries that are ERP modules in all but name

| `nUlakam` family | What it gives an ERP | Which pilot |
|---|---|---|
| `kaNakkiyal/` | Chart of accounts, journal and ledger with a balance check, GST with per-line CGST/SGST/IGST splitting, periods, Indian and calendar fiscal years, allocation and ageing, trial balance / income statement / balance sheet / cash flow, entities and currency conversion, period close, depreciation, payroll arithmetic, effective-dated tax rates | all |
| `varuvAy/` | Ind AS 115 end to end: the five steps, contract modifications, principal-versus-agent, contract assets and liabilities, contract costs, onerous contracts | MNC |
| `vaLam/` | Resources with effective-dated rates, **timesheets**, and posting project cost to the ledger | MNC |
| `qittam/` | WBS, critical path, dependencies, earned value, scheduling on a real calendar, rescheduling on actuals, change control, **lifecycle state machines as data**, tasks, portfolio | MNC |
| `celavu/` | Cost centres, the cost sheet, overhead absorption, standard-costing variances, marginal costing and CVP, reciprocal apportionment, **process costing with equivalent units**, activity-based costing | bakery |
| `nErativari/` | Five heads and loss set-off, Chapter VI-A, total income to tax payable, TDS, advance tax, deferred tax under Ind AS 12 | all |
| `oruwkiNYppu/` | **The integration kernel**: an outbox that cannot queue twice, retry and backoff, idempotency fingerprints, JSON Patch, field ownership, inbound webhook verification. Named for Azure DevOps, almost entirely generic | all |
| `cuwkam/` | Customs, the duty cascade, **e-way bill validity that rounds up** | retail, pharmacy |
| `vativam/` | JSON, base64/hex, and the **document renderer** — `.odt`, `.ods`, `.docx`, `.xlsx` from templates, PDF through LibreOffice | all |
| `upi/` | VPA validation, pay links, a payment state machine whose rule is "pending is not failure" | retail, pharmacy, bakery |
| `vawki/` | Interest, loans and EMI schedules, asset classification | all |
| `nAtkAtti/`, `paNam/` | Dates, leap years, month ends, working-day calendars with holidays; rupee formatting with Indian digit grouping, paise-exact money | all |
| `qaLam/`, `cOqaZY/`, `atippatY/`, `kOppumuRY/` | Redis, the test framework, strings/math/arrays/records, the file system | all |

### 4.2 Platform capability

| Capability | State |
|---|---|
| HTTP server | `--server` (thread pool) and `--async` (tokio accept, blocking handlers). `வழி` routes with `:id`, query params, headers, bodies. `பதில்` with status, body, headers |
| Database | SQLite, **PostgreSQL** (money as native `NUMERIC`), MySQL. Always parameterised — no way to splice a value into SQL from eTamil. Transactions as plain `BEGIN`/`COMMIT`. Connection reuse from an idle cache |
| Auth | bcrypt, JWT HS256, and **RS256 SSO against a provider's JWKS** with issuer and audience required — which the MNC pilot will want |
| Files | Multipart uploads parsed over bytes, whole-file write, file responses for PDFs and documents |
| Crypto | HMAC-SHA256 constant-time, ECDSA P-256, XChaCha20-Poly1305 over Argon2id, mutual TLS |
| Outbound | `வலை_பெறு` / `வலை_பதி` / `வலை_அனுப்பு`; a non-2xx is a value, not a failure |
| Scheduling | `இடைவெளி N { … }` in-process, N being the gap *between* runs |
| Money | Fixed-point decimal everywhere. `0.1 + 0.2` is `0.3`. Equality is exact |
| Testing | `cOqaZY` with assertions, a summary and a non-zero exit, so a suite gates CI |
| Android | A packaged `etamil-android-arm64` build — which is what makes decision 5 cheap |

### 4.3 Two services that are already proto-modules

**`examples/katY`** (கடை, a shop) — 520 lines. A catalogue in SQLite, carts
priced with per-line GST, orders placed atomically against stock inside
`BEGIN`/`COMMIT`, a payment webhook whose signature is checked in constant
time, and the orders posted into the double-entry ledger.

**`examples/aluvalakam`** (அலுவலகம், an office) — 2,353 lines. Sign-in issuing
a JWT, three roles with per-route authorisation, user administration that
refuses to demote the last manager, SQLite persistence, documents rendered from
`.docx` templates and returned as files, an outbox worker that backs off and
parks a row after eight attempts rather than pretending to have sent it, and an
inbound webhook that applies a patch while refusing edits to fields it does not
own.

### 4.4 Conventions nivaqi inherits

Read `eTamil_lang/examples/aluvalakam/` before writing any nivaqi file.

| Convention | Form |
|---|---|
| File naming | `<module>_amYppu.qmz` schema and seed · `<module>_kaLam.qmz` persistence · `<module>_kYyALi.qmz` handlers · `<module>_cEvY.qmz` routes and workers · `<module>_cOqaZY.qmz` tests |
| Header | SPDX line, copyright line (section 14), then a `__…__` title comment naming the file and how to run it |
| Comments | English inside `__…__`, Tamil in plain text, both above every function |
| Configuration | `சூழல்("NAME", "default")` — environment only, with a default safe on a laptop and obviously wrong in production |
| Names | Stored exactly as typed. `{வரி: 1}` and `{vari: 1}` are **different fields**. One spelling per program, never mixed |
| Hard-reserved words | `எண்`, `சொல்`, `அணி`, `வரிசை`, `விதி`, `இடம்`, `உள்`, `வெளி`, `குழு`, `சேர்` cannot be names. Financial words are free |
| Name collisions | Two imported modules may not define one name — refused at load, with both file names. Run `scripts/check_names.py` before proposing any library function |

---

## 5. What does not exist

### 5.1 Language and runtime gaps

Most of what `eTamil_lang/docs/language-gaps-from-nuNNaRivu.md` found has since
been fixed — record key lookup, host JSON, string accumulation, logarithms,
sorting. What remains matters more for nivaqi than it did there, because nivaqi
is eighteen library families rather than one.

| Gap | Consequence | Mitigation |
|---|---|---|
| **No module namespaces, and nothing can be private.** Every helper is part of the public surface, and two modules defining one name is a hard load failure | The biggest scaling risk. Fourteen modules over a flat namespace, against 153 existing library modules, will collide repeatedly, and the failure arrives at load time far from the cause | Naming convention (section 8.1), `check_names.py` in CI from day one, and a language proposal for `இறக்கு "…" ஆக ச`. **Raise before wave 2** |
| **One database connection at a time**; `தளம்_வினா` names no handle | Rules out database-per-tenant, and rules out reading a replica in the same process | Shared schema with a tenant column, and one process per database for the customers who need their own (section 6.7). No longer a blocker (decision 0011) |
| **No language-level transaction construct** | `BEGIN`/`COMMIT` as plain SQL. Correct, but a handler returning early between them leaks an open transaction | `atiqqaLam/qaLap_parivarqqaZY.qmz` — one kernel wrapper, the only place in nivaqi that writes those words |
| **No static file serving** | The React build cannot be served by the eTamil process | nginx or Caddy in front, serving `/` and proxying `/api`. The right production shape anyway |
| **No SMTP** | Invoice email, portal invitations, password resets, overdue notices | A mail provider's REST API through `வலை_பதி`, routed via the outbox so a failed send is retried, not lost. No host primitive needed |
| **No updater** on a customer-hosted instance | Version skew, migrations on machines nobody can reach, sync-protocol compatibility | An updater with staged rollout, and schema and protocol versioning. Not designed or estimated (decision 0013) |
| **Local disk only** for attachments and generated documents | Fails on two or more nodes, which SaaS needs | Shared object storage with an S3-style client. The HMAC signing primitive exists in the host; the client does not. It is written in eTamil (decision 0014); the design is open |
| **Startup ≈ 1 s** with a large import set, and nivaqi's will be larger | Rules out any per-request process — already the chosen shape — but CI pays it per test file | Long-running server; group `cOqaZY` assertions per file |
| **No argv, no `--quiet`** | Admin and migration scripts take input through the environment | Acceptable. Worth raising |
| **LLVM backend refuses I/O statements** | nivaqi runs on the VM; no AOT-compiled deployment | Not a blocker. Noted so nobody plans a performance win around it |

### 5.2 Library gaps

Nothing in `nUlakam` currently addresses:

- **Identity** — users, credentials, sessions, roles, permissions, record rules. `aluvalakam` has an application-level version; it is not a library.
- **Partners** — party master, addresses, GSTIN/PAN validation, partner bank accounts, credit limits, duplicate detection.
- **Inventory** — units of measure and conversion, products and variants, lots and serials, locations, stock moves, reservations, and **perpetual valuation**. `celavu/` costs production; it does not value a warehouse.
- **Batch, expiry and FEFO** — required by the pharmacy, and by the bakery for finished goods.
- **Weight-based units and loose sale** — required by the dry fruit store.
- **Manufacturing** — bills of materials, production orders, yield and wastage. Required by the bakery, and partly by the dry fruit store for repacking.
- **Trade documents** — the quotation → order → delivery → invoice → payment chain and the matching rules between them. `qittam/nilYmARRam` gives lifecycles as data, which is the mechanism; the documents are missing.
- **Employee master and HR** — `kaNakkiyal/Uqiyam` has payroll *arithmetic*; nothing holds an employee, a contract, leave or attendance.
- **PoS** — terminal sessions, cash drawer, offline queueing, receipts; and restaurant mode on top.
- **Kernel pieces** — sequences, settings, attachments, i18n, audit trail (section 6.3).

### 5.3 The persistence pattern in the examples will not scale

`aluvalakam_kaLam.qmz` stores a record as JSON in a `qaravu` column keyed by
`kuRi`. Fine for a few hundred projects. Wrong for an ERP: it cannot be
indexed or aggregated, and every report becomes a full scan and a parse in the
VM.

**Rule:** anything queried, filtered, sorted, aggregated or reported on is a
typed column. JSON is for extension fields only, and an extension field that
acquires a report becomes a column in the next migration.

---

## 6. Architecture

### 6.1 The layering rule

| Layer | Written in | Contents |
|---|---|---|
| 0 — Runtime primitives | Rust | Sockets, drivers, decimal, clock, crypto |
| 1 — Language capability | Rust | Functions, collections, results, modules |
| 2 — Frameworks | **eTamil** (`nUlakam`) | The standard library, accounting, costing, tax, **and every nivaqi business rule** |
| 3 — Application | **eTamil** (`nivaqi`) | Routes, persistence mapping, orchestration, response shaping |
| 4 — Interface | TypeScript / React | Presentation only. No rules |

**The backend is eTamil by default** (decision 0014): every server-side and
operational program nivaqi authors, including migrations, seeds, sync, the
tenant export tool and the tests. When eTamil has a gap, the gap is filled by
building the capability into eTamil in `eTamil_lang`; another language is
chosen only when that would be huge or complicated, and each exception is
recorded in 0014. The recorded exceptions are the React front end, SQLite and
PostgreSQL, and infrastructure products such as PgBouncer and a reverse
proxy. The databases are SQLite on the edge and PostgreSQL in the cloud.

The applied test transfers from `eTamil_lang/docs/ARCHITECTURE.md`: rounding
went into the host because it needs the decimal representation; Indian digit
grouping stayed in eTamil because string manipulation is expressible. For
nivaqi: **a credit limit check is Layer 2 even though it reads a database
value** — pass it the balance and the limit, not a connection.

### 6.2 Repository layout

```
nivaqi/
  README.md  AUTHORS  LICENSE
  docs/
    ARCHITECTURE.md          this document
    DECISIONS/               one numbered file per decision
    API.md                   generated route index across all modules
    DEPLOY.md
  atiqqaLam/                 the kernel (6.3)
  payiRci/                   seeds and demo data per pilot
  <module>/                  one directory per module
    README.md
    MODEL.md  API.md  RULES.md  LIBRARY.md  DECISIONS.md
    <module>_amYppu.qmz      schema and migrations
    <module>_kaLam.qmz       persistence
    <module>_kYyALi.qmz      handlers
    <module>_cEvY.qmz        routes and workers
    <module>_cOqaZY.qmz      tests
    uqavi/                   runnable worked examples
  mukappu/                   the React application
  itam/                      migrations, as numbered SQL
  karuvi/                    eTamil programs: migrate, seed, export, check
  .github/workflows/
```

**One process or several?** Start with one `--server` process mounting every
module's routes. A process per module is a deployment per module, and the module
boundaries are not yet proven. The layout keeps splitting cheap: a module is
already a directory with its own routes, schema and tests. Split when a module
has a different scaling or availability profile — PoS is the one that actually
will.

### 6.3 The kernel: `nUlakam/atiqqaLam/` (அடித்தளம், foundation)

**Build the kernel first and completely, before module two.** If each module
invents its own CRUD, list filtering, permission check and audit row, there is
no minimal code — there are eighteen copies of the same 400 lines, and the
fourth is subtly different from the first.

Equally: **do not build Odoo's kernel.** Odoo's power comes from a dynamic ORM
with declarative fields, inheritance, computed/stored fields, record rules and
XML views, which together let a module be a data file. That engine is years of
work, it is where Odoo's performance problems live, and in a language with no
namespaces and no reflection it would reimplement everything Layer 1
deliberately does not do. The research claim is that eTamil is sufficient as
written; a metaprogramming layer would argue it is not.

The middle path is **thin, explicit and boring**:

| Module | Tamil | What it holds |
|---|---|---|
| `atiqqaLam/paqivEtu.qmz` | பதிவேடு | The record contract: a `வடிவம்` plus its table, key, columns and extension field; row → record and record → parameters, in one place |
| `atiqqaLam/vatikattal.qmz` | வடிகட்டல் | List queries — filter, sort, paginate — compiled to **parameterised** SQL. Every list screen goes through this, so every list screen gets the same semantics and the same injection safety |
| `atiqqaLam/aZumaqi.qmz` | அனுமதி | Roles, permissions and record rules: can this user do this verb to this record in this company and tenant |
| `atiqqaLam/kaNkANippu.qmz` | கண்காணிப்பு | The append-only audit trail: who, what, before, after, when. Never updated, never deleted |
| `atiqqaLam/qotareN.qmz` | தொடரெண் | Numbering sequences: per tenant, per company, per document type, per fiscal year, gapless where the statute requires it |
| `atiqqaLam/caripArppu.qmz` | சரிபார்ப்பு | Validation results that accumulate rather than stop at the first failure, so a form returns every error at once |
| `atiqqaLam/nikazvu.qmz` | நிகழ்வு | Domain events, written in the same transaction as the change, drained by `oruwkiNYppu/aZuppu_varicY` |
| `atiqqaLam/amYvu.qmz` | அமைவு | Settings with layered defaults — system, tenant, company, user. `qittam/AvaNac_cUzal` already does this for documents; extend it rather than write a second |
| `atiqqaLam/kOppiNYppu.qmz` | கோப்பிணைப்பு | Attachments: content-addressed storage, type and size limits, and the rule that an attachment never determines access to the record it hangs on |
| `atiqqaLam/mozipeyarppu.qmz` | மொழிபெயர்ப்பு | Translation lookup for labels, document types and error messages, Tamil and English |
| `atiqqaLam/qaLap_parivarqqaZY.qmz` | தளப் பரிவர்த்தனை | A database transaction: takes a `செயல்`, wraps `BEGIN`/`COMMIT`, rolls back on a `தவறு`. Qualified, because `parivarqqaZY` alone already means an accounting transaction in `kaNakkiyal` |

Roughly 1,300–1,700 library lines. Everything else sits on it.

**The kernel assumes no always-on cloud** (decision 0013). Nothing in it may
need a connection to a cloud service to complete a request; anything that
must reach the cloud goes through the outbox.

### 6.4 The data model rules

Nine rules, each cheap now and very expensive to retrofit.

1. **The ledger is the single source of truth for money.** Every module that
   moves value posts a balanced journal entry through `kaNakkiyal/pErEtu`. No
   module keeps its own running total as the authority. `katY` already
   demonstrates this; it is what makes the three statements correct by
   construction rather than by reconciliation.
2. **The journal is immutable.** A correction is a reversing entry with a
   reason and a link, never an `UPDATE`. Statutory in India, and it is what
   makes the audit trail worth having.
3. **Master data is effective-dated.** `kaNakkiyal/vari_vikiqam` and
   `vaLam/vaLam` already work this way: every lookup takes the date it is asked
   about. Apply to prices, tax codes, UoM conversions, payroll components,
   exchange rates and recipes. An ERP that cannot reprice last quarter cannot
   restate last quarter.
4. **Nothing user-facing is hard-deleted.** Archive with a flag and a reason.
   A deleted customer with three years of invoices is a corrupted ledger.
5. **Every row carries tenant, company, created-by/at and modified-by/at.**
   Written by the kernel, never by modules.
6. **Money is an integer count of the currency's minor unit** — `BIGINT` on
   PostgreSQL, `INTEGER` on SQLite — never `NUMERIC`, never text, never a
   float (decision 0015). One representation on both engines, so sums,
   ordering, comparison, `MAX` and indexes are exact and correct everywhere,
   and a tenant's data moves between engines unchanged. A text money column
   sorts and compares wrongly in SQLite, which is why the earlier rule, text
   on SQLite and `NUMERIC` on PostgreSQL, was replaced. In memory the ledger
   still works in exact decimals; the conversion happens once, in the
   persistence layer, and refuses an amount with more precision than the
   currency's minor unit instead of rounding it. Rates, which are never summed
   or ordered in SQL, stay exact decimal text. Quantities have the same
   problem and are open (0015).
7. **Document state is data, not code.** Use `qittam/nilYmARRam` — a lifecycle
   as a transition table — for draft → confirmed → done → cancelled on every
   document type. Transitions are then inspectable, testable and renderable,
   and adding a state is not editing a handler.
8. **Quantity carries its unit.** Never a bare number. The dry fruit store
   sells by weight and buys by sack; the bakery issues by gram and counts by
   tray. A quantity without a unit is a defect waiting for the first
   conversion.
9. **Identifiers are globally unique.** UUIDv7 primary keys, or composite
   `(tenant, id)` keys. A row must be able to move between the pooled tier,
   a silo and a restored copy without its identifier colliding with another
   row's. Per-database serials would make tenant restore, merge and the move
   to a silo painful. Decide this before any schema exists. How UUIDv7 is
   generated (a database default, or the host) is unverified and is an M0
   spike item.

### 6.5 The HTTP API contract

| Concern | Decision |
|---|---|
| Shape | `/api/v1/<module>/<resource>`, JSON in and out. The version is there from the first commit — adding one later means touching every client |
| Authentication | `Authorization: Bearer <jwt>`. Tokens carry user, tenant, company and role; eight hours. RS256 SSO where the customer has an identity provider |
| Authorisation | Every handler's first call after parsing is `aZumaqi`. Never implicit |
| Errors | One body shape with a stable machine code, a bilingual message, and the field it concerns. Status codes mean what they say — 409 for a duplicate key, not 500 (`தளம்_செய்_முயற்சி` exists precisely so this is possible) |
| Idempotency | Every mutating call takes an `Idempotency-Key`. `oruwkiNYppu/kYrEkY` computes stable fingerprints. Not optional for PoS or payments |
| Lists | Uniform `?vati=…&varicY=…&pakkam=…` through `vatikattal`, with a total count |
| Documentation | `API.md` per module, plus a generated index. A route not in `API.md` fails CI |

### 6.6 The front end

React 19 + Vite + TypeScript, Tailwind, over the JSON API — the pattern
`eTamil_lang/docs/studio/MVP.md` already assumes for every eTamil demo
application.

- **nginx or Caddy serves the build and proxies `/api`.** The eTamil server has
  no static file serving, and that is the wrong place for it.
- **Presentation only.** No totals computed in the browser, no tax rates in
  TypeScript, no state machine duplicated in a reducer. If a number appears on
  screen, the server computed it.
- **One component library and one data table, used everywhere.** An ERP is
  fifty list screens and eighty forms; the leverage is in having one of each.
- **Tamil and English from the first screen**, through
  `atiqqaLam/mozipeyarppu`. Retrofitting i18n across eighty forms is a project
  of its own, and in this product it would be an embarrassing one to postpone.
- **The PoS is not this application.** See 7.3.

### 6.7 Tenants, companies and the SaaS shape

Decided in [0011](DECISIONS/0011-saas-architecture.md), which supersedes 0006.
The reasoning, the options not taken, and the backup design are there; this is
the summary a developer needs.

**Multi-company** — every table carries a company column, every query is scoped
by the kernel, and `kaNakkiyal/niRuvaZam` already handles entities, base
currency and conversion. Consolidation is a reporting concern on top, not a
second database.

**Multi-tenant SaaS** — a modular monolith: one codebase, one shared schema,
modules sold as entitlements, deployables split by load profile, and a separate
database only for customers who need one.

- Every row carries a tenant as well as a company, written by the kernel. No
  query reaches the database except through `vatikattal`, so the tenant
  predicate is applied in one place. PostgreSQL row-level security is a second
  lock, set with `SET LOCAL` inside `qaLap_parivarqqaZY`.
- **Modules are entitlements.** A control-plane table maps tenant → plan →
  modules, enforced once in `aZumaqi` at route entry. A switched-off module
  keeps its data.
- **Pools split by load profile, not popularity.** A pool is an entry file that
  imports a subset of route files (`etamil --server pool_core.qmz`), on the same
  binary and database. Candidates: core, PoS sync, reports (pointed at a read
  replica) and jobs. Do not build pools speculatively; start with one.
- **Silos on request.** One process is one database, so a customer who needs
  isolation gets a database and a small process group of its own. Keep them
  few: migrations run per database.
- **Three placements** — cloud pool, cloud silo and **edge**, where the
  authority runs on the customer's premises and the cloud is used for
  backup, updates and optional sync. Same code, one authority per company,
  no multi-master merge (decision 0013).
- No cross-tenant sequences, and no process-wide cache keyed without a tenant.
- **No platform vocabulary reaches the customer** (section 1, product
  principles): a company, never a tenant, pool or silo.

**Backup** has two parts: platform-wide point-in-time recovery, and a
tenant-scoped export tool for restoring one customer. A tenant is restored into
a side copy by default, because statutory documents have already left the
system. See 0011.

**Not yet measured:** throughput per core with PostgreSQL, memory under load
with the full import set, and row-level security through the eTamil driver. No
customer-count claim is made until they are.

### 6.8 Identity, roles and record rules

Three layers, kept apart:

1. **Authentication** — who. bcrypt with a real policy, JWT sessions, RS256 SSO
   against a provider's JWKS (already in the host, and the MNC will want it),
   API keys for integrations, sign-in attempts in the audit trail.
2. **Authorisation** — what verb on what model. Role → permission, coarse.
3. **Record rules** — which rows. "A salesperson sees their own customers"; "a
   vendor sees only their own purchase orders". These compose as additional
   `WHERE` predicates produced by `aZumaqi` and consumed by `vatikattal`, so a
   developer cannot forget one by writing a query by hand — which is the only
   reason list queries are centralised.

`aluvalakam` shows the discipline worth copying: the last manager cannot be
demoted, a password reset signs the user out everywhere, and user listings
never return password hashes.

The MNC pilot will raise segregation of duties — the person who creates a
vendor may not approve its payment. That is a rule over *pairs* of permissions
and it belongs in `aZumaqi` from the start, not bolted on.

### 6.9 Documents and reports

`vativam/AvaNam` fills `.odt`, `.ods`, `.docx` and `.xlsx` templates and
converts to PDF through LibreOffice; `பதில்_கோப்பு` returns a file as a
response body. That is the invoice/receipt/statement pipeline, already built.

- **Templates are data**, in the database with a version and an effective date,
  not files in the repository — an invoice layout changes without a deployment.
- **No report designer.** Reports are eTamil functions returning records, plus
  a template. A drag-and-drop builder is a product in its own right, and it is
  one of the three the owner is keeping private (section 14).
- **CSV and XLSX export on every list screen**, from the same `vatikattal`
  query as the screen, so export and screen cannot disagree.
- **PDF conversion is out of process** and fails loudly. `கட்டளை_ஓட்டு` is
  deny-by-default and needs `ETAMIL_EXEC_ALLOW`; keep it that way.

### 6.10 Jobs and integration

- In-process `இடைவெளி` for the outbox drain and short periodic work, as
  `aluvalakam_cEvY.qmz` does.
- **The outbox is the only way anything leaves nivaqi** — written in the same
  transaction as the change it reports, drained with backoff, parked after N
  attempts, never silently dropped. `oruwkiNYppu/aZuppu_varicY` and
  `oruwkiNYppu/mItci` already implement this.
- **Inbound webhooks verify a signature in constant time** and are idempotent
  on a fingerprint. `katY` and `aluvalakam` both show the shape. The bakery's
  delivery-aggregator orders will arrive this way.
- Long or heavy jobs — payroll for the MNC, a year-end close, a stock
  revaluation — run in a separate worker process against the same database, so
  a long job cannot occupy a request worker.

---

## 7. The module plan

### 7.1 Dependency order

```
  M0  atiqqaLam      kernel
   │
  M1  atYyALam       identity, roles, record rules
   │
  M2  pawkALi        partners
   ├──────────────┬──────────────┐
  M3 kaNakku    M7 paNiyALar   M4 carakku
   │              │              │
   │             M8 nEraqqAL    M5 vaNikam      sales & purchase
   │              │              │
   └──────┬───────┘             M6 muZYyam      point of sale
          │                      │
         M9 maZiqavaLam         M4b batch/expiry · M4c weight UoM
          │                      │
        M10 vAcal               M13 uRpaqqi     manufacturing
          │                      │
        M11 miZvaNikam          M6b restaurant PoS
          │
        M12 iNYyaqaLam          website — see decision 4
```

### 7.2 The waves, driven by the pilots

| Wave | Modules | Goes live |
|---|---|---|
| **1** | M0 kernel · M1 identity · M2 partners · M3 accounting · M7 employee · M8 timesheet · project billing | **IT consulting MNC** |
| **2** | M4 inventory · M4c weight UoM · M5 sales & purchase · M6 PoS (retail, offline) | **Dry fruit retail** |
| **3** | M4b batch, expiry and FEFO · `marunqakam` regulated items | **Pharmacy** |
| **4** | M13 manufacturing · M6b restaurant PoS | **Bakery restaurant** |
| **5** | M9 HR & payroll · M10 portals · M11 eCommerce · M12 website | — |

**Why the MNC first.** It is the only pilot that needs neither Inventory nor
PoS, it needs the least new library code, and it exercises the parts of
`nUlakam` that are already strongest — `varuvAy/` for revenue recognition,
`vaLam/nEraqqAL` for timesheets, `vaLam/paqivu` for project cost into the
ledger, `qittam/` for the projects themselves. Wave 1 is the shortest path from
zero to a real business running on nivaqi, which is the only thing that
validates any of this.

**A staged go-live inside wave 1** is worth considering: books and invoicing
first (M0–M3), timesheets and project billing second. It puts a real user on
the system months earlier, at the cost of the MNC running two systems for a
while — which is their call, not ours.

**Before anything else, build one module end to end.** Partners is the right
choice: smallest domain, every kernel capability exercised — CRUD, list
filtering, permissions, audit, sequences, validation, attachments, i18n,
import/export, documents, tests, React screens. Ship it, use it, and only then
start the rest. A kernel proven on one real module is worth more than one
designed for eighteen imagined ones.

### 7.3 The PoS is a separate application

Decision 5 was accepted, and it is an architecture rather than a feature.

Because there is a packaged Android arm64 build and SQLite is in the default
feature set, a terminal runs **a real eTamil process locally against a local
SQLite database, holding the same `nUlakam` rules as the server** — the same
tax arithmetic, the same price resolution, the same receipt numbering — and
synchronises through the outbox pattern `oruwkiNYppu/` already implements.

Most offline PoS implementations reimplement price and tax in the client and
drift from the server. Here that is avoided by construction, and it is a claim
no other ERP can make.

Three consequences to accept now:

- Receipt numbers are gapless **per terminal**, not globally.
- Stock at a terminal is advisory until it syncs.
- A line sold twice while two tills were offline needs a **commercial** rule —
  refund, backorder, or honour and go short. That is the owner's decision, not
  the developer's, and it is written into `muZYyam/RULES.md` before wave 2
  starts.

---

## 8. The library plan

What each module contributes to `nUlakam`. The module is the thin part; this is
the thick part.

### 8.1 The naming convention, which is not optional

With no namespaces, a flat global surface and a hard load failure on any
collision, names are governed from the first function:

- **Every public function in a new family is prefixed by its family's noun**
  where the name is not already unmistakable: `carakku_alaku_mARRu`, not
  `alaku_mARRu`.
- **A helper that is not part of the contract gets a `_` prefix** and is
  documented as private by convention. The language cannot enforce it; review
  can.
- **`scripts/check_names.py` runs in CI** on every library PR, against every
  name already in `nUlakam`. It is a Python tool of 144 lines in `eTamil_lang`, a candidate to port to
  eTamil under decision 0014's policy; not decided.
- **No name may be a hard-reserved word** (section 4.4). Financial keywords are
  free — `தொகை` is a good name for an amount.

### 8.2 Per module

| Module | `nUlakam` family | Function groups |
|---|---|---|
| **M0 kernel** | `atiqqaLam/` | Section 6.3 — eleven modules, ~1,500 lines |
| **M1 Identity** | `atYyALam/` | Password policy and strength · session issue, refresh and revoke · role and permission resolution · **segregation-of-duties pairs** · record-rule predicates · API keys with scopes · sign-in throttling · consent and terms |
| **M2 Partners** | `pawkALi/` | Party master (customer / vendor / both / employee) · addresses with state codes for GST place-of-supply · **GSTIN checksum, PAN format, IFSC validation** · **drug licence and FSSAI numbers** for the pharmacy and bakery · contact methods · partner bank accounts · categories · credit limit and exposure · duplicate detection by name, GSTIN and phone · merge rules |
| **M3 Accounting** | extend `kaNakkiyal/` | Customer invoice and vendor bill as documents · credit and debit notes · payments and **matching** (`oqukkItu` has allocation and ageing; matching is the missing half) · bank statement import and reconciliation · **e-invoice IRN payload and QR** · **export invoicing, LUT and zero-rated supply** for the MNC · GSTR-1/3B summarisation · TDS on purchase via `nErativari/mUlavari` · recurring entries · multi-currency revaluation · fixed-asset register on `qEymAZam` |
| **M7 Employee** | `paNiyALar/` | Employee master and lifecycle · contracts with effective dates · departments and reporting lines · skills · documents with expiry (visa, licence) · onboarding and exit checklists |
| **M8 Timesheet** | extend `vaLam/nEraqqAL` | Approval workflow · billable versus internal · **T&M and fixed-price billing rules** · **project cost posting** (`vaLam/paqivu` exists) · overtime · utilisation · the bridge to `varuvAy/` for revenue recognition |
| **M4 Inventory** | `carakku/` | **UoM and conversion with a reference unit per category** · product, variant and attribute · barcodes · locations and warehouse hierarchy · **stock moves and quants** · reservation and availability · **valuation: FIFO, weighted average, standard, each posting to the ledger** · reordering rules · stock counts and adjustments with a reason · landed costs via `cuwkam` |
| **M4b Batch & expiry** | extend `carakku/` | Lots and serials · **expiry dates and FEFO picking** · near-expiry reporting and alerts · expiry write-off posting to the ledger · supplier returns of expired stock · shelf life on manufactured goods |
| **M4c Weight units** | extend `carakku/` | **Weight and volume units with loose sale** · price per kg with tare · **variable-weight barcodes** (weight embedded in EAN-13) · scale input · shrinkage and wastage · repacking bulk into retail packs |
| **M5 Sales & Purchase** | `vaNikam/` | The document chain as lifecycles on `qittam/nilYmARRam` · quotation, order, delivery, invoice and their matching rules · **price lists, effective-dated, with quantity breaks** · discount rules · **three-way match** (order / receipt / bill) · returns and RMA · blanket orders · margin on a line and on a document |
| **M6 PoS** | `muZYyam/` | Terminal session open/close · **cash drawer declaration and variance** · receipt numbering, gapless per terminal · split tender across cash, card and UPI (`upi/` exists) · **offline queue and replay with conflict rules** · price and tax resolution at the terminal · shift and Z-report |
| **M6b Restaurant PoS** | extend `muZYyam/` | **Tables and covers** · **kitchen order tickets** and course timing · split and merge bills · modifiers and combos · dine-in versus takeaway versus delivery tax treatment · aggregator orders inbound through the webhook pattern |
| **M13 Manufacturing** | `uRpaqqi/` | **Bills of materials**, multi-level, effective-dated · **production orders** and their lifecycle · issue and receipt against an order · **yield, scrap and wastage** · recipe costing bridged to `celavu/` process costing · daily production planning against shelf life · by-products |
| **Pharmacy** | `marunqakam/` | **Drug schedules (H, H1, X)** and what each requires at sale · prescription capture and the H1 register · substitutes and generic equivalence · drug licence validation on company and supplier |
| **M9 HR & Payroll** | `maZiqavaLam/` + extend `kaNakkiyal/Uqiyam` | Leave types, accrual and balances · attendance and shifts · **salary structures as effective-dated component rules** · PF, ESI, professional tax, gratuity · payslips through `vativam/AvaNam` · Form 16 inputs via `nErativari/` · loans and advances via `vawki/kataZ` · the payroll run as a closable period, posting to the ledger |
| **M10 Portals** | `vAcal/` | Portal identity, separate from staff identity · invitation and activation · scoped record rules · document sharing with expiring links · portal-side approvals · **recruitment**: posting, application, pipeline, interview scheduling, offer |
| **M11 eCommerce** | `miZvaNikam/` (much from `katY`) | Catalogue publication rules · cart with per-line GST (exists) · checkout and payment through `upi/` and a gateway · order placement against stock, atomically (exists) · shipping rules and rates · coupons · abandoned cart · bridge to `vAcal` |
| **M12 Website** | — | Decision 4: static site generator, not a CMS. A small content module only if non-developers must edit pages |

---

## 9. What each module ships

A module is done when all nine exist. Not eight.

| # | Deliverable |
|---|---|
| 1 | `nUlakam` library family, bilingual comments on every function |
| 2 | One runnable `uqavi/` example **per public function** — the convention `nUlakam` holds at 937 |
| 3 | `cOqaZY` suite covering every rule, exiting non-zero on failure |
| 4 | Migrations in `itam/`, numbered, forward-only, each reversible or explicitly not |
| 5 | Handlers and routes, within section 3.1's limits |
| 6 | React screens: list, form, and the module's documents |
| 7 | The document set (section 11) |
| 8 | Seed data in `payiRci/` sufficient to demonstrate the module without typing |
| 9 | An end-to-end worked script, in the shape of `aluvalakam.qmz`: the module's whole story as one runnable program |

Number 9 is worth defending. `aluvalakam.qmz` is the best documentation in the
eTamil repository — executable, so it cannot go stale, and it demonstrates the
libraries composing rather than each working alone.

---

## 10. Testing and quality gates

| Phase | What | Tool |
|---|---|---|
| 1. Library | Every rule, as a pure function. No database, no server | `cOqaZY` |
| 2. Persistence and API | Routes against a real database with seeded data | `cOqaZY` + a test database |
| 3. Cross-module | Order → delivery → invoice → payment → ledger as one script, asserting the **trial balance still balances** | A worked script per flow |
| 4. Interface | Critical journeys only: sign in, create an invoice, take a payment, close a day on the PoS | Playwright |

**The gate that matters most:** after every cross-module test, assert that the
trial balance balances and that no journal row was updated. An ERP whose ledger
silently stops balancing is worse than one that crashes, for the reason
`eTamil_lang/docs/ARCHITECTURE.md` gives for failing loudly: nobody
investigates a success.

CI on every PR: `--check` on every `.qmz`, the full `cOqaZY` suite,
`check_names.py`, migrations applied forward from empty, the API-route/`API.md`
drift check, `npm audit` on the front end, and the code ratio from section 3.1
reported as a comment.

---

## 11. The documentation standard

Per module, seven files, each with one job:

| File | Answers |
|---|---|
| `README.md` | What this module is, its boundary, **what it deliberately does not do**, and its code ratio |
| `MODEL.md` | Entities, fields, types, keys, indexes, invariants, and every lifecycle as its transition table |
| `API.md` | Every route: method, path, auth, permission, request, response, every error code |
| `RULES.md` | Every business rule, numbered, each with its source — a section of the GST Act, an Ind AS paragraph, a drug schedule, or "a decision we made, here is why" |
| `LIBRARY.md` | Which `nUlakam` families it uses, what it added, and what it deliberately did not add |
| `DECISIONS.md` | Numbered: what was decided, what was rejected, why |
| `uqavi/` | Runnable worked examples, one per public function |

Two rules about prose, taken from the eTamil repository's own documentation:

1. **Record what was found to be wrong, not only what is right.**
   `language-gaps-from-nuNNaRivu.md` has a section titled "A wrong diagnosis,
   recorded because it was costly". It is the most useful thing in the file.
2. **Say what a thing is not.** `alaku_pawkaLippu.qmz` spends two comment lines
   saying contribution is not profit. In an ERP, where every number is about to
   be read by an accountant, that habit is worth more than it costs.

---

## 12. Sequencing and effort

One developer with AI assistance, eTamil and `nUlakam` as they stand, and
**20–30% of capacity reserved for supporting each pilot already live**
(section 2).

| Wave | Scope | Weeks |
|---|---|---|
| **1** | M0 kernel 6–8 · M1 identity 3–4 · M2 partners 4–5 · *review gate* 1 · M3 accounting 8–9 · M7 employee 4–5 · M8 timesheet and project billing 5–7 | **31–39** |
| | → **IT consulting MNC live** (or books-only at ~22–27) | |
| **2** | M4 inventory 8–10 · M4c weight units 2–3 · M5 sales & purchase 6–8 · M6 PoS offline 8–10 | **24–31** |
| | → **Dry fruit retail live** | |
| **3** | M4b batch, expiry, FEFO 3–4 · `marunqakam` 2–3 | **5–7** |
| | → **Pharmacy live** | |
| **4** | M13 manufacturing 5–6 · M6b restaurant PoS 3–4 | **8–10** |
| | → **Bakery restaurant live** | |
| **5** | M9 HR & payroll 8–10 · M10 portals 8–10 · M11 eCommerce 6–8 · M12 website 2–3 | **24–31** |
| | **Total** | **92–118** |

**These totals exclude the SaaS control plane** (decision 0011): tenant
registry, entitlements, provisioning and metering, with billing through a
payment provider. My estimate is 4–6 weeks for a minimal version on top of
wave 1. That is a guess, not a measurement, and it moves the first go-live
by about the same amount.

Roughly two to two and a half years for one developer — longer than the first
estimate, because the four pilots added weight units, batch and expiry,
manufacturing and restaurant PoS, none of which was in the original module
list. What improved is **time to the first real user: about 31–39 weeks instead
of 48–60**, or half that for a books-only go-live.

The review gate after Partners is the important row. If the code ratio is above
25% there, something in the kernel is wrong, and the right response is to fix it
before wave 1 continues — one module to migrate rather than four.

---

## 13. Risks

| Risk | Likelihood | Impact | Response |
|---|---|---|---|
| **Four live pilots starve the build** | High | High | Stagger go-lives, one per wave; reserve 20–30% capacity; a second person rather than a longer schedule if they must be concurrent |
| **Name collisions across a flat namespace** | High | High | Convention + CI from day one; raise namespaced imports with the language before wave 2 |
| **The kernel is wrong, found at module four** | Medium | High | The review gate after Partners exists for this |
| **VM performance on real volumes** — a 50,000-line trial balance, an MNC payroll, a year-end close | Medium | High | **Spike this in M0**, before the kernel is finished: measure a 100k-row aggregation and a 10k-line report. The answer decides how much goes into SQL aggregation versus eTamil iteration, which is a kernel decision |
| **Pilot requirements were inferred, not gathered** | High | Medium | Confirm with each business before its wave starts. Section 2 is a planning input, not a specification |
| **Pharmacy regulatory exposure** | Medium | High | Schedules and registers are statutory. Confirm scope with the pharmacy and a pharmacist before wave 3; `RULES.md` cites the rule for each |
| **Startup time with a large import set** | Medium | Medium | Measure in M0 with 60 imports. It decides whether modules can be separate processes |
| **Compliance changes mid-build** | High | Medium | Rates in effective-dated tables (`vari_vikiqam` already works this way; **no rate is seeded**), never in code |
| **Two-repository friction** | Medium | Medium | Section 3.3's policy, and `ETAMIL_PATH` for the inner loop |
| **The front end accumulates rules** | High | Medium | Review rule: any calculation in TypeScript is a defect |
| **A fleet of customer-hosted edge instances** — no updater, no way to reach the machines | High | High | Updater, staged rollout and versioned schema and sync protocol before the first edge customer (decision 0013) |
| **A customer's edge device dies** and the off-site backup is stale | Medium | High | Encrypted off-site backup, restored on a schedule in testing; backup is part of the product, not an add-on (decision 0013) |
| **Noisy neighbours in the shared pool** | Medium | Medium | Per-tenant rate limits at the edge, statement timeouts, jobs and reports in their own pools, and a silo for anyone who needs isolation (decision 0011) |
| **Restoring a tenant breaks statutory records** — IRNs, e-way bills and payments have already left the system | Medium | High | Restore into a side copy by default; overwrite only with the tenant frozen, then reconcile against the outbox (decision 0011) |
| **Single developer** | — | High | The documentation standard *is* the mitigation. That is why section 11 is not optional |

---

## 14. Decisions, licence and authorship

All ten founding decisions were taken on 2026-10-24 and are recorded
individually in [DECISIONS/](DECISIONS/):

| # | Decision |
|---|---|
| [0001](DECISIONS/0001-the-name.md) | நிவதி — `nivaqi`, an acronym of niRuvaZa vaLa qittamital |
| [0002](DECISIONS/0002-compliance-scope.md) | Books, e-invoicing, e-way bills, audit trail. Returns summarised, not filed |
| [0003](DECISIONS/0003-sales-and-purchase.md) | Sales & Purchase is a first-class module |
| [0004](DECISIONS/0004-website-and-blog.md) | No CMS in eTamil; static site plus a React storefront |
| [0005](DECISIONS/0005-pos-offline-first.md) | PoS is offline-first, eTamil on Android |
| [0006](DECISIONS/0006-multi-tenancy.md) | Multi-tenant data model now; SaaS product surface later — **superseded by 0011** |
| [0007](DECISIONS/0007-pilots.md) | Four pilots, staggered, MNC first |
| [0008](DECISIONS/0008-repositories-and-licence.md) | All nivaqi repositories public; AI, BI and Studio private |
| [0009](DECISIONS/0009-where-this-document-lives.md) | This document lives here |
| [0010](DECISIONS/0010-authorship.md) | Authored as Isha Maruff and Esan Maruff |

Decisions taken after the founding ten:
[0011](DECISIONS/0011-saas-architecture.md), the SaaS architecture and backup
(section 6.7); [0012](DECISIONS/0012-product-principles.md), product
principles (section 1); [0013](DECISIONS/0013-edge-placement.md), the
edge placement and local-first with a single authority (sections 6.3 and
6.7); [0014](DECISIONS/0014-databases-and-language.md), the databases by
placement and eTamil as the default backend language (section 6.1); and
[0015](DECISIONS/0015-money-as-integer-minor-units.md), stored money as
integer minor units (section 6.4).

### Licence

**AGPL-3.0-or-later**, matching `eTamil_lang`. nivaqi imports `nUlakam` at
runtime, both are distributed, and all nivaqi repositories are public — so
AGPL is both the consistent choice and the one that raises no question.

**The three private products do raise it.** AI, BI and the Customizing Studio
are planned as private and will build on AGPL `nUlakam` and AGPL nivaqi.
Whether a private work that imports AGPL libraries at runtime is itself subject
to AGPL is a real question, and it wants a licensing opinion rather than a
developer's guess — **before those products start, not after**.

Two things make that easier, and one of them is time-limited.
`eTamil_lang/AUTHORS` records a sole copyright holder and no contributors yet,
and states that copyright is not assigned to the project — a contributor keeps
their own. So dual-licensing `nUlakam` is entirely the owner's to do **today**,
and that option closes the moment the first external contribution lands without
a contributor licence agreement in place. If a commercial licence on the
libraries might ever be wanted, decide on a CLA before accepting a
contribution.

### Authorship

Every nivaqi source file carries:

```
// SPDX-License-Identifier: AGPL-3.0-or-later
// Copyright (C) 2026 Isha Maruff and Esan Maruff
```

and `AUTHORS` records both, in full, with the same care
`eTamil_lang/AUTHORS` takes over names. Documentation, the README, release
notes and package metadata name both.

Git commit authorship is a separate matter and is the owner's to configure;
nothing in this repository changes it.

---

## 15. Where to start

1. **Run the performance spike** (risks, row 4). A few days. It is the only
   thing that could invalidate the architecture, and it is cheap to find out
   now. Decision 0011 adds to it: requests per second per core with
   PostgreSQL under a real handler; memory under load with the full import
   set; row-level security through the driver with `SET LOCAL` under
   PgBouncer transaction pooling; how UUIDv7 is generated; and whether
   bulk copy is available for the tenant export tool. Decision 0013 adds what
   happens with several terminals writing to one SQLite file through one
   eTamil process. Decision 0015 adds whether integer money round-trips
   through eTamil's driver as `INTEGER` and sums exactly on both engines. Decision 0014 adds two more: how fast the tenant export
   runs when written in eTamil, and whether eTamil can generate UUIDv7 (it has
   a seconds clock, and no general UUID or random builtin was found).
2. **Confirm the four pilots' real requirements** (section 2). A conversation
   each. The table is inferred and some of it will be wrong.
3. Set up CI, migrations, the React shell and deployment. Deploy an empty
   application that can sign nobody in.
4. Write `atiqqaLam/` — all eleven modules, with `uqavi/` examples and tests.
5. Write Partners on top of it, all nine deliverables, and *use* it.
6. Hold the review gate honestly. If the kernel is wrong, it is wrong now and
   cheap.

Wave 1's remaining modules are then a repeating exercise rather than a series
of inventions — which is the whole point of spending six weeks on a kernel
before writing a line of business code.

---

*This document will be wrong in places. The places it is wrong are worth
recording in [DECISIONS/](DECISIONS/) as they are found, in the style the
eTamil repository already uses — so that a decision is not relitigated, or
accidentally reversed.*
