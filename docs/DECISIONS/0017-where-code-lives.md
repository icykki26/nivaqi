# 0017 — Where code lives, and the language of the documentation

**Status:** accepted, 2026-10-26, recorded on the owner's instruction. Settles the
open point in [0016](0016-front-end.md) about the kernel. The scope of the Tamil
documentation and the automated twin check were settled later the same day.

## Context

Three repositories now take part in building nivaqi, and the question was which
code goes where. `Maruff/eTamil-ciRal`'s earlier plan overlapped what the nivaqi
kernel does, so the kernel's home was open. The owner also asked that nivaqi's
code be very small and its documentation clear, in Tamil and in English.

## Decision

**1. Where code lives**

| Kind of code | Repository |
|---|---|
| eTamil libraries: the business rules, the kernel, the domain families | `nUlakam`, in `Maruff/eTamil_lang` |
| JavaScript and TypeScript libraries: the form and list framework | `Maruff/eTamil-ciRal` |
| The application built from those libraries | `icykki26/nivaqi` |

So **the kernel, `atiqqaLam`, stays in `nUlakam`**, and ciRal holds only
JavaScript and TypeScript libraries.

**2. nivaqi holds very little code.** Routes and workers, the mapping to the
database, configuration, migrations, seed data, thin eTamil entry points, and
screens composed from the ciRal framework. It holds **no business rule and no
helper that two modules share**. Anything like that moves at once to `nUlakam`
(if it is eTamil) or to ciRal (if it is JavaScript or TypeScript).

**3. Documentation is in Tamil and in English.**

- Every document a person reads to use or extend nivaqi, **inside a code folder**,
  has an English file and a Tamil twin: `X.md` and `X.ta.md`, each linking to the
  other at the top. That includes the root README and every module's documents.
- **`docs/` is English only, by decision**: the architecture, the decision records
  and the capability list. The check skips it.
- Code comments follow the eTamil convention: English inside `__…__`, Tamil in
  plain text.
- The interface is in both languages from the first screen.
- **An automated check enforces the twins.** `karuvi/qamiz_iNY_caripArppu.qmz` is
  an eTamil program. It finds every document outside `docs/`, `.git`, `.github`,
  `node_modules` and the test fixtures. It requires each `X.md` to have an
  `X.ta.md` and the other way round, and each to link to the other within its first
  twelve lines. It requires the same number of headings, table rows and code fences
  (a `#` inside a code fence is code, not a heading), and at least ten words that
  begin with a Tamil letter, so a copy of the English does not pass. It exits
  non-zero on any problem. `.github/workflows/docs-twin.yml` runs it, and its own
  tests, in CI on a pinned eTamil release. It cannot judge whether the Tamil is
  any good.

## What it changes

- **ARCHITECTURE section 3.3** describes three repositories, not two.
- **Section 6.2's layout** loses `atiqqaLam/` and the per-module `uqavi/` examples,
  which are library material, and `karuvi/` is described as thin entry points.
- **Section 9** says where each of a module's nine deliverables lives: the library
  family, its examples and its tests in `nUlakam`; the rest here, with the React
  screens built from ciRal.
- **Section 11** requires the Tamil twin.
- The root README now has a Tamil twin, `README.ta.md`.
- `karuvi/` also holds one tool of its own, the twin check with its tests and
  fixtures, because nothing else uses it. It moves to `nUlakam` if another
  repository wants it.

## Consequences

- A feature can touch up to three repositories. Each library change lands in its
  own repository first, with its tests, and is released; nivaqi pins the version of
  each and bumps it deliberately.
- Library tests and `uqavi/` examples are library material, so they live with the
  library. nivaqi keeps the tests of its own routes against seeded data, the
  cross-module scripts and the interface tests.
- The "code ratio" of section 3.1 gets simpler to hold: if nivaqi contains a rule or
  a shared helper, that is the defect.

## Not taken

- **The kernel in eTamil-ciRal.** ciRal is for JavaScript and TypeScript.
- **Libraries inside nivaqi.** That is how an application quietly grows a private
  copy of a standard library.
- **English-only documentation inside a code folder.** `docs/` is English only, by
  decision; everything else is written twice.

## Open

- **Review.** Every Tamil text here is a first draft, including `README.ta.md` and
  the Tamil in the check's messages. The owner will read it and change what needs
  changing, later.
- **The check has not run in CI.** `.github/workflows/docs-twin.yml` is written and
  not yet run, because the repository has to be on GitHub first. The two `etamil`
  commands in it were run locally and pass; the install steps (download, checksum,
  unpack) were not run, and the YAML has not been parsed by anything.
- How nivaqi consumes eTamil-ciRal is settled in [0018](0018-consuming-ciral.md).
