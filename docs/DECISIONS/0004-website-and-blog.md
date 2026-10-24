# 0004 — No CMS in eTamil

**Status:** accepted, 2026-10-24.

## Context

The third module set listed Website, Blog, eCommerce and three portals. The
last four are ERP. The first two are a content management system.

## Decision

Three parts:

- **Marketing site and blog** — a static site generator (Astro or Hugo),
  content in Markdown in the repository. Deployed beside nivaqi, linking into
  it.
- **eCommerce storefront** — React against the nivaqi API. This is the real
  work, and `examples/katY` has started it.
- **Content module** — only if non-developers must edit pages, and then it is
  pages, blocks, media and publish state. About a week. Not a website builder.

## Rationale

The decisive argument is structural, not effort. **A blog has no business
rule**, so it can contribute nothing to `nUlakam`. Requirement 3 says every
module either adds a library family or extends one; a blog engine cannot. It
would be the one module in nivaqi that is pure application code — exactly what
the minimal-code constraint exists to prevent.

## Consequences

- Saves roughly 10 weeks against building a CMS.
- The website is a separate deployment with its own lifecycle, which is also
  how it should be: marketing copy should not need an ERP release.
