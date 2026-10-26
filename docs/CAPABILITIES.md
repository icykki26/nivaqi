# Capabilities to build

nivaqi's backend is eTamil by default
([decision 0014](DECISIONS/0014-databases-and-language.md)). When eTamil has a
gap, the gap is filled by building the capability into eTamil, in
`eTamil_lang`, unless that would be huge or complicated. This is the list of
gaps found so far, how sure each finding is, and when each is needed.

Written 2026-10-26. Checked against `eTamil_lang` at `7df1fbd`, compiler 1.4.2.

## How to read it

**Evidence**

- ✔ confirmed in the code or the docs, with the place named.
- ~ inferred, or searched for and not found, but not tested. "Not found" means
  absent from my searches of the compiler source and docs. A builtin under a
  name I did not search for would be missed, which is why those items are
  marked ~.

**Priority**

| | Meaning |
|---|---|
| **P0** | Before the first schema is written |
| **P1** | Before wave 1 goes live (the IT consulting MNC) |
| **P2** | Before wave 2 (the edge placement and the dry fruit store) |
| **P3** | Later, or optional |

## What blocks the first schema

Items 1, 2 and 14, or composite `(tenant, id)` keys in place of 1 and 2
([0014](DECISIONS/0014-databases-and-language.md), open point on UUIDv7), plus
the kernel (item 15), which is M0 itself.

**Verify first**, because I am least sure of them: items 3, 6 and 11, and the
concurrent-terminal behaviour in item 5.

## Language and host (Rust, in `eTamil_lang`)

| # | Capability | Evidence | Needed by | Priority |
|---|---|---|---|---|
| 1 | **UUID and random-number builtin**, so eTamil can generate UUIDv7 | ✔ `Cargo.toml` depends on `uuid` (v4) and `getrandom`, and `crypt.rs` uses `getrandom` for salts, nonces and keys. ~ I found no builtin that exposes either to a program | [0014](DECISIONS/0014-databases-and-language.md), ARCHITECTURE §6.4 rule 9 | P0 |
| 2 | **Millisecond clock** | ✔ The only clock builtin is `இப்போதைய_நொடி`, whole seconds (`interpreter.rs`) | UUIDv7 and sync ordering | P0 |
| 3 | **Plain SHA-256 digest** | ✔ `sha2` is a dependency. ~ I found HMAC (`கையொப்பம்`) and bcrypt (`கடவுச்சொல்_மறை`) builtins and no plain digest | Content-addressed attachments (`kOppiNYppu`) and request signing for object storage | P1 |
| 4 | **Module namespaces on import, and private helpers** | ✔ `docs/language-gaps-from-nuNNaRivu.md`: "Namespaced imports and private helpers are **not** done" | The biggest scaling risk (ARCHITECTURE §5.1). Raise it at M0 | P2 |
| 5 | **SQLite settings: WAL mode and a busy timeout**, and how several terminals behave writing to one file | ✔ I found no explicit setting in `db/sqlite.rs`. ~ rusqlite's own defaults were not checked, and concurrent writers are untested | The edge placement, [0013](DECISIONS/0013-edge-placement.md) | P2 |
| 6 | **PostgreSQL session hygiene.** The pool resets a returned connection with `ROLLBACK` only, so a session-level `SET` could leak to the next lease. Also check the driver under PgBouncer transaction pooling | ✔ `db/pool.rs` issues `ROLLBACK` on return, and I found no `DISCARD` or `RESET`. ~ The driver uses the Rust `postgres` crate's `query` and `execute`; PgBouncer compatibility is untested | Row-level security with `SET LOCAL`, [0011](DECISIONS/0011-saas-architecture.md) | P1 |
| 7 | **Request and connection timeouts, and logging and metrics on the async server** | ✔ `docs/ROADMAP.md` §4: "no connection or request timeout" | Exposing a SaaS service | P1 |
| 8 | **Named database handles**, so one process can read PostgreSQL and write SQLite | ✔ `docs/ROADMAP.md` §3: one database at a time; a second is refused | Would remove the file exchange in the export tool | P3 |
| 9 | **argv and `--quiet`** | ✔ The language-gaps doc lists them as missing and does not list them as fixed. ~ My search of `main.rs` and `interpreter.rs` found no argument builtin | Admin and migration tools take input through the environment today | P3 |
| 10 | **A language-level transaction construct** | ✔ `docs/ROADMAP.md` §3: still open | The kernel's `qaLap_parivarqqaZY` wrapper covers it | P3 |
| 11 | **Streaming or cursor reads, and line-by-line file reading** | ~ The README says rows return as an array of records, and I found no cursor API. I did not enumerate the file-read builtins | Exporting and importing large tenants | P1 |
| 12 | **Time-of-day scheduling.** `இடைவெளி N` runs on an interval only | ✔ README. `examples/katY/katY_cEvY.qmz` says a real shop would want its job nightly | Nightly jobs and payroll runs. Workaround: an interval plus a clock check | P3 |
| 13 | **A decimal aggregate in the SQLite driver** | ✔ `Cargo.toml` enables rusqlite's `functions` feature, and `db/sqlite.rs` registers scalar functions only | Fallback only. [0015](DECISIONS/0015-money-as-integer-minor-units.md) makes it unnecessary for money | P3 |
| 24 | **Static file serving in the eTamil server** | ✔ I searched `etamil_compiler/src/http/` for static file serving and found none; ARCHITECTURE §5.1 records the gap | A static Next.js export served by the one eTamil binary on an edge machine, with no reverse proxy to install ([0016](DECISIONS/0016-front-end.md)) | P2 |
| 25 | **Create a directory** | ✔ I searched the interpreter's file builtins: `கோப்பகம்_படி` reads a directory and nothing makes one. Found while writing the Tamil-twin tests, which therefore use committed fixture folders | Tests and tools that need a temporary folder | P3 |

