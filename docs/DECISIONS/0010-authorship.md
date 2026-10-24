# 0010 — Authorship

**Status:** accepted, 2026-10-24.

## Decision

All nivaqi work is documented as authored by **Isha Maruff and Esan Maruff**,
both inside files and outside them.

**Inside every source file**, after the SPDX line:

    // SPDX-License-Identifier: AGPL-3.0-or-later
    // Copyright (C) 2026 Isha Maruff and Esan Maruff

**Outside:** `AUTHORS`, `README.md`, documentation, release notes and package
metadata all name both.

`AUTHORS` takes the same care over names that `eTamil_lang/AUTHORS` does — one
file, so that "who wrote this" has a single answer.

## Settled

`AUTHORS` carries the contact addresses: Isha Maruff &lt;isha@etamil.in&gt; and
Esan Maruff &lt;esan@etamil.in&gt;.

**Commits are authored `Isha Maruff &lt;icykki26@gmail.com&gt;`**, set
repo-locally. The commit address differs from the contact address because
GitHub links a commit to an account by matching the author's email, and
`icykki26@gmail.com` is the address on the account the nivaqi repositories live
under — so attribution works with nothing to verify. `AUTHORS` explains the
pair. Repositories under the Maruff account are unaffected: the global identity
is unchanged.

## Note

Git commit authorship is a separate matter and is the owner's to configure.
Nothing in this repository changes it, and nothing here should be read as
doing so.
