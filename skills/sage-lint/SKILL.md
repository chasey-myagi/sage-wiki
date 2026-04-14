---
name: sage-lint
description: >
  Run a health check on an LLM wiki to find broken links, orphaned pages,
  contradictions, stale content, and knowledge gaps. Use this skill for wiki
  maintenance, quality audits, or when the wiki feels disorganized or unreliable.
  Trigger on: "lint my wiki", "check my wiki", "audit wiki", "wiki health check",
  "fix wiki links", "find contradictions", "sage lint", "wiki cleanup",
  "clean up wiki", "check for orphaned pages".
  Proactively suggest running sage-lint after every 5–10 ingests even if the
  user hasn't asked — maintenance debt compounds faster than content debt.
---

# sage-lint — Wiki Health Audit

Audits the wiki for structural and semantic issues, then writes a lint report
page with actionable remediation steps. Runs without confirmation — this is a
read-then-write operation with no destructive edits.

## Step 0 — Find the wiki

Search upward from the current directory for `SCHEMA.md`. Fallback: `~/wikis/`.
If not found, tell the user to run `sage-init` first and stop.

## Step 1 — Inventory

Read all pages in `wiki/pages/`, plus `wiki/index.md` and `wiki/overview.md`.
Build an in-memory map of:
- All page slugs and their titles
- All `[[slug]]` links used in each page
- All tags used
- All `updated` dates

## Step 2 — Run checks by severity

### 🔴 Errors (fix before trusting the wiki)

- **Broken links** — `[[slug]]` references with no corresponding `.md` file in `wiki/pages/`
- **Missing required frontmatter** — pages lacking `title`, `slug`, `confidence`, `created`, or `updated`
- **Slug/filename mismatch** — frontmatter `slug` doesn't match the filename

### 🟡 Warnings (worth addressing soon)

- **Orphaned pages** — pages with no incoming `[[slug]]` links from any other page
- **Contradictions** — two pages making incompatible claims about the same concept.
  Look for: pages with the same tags and opposing language ("X causes Y" vs "X does not cause Y"),
  or pages where one `supersedes` the other but neither links to the other.
- **Stale content** — pages with `updated` > 90 days ago that use present-tense language
  ("X is currently...", "the latest approach is..."). Flag these for review, not automatic deletion.
- **Low-confidence clusters** — groups of 3+ pages all marked `confidence: low` on overlapping
  topics. Suggests a knowledge gap worth a targeted ingest.

### 🔵 Info (nice to have)

- **Missing backlinks** — pages that mention a concept for which a page exists, but
  don't use `[[slug]]` syntax. List the page and the unlinked term.
- **Implied concepts** — terms that appear in 3+ pages' Open Questions but have no page yet.
  These are the wiki's growing edge.
- **Index gaps** — pages that exist in `wiki/pages/` but aren't listed in `wiki/index.md`
- **Tag sprawl** — tags used only once (often a sign of inconsistent tagging)

## Step 3 — Write the lint report

Create `wiki/pages/lint-<YYYY-MM-DD>.md` automatically (no confirmation needed):

```markdown
---
title: "Lint Report — YYYY-MM-DD"
slug: lint-YYYY-MM-DD
tags: [maintenance, lint]
sources: []
confidence: high
created: YYYY-MM-DD
updated: YYYY-MM-DD
supersedes: []
superseded-by: ""
---

# Lint Report — YYYY-MM-DD

## Summary

| Severity | Count |
|----------|-------|
| 🔴 Errors | N |
| 🟡 Warnings | N |
| 🔵 Info | N |

## 🔴 Errors

### Broken links
- `[[missing-slug]]` referenced in: [[page-a]], [[page-b]]
  → Create the page or remove the link

...

## 🟡 Warnings

### Orphaned pages
- [[orphan-slug]] — no incoming links
  → Add a link from [[suggested-parent]] or delete if no longer relevant

### Contradictions
- [[page-a]] claims "X causes Y" but [[page-b]] claims "X does not cause Y"
  → Resolve by ingesting a clarifying source, then update one page and set
    `superseded-by` on the older claim

...

## 🔵 Info

### Implied concepts (worth creating pages for)
- "concept-name" — mentioned in open questions of: [[page-a]], [[page-b]], [[page-c]]

...

## Recommended next actions

1. Fix broken links first (errors block reliable querying)
2. Resolve [[page-a]] vs [[page-b]] contradiction by ingesting <suggested source>
3. Create pages for top 2 implied concepts
```

## Step 4 — Update index and overview

- Add the lint report to `wiki/index.md` under a `maintenance` section
- Re-read `wiki/overview.md`. If the wiki has grown significantly since it was last
  written, propose an updated version and ask the user whether to apply it.

## Step 5 — Log

Append to `wiki/log.md`:
```
[YYYY-MM-DD HH:MM] lint | errors: N, warnings: N, info: N | lint-YYYY-MM-DD
```

## After the audit

Tell the user:
- Total page count
- Overall health (green / yellow / red based on error/warning counts)
- The top 3 most impactful fixes
- How many pages were last updated more than 90 days ago