## Libraries (`nUlakam`, written in eTamil)

| # | Capability | Evidence | Needed by | Priority |
|---|---|---|---|---|
| 14 | **Currency-aware money:** a minor-unit exponent per currency (INR 2, JPY 0, KWD 3), and conversion between a decimal amount and whole minor units that refuses a fraction instead of rounding | ✔ `paNam/kAcu.qmz` says "two decimal places, exactly" and has no decimal-to-paise conversion. ✔ I searched `kaNakkiyal/niRuvaZam.qmz` for minor-unit terms and found none. A two-function prototype of the conversion exists in `examples/katY/katY.qmz` in an uncommitted `eTamil_lang` worktree | [0015](DECISIONS/0015-money-as-integer-minor-units.md) | P0 |
| 15 | **The kernel, `atiqqaLam/`:** `paqivEtu`, `vatikattal`, `aZumaqi`, `kaNkANippu`, `qotareN`, `caripArppu`, `nikazvu`, `amYvu`, `kOppiNYppu`, `mozipeyarppu`, `qaLap_parivarqqaZY` | Planned, ARCHITECTURE §6.3 | M0 | P0 |
| 16 | **Domain families:** `atYyALam`, `pawkALi`, extensions to `kaNakkiyal`, `paNiyALar`, an extension to `vaLam/nEraqqAL`, `carakku` with batch, expiry and weight units, `vaNikam`, `muZYyam`, `uRpaqqi`, `marunqakam`, `maZiqavaLam`, `vAcal`, `miZvaNikam` | Planned, ARCHITECTURE §8.2 | By wave | P1 to P2 |
| 17 | **Tenant export and import** as eTamil programs exchanging JSON lines | Planned, [0011](DECISIONS/0011-saas-architecture.md) and [0014](DECISIONS/0014-databases-and-language.md). Speed in eTamil is unmeasured | Per-tenant restore, tenant export, moving between placements | P1 |
| 18 | **Object-storage client** with request signing | ✔ Attachments and generated documents are written to local disk today. ~ Depends on item 3 | Running on two or more nodes, [0011](DECISIONS/0011-saas-architecture.md) | P1 |
| 19 | **Mail through a provider's REST API,** routed via the outbox | ✔ I found no SMTP support in the compiler or `nUlakam` | Invoices, portal invitations, password resets | P1 |
| 20 | **Sync over the outbox,** with a versioned protocol | ✔ The outbox, retry and fingerprint modules exist in `oruwkiNYppu/`. The sync protocol does not | The edge placement, [0013](DECISIONS/0013-edge-placement.md) | P2 |
| 21 | **Updater:** download, verify, swap. Replacing a running binary may need host support | ✔ I found no updater in the compiler source, packaging, the install script or the command reference | The edge placement. Not designed or estimated | P2 |
| 22 | **Quantities as integers in a unit category's reference unit** | Open in [0015](DECISIONS/0015-money-as-integer-minor-units.md) | The inventory schema | P2 |
| 23 | **Port `check_names.py` to eTamil** | ✔ 144 lines of Python in `eTamil_lang`; nivaqi's CI runs it | [0014](DECISIONS/0014-databases-and-language.md) policy. Not decided | P3 |
| 26 | **A directory walk that can skip folders** | ✔ `kOppumuRY/கோப்பக_நட` enters every folder and has no way to prune, so a `node_modules` or `.git` is walked in full. The Tamil-twin check has its own walk for that reason | Any tool run over a repository | P3 |

## Checked, and not a gap

- **Integer money through the driver.** Integer paise are stored as `INTEGER`,
  not text, and `SUM` over such a column returned the expected total for one
  order. Tested by running `examples/katY` before and after the change to paise.
- **Rounding to whole paise.** `வட்டமிடு(x, 0)` works, and `0.1 + 0.2` converts to
  exactly 30 paise. Same test.
- **LAN serving.** `--host 0.0.0.0` is documented (`docs/reference/COMMANDS.md`).
- **Signing and verification.** HMAC-SHA256, ECDSA P-256, XChaCha20-Poly1305
  over Argon2id, and RS256 single sign-on are present. These are what sync
  signing, update verification and SSO need.
- **Fixed since the language-gaps document was written:** record key lookup,
  host JSON, in-place string accumulation, logarithms and powers, and sorting.
- **Outbound HTTP and Redis** are present.

## How to use this document

- Each item becomes a request or a branch in `eTamil_lang`. A host item is a
  Rust change; a library item follows the nine deliverables in
  [ARCHITECTURE §9](ARCHITECTURE.md) and the naming rules in §8.1.
- Re-check the ~ items in the M0 spike and update their evidence marks, so the
  list does not drift from the code.
- Add an item when nivaqi finds a gap, and remove it when it is built, with the
  commit that built it.
- **No item is started.** Item 14's prototype is the only code that exists.
