# 0016 — The front end: Next.js, with the framework in eTamil-ciRal

**Status:** accepted, 2026-10-26, recorded on the owner's instruction. Settles the
Next.js or Vite question, and where the form framework lives.

## Context

While nivaqi is built, the owner asked for React and Next.js code, with HTML and
CSS, to become the base of the eTamil Studio framework. Four questions came back
with it, and the owner answered them: use `Maruff/eTamil-ciRal` for these
frameworks and libraries; Studio's scope has been redefined in the work breakdown;
Next.js is fine; and the working name is fine.

## Decision

1. **The front end is React with Next.js and TypeScript.** It was React 19 with
   Vite. **Recommended, not decided:** use Next.js with a static export, so no Node
   server runs at run time. That matters on the small machines the edge placement
   runs on (0013).
2. **The form and list framework lives in the public repository
   `Maruff/eTamil-ciRal`, in `ui/`**, not in nivaqi. nivaqi's own application,
   `mukappu/`, is built with it and holds nivaqi's screens, not framework code.
3. **Studio's scope is group 2 of the work breakdown**, "eTamil Studio (Visual IDE)":
   a low-code platform whose first module is the form builder (2.1), with text and
   data controls (2.1.1, 2.1.2). The owner's decisions of 2026-10-11 on the order
   still stand (`docs/studio/FIRST-SLICE.md`, on the `docs/studio-first-slice`
   branch of `eTamil_lang`): the IDE first, low-code built on top as a generator,
   and a form builder whose first slice generates an eTamil program and a plain HTML
   form that run in the browser. A generated React form is "the destination, after
   the slice works". The framework is the React half of that destination. It does
   not replace the first slice.
4. **A rule is eTamil.** A calculation or a check is eTamil: in the browser it runs
   on the compiler's WebAssembly build, and on a server it is the same eTamil again,
   whose answer wins. The framework carries such a rule as eTamil source and does
   not grow a rule language in JavaScript. Carrying them is not built yet.
5. **A field name is a plain ASCII name** in the project's romanization, never Tamil
   letters, because it is seen by systems with no Tamil fonts. This is decision 3 of
   `docs/studio/FORM-STORAGE.md`, and the framework's `checkForm` enforces it.

The React, Next.js and TypeScript front end is an exception to the eTamil-by-default
policy in 0014, and its row there says so.

## What was done to the repository

`eTamil-ciRal` held one README, a 2024 plan for a Rust and eTamil framework. The
working tree was emptied and `ui/` and a new README put in its place, on `main`.
**History was not rewritten and nothing was pushed**, so the old README is still
readable with `git show 953ed4f:README.md`.

## Consequences

- ARCHITECTURE section 6.6 and the README change from Vite to Next.js, and **drop
  Tailwind**: the framework's CSS is plain, with the design tokens as variables.
- A static Next.js export still has to be served. On the edge that means a reverse
  proxy, or **static file serving in the eTamil server**, which CAPABILITIES item 24
  records as a gap.
- `ui/` is `private: true` in `package.json` and is not on npm, so how nivaqi
  consumes it is open.

## Not taken

- **Vite** for the nivaqi application.
- **Keeping the framework in nivaqi.** Studio needs it too, and nivaqi's repository
  is for nivaqi.
- **Tailwind**, which the framework's own stylesheet makes unnecessary.

## Open

- **Whether the kernel and other generic libraries also belong in eTamil-ciRal.**
  Settled in [0017](0017-where-code-lives.md): JavaScript and TypeScript libraries go to
  eTamil-ciRal and eTamil libraries to `nUlakam`, so the kernel stays in `nUlakam`.
- **The licence file and the copyright line for eTamil-ciRal.** The headers say
  "Isha Maruff and Esan Maruff", as nivaqi's do; `eTamil_lang` asserts copyright in
  Mohammed Maruff. There is no `LICENSE` file yet.
- **Whether to keep eTamil-ciRal's history** or start clean. Starting clean means an
  orphan commit and a force-push, which is the owner's to do.
- **Static export or server rendering** for Next.js.
- **How nivaqi consumes the package.** Settled in [0018](0018-consuming-ciral.md): a git dependency
  pinned to a commit, with no copy.
