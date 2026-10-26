# 0018 — How nivaqi consumes eTamil-ciRal

**Status:** accepted, 2026-10-26, recorded on the owner's instruction: a workspace or
a git dependency, whichever is better for performance and maintenance, and no copy.
Settles the open point carried in [0016](0016-front-end.md) and
[0017](0017-where-code-lives.md). **Not yet tried:** no install has been run, so what
follows rests on documentation, and the things to confirm are listed below.

## Context

The framework is the package in `ui/` of `Maruff/eTamil-ciRal`. It ships TypeScript
source (`main` is `src/index.ts`), it is `private: true`, it is not on npm, and
eTamil-ciRal has nothing pushed for it yet. nivaqi's application, `mukappu/`, does
not exist yet.

## Decision

1. **nivaqi depends on eTamil-ciRal as a git dependency pinned to a commit,
   installed with pnpm.** The specifier has the form
   `github:Maruff/eTamil-ciRal#<commit>&path:/ui`; the exact string is confirmed at
   the first install.
2. **For the inner loop, link the sibling checkout** (`pnpm link` to
   `../eTamil-ciRal/ui`). That is local, is never committed, and never reaches CI or a
   release, which always use the pinned dependency.
3. **No copy, no git submodule, and no workspace spanning the two repositories.**
4. **A bump is one reviewed change:** the commit in `mukappu/package.json` and the
   lockfile together.

## Why a git dependency and not a workspace

- A workspace needs its packages under one root. Two repositories means a submodule
  or a sibling path. Neither pins a version, and a sibling path does not exist in CI.
- A pinned commit is recorded in the lockfile, so a build is reproducible. That is
  what ARCHITECTURE section 3.3 already asks of every library: pin it, never track
  `main`.
- **Performance.** An install fetches one small tarball per pin and pnpm caches it in
  its content-addressable store. The slow part of a cross-repository loop, commit,
  push, bump and reinstall, is avoided by linking, so editing a ciRal component shows
  in nivaqi at once.

## Why pnpm and not npm

pnpm documents installing a subdirectory of a git repository with a `path:`
parameter, and pinning a git dependency to a commit
([package sources](https://pnpm.io/package-sources)). I found no evidence that npm
can install a subfolder of a git repository. The alternative is to move the package
to the root of eTamil-ciRal, which would undo its being a home for several packages.

## Why not a registry

Publishing to npm or GitHub Packages is outward-facing and was not asked for. Revisit
it if more than one repository ends up consuming the framework.

## To confirm at the first install

These come from reading documentation, not from trying them:

- pnpm accepts `#<commit>&path:/ui` together.
- Next.js needs `transpilePackages: ['ciral-ui']`, because the package ships
  TypeScript source and not a build.
- The application's `tsconfig` needs `allowImportingTsExtensions`, because ciRal's
  files import each other with the `.ts` extension.
- pnpm's settings for git dependencies, which differ between its recent versions,
  allow a direct git dependency. They concern a git dependency of a dependency.

## Before anything can pin it

eTamil-ciRal needs a pushed commit, and a tag is better. Nothing is committed there
yet. pnpm also has to be installed; it is not on this machine.

## Not taken

- **A copy of the package in nivaqi**, which the owner ruled out.
- **A git submodule with a workspace over it.** It pins a commit and gives an
  edit-in-place loop, but every clone then needs `--recursive`, and CI has to remember
  to initialise it. Linking gives the same loop without the submodule.
- **npm**, for the subfolder reason above.

## Open

- The first-install checks above.
- Pinning the pnpm version in `mukappu/package.json` (`packageManager`).
- Whether to publish to a registry later.
