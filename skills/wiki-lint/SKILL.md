---
name: wiki-lint
description: >
  Run a health check on the Agent's wiki. Finds broken links, orphaned pages,
  contradictions, stale content, unprocessed session backlog, and knowledge gaps.
  Use after every 5–10 ingests, or when the wiki feels disorganized.
  Trigger: "wiki lint", "wiki health check", "check wiki", "audit wiki",
  "/wiki lint", "how healthy is my wiki", "clean up wiki".
  Also suggest proactively after 10+ ingest operations in wiki/log.md without a
  lint entry — maintenance debt compounds.
user-invokable: true
---

# wiki-lint — Wiki Health Audit

Audits the wiki for structural and semantic issues, then writes a lint report page.
Runs without user confirmation — read-heavy with one new file written at the end.

## Step 0 — Load context

Read `SCHEMA.md` and `wiki/log.md` fully.

From `wiki/log.md`, compute:
- Total ingest operations since last lint (or since init if never linted)
- Total sessions processed vs. unprocessed in `raw/sessions/`
- Pages created/updated since last lint

## Step 1 — Inventory all wiki content

Read every file in `wiki/pages/`. Build an in-memory map of:
- All page slugs → titles, page_type, tags, sources, confidence, session_count, updated date
- All `[[slug]]` links used per page (outgoing links)
- All tags used across all pages
- Invert: for each slug, which pages link TO it (incoming links)

Also read `wiki/index.md` and `wiki/overview.md`.

## Step 2 — Check unprocessed session backlog (Sage-specific)

List `raw/sessions/*.jsonl`. Cross-reference with ingest entries in `wiki/log.md`.

Flag:
- **Backlog > 10 sessions** → 🔴 Error: wiki is falling behind, run wiki-ingest immediately
- **Backlog 5–10 sessions** → 🟡 Warning: schedule ingest soon
- **Backlog < 5 sessions** → 🔵 Info: normal accumulation

## Step 3 — Run structural checks

### 🔴 Errors (fix before trusting the wiki)

- **Broken links** — `[[slug]]` references with no corresponding `.md` in `wiki/pages/`
- **Missing required frontmatter** — pages lacking `title`, `slug`, `page_type`,
  `confidence`, `session_count`, `created`, or `updated`
- **Slug/filename mismatch** — frontmatter `slug` doesn't match filename
- **Invalid page_type** — `page_type` is not one of:
  pitfall / pattern / api-ref / decision / concept / synthesis
- **Superseded chain broken** — page has `superseded-by: slug` but that slug doesn't exist

### 🟡 Warnings (worth addressing soon)

- **Orphaned pages** — pages with zero incoming `[[slug]]` links from any other page.
  Lone knowledge is forgotten knowledge.
- **Contradictions** — two pages on the same topic with opposing claims.
  Look for: same primary tag + opposing language, or one `supersedes` the other
  without matching `superseded-by` on the other side.
- **Stale medium/low-confidence pages** — pages with `confidence: medium` or `low`
  that haven't been updated in 60+ days and `session_count < 3`. These should either
  be promoted (more sessions corroborate) or marked as `superseded-by: ""` / deleted.
- **Low-confidence clusters** — 3+ pages all `confidence: low` on overlapping tags.
  This suggests a knowledge gap worth a targeted ingest run.
- **Index gaps** — pages in `wiki/pages/` not listed in `wiki/index.md`.

### 🔵 Info (nice to have)

- **Missing backlinks** — page content mentions a concept that has a slug, but
  doesn't use `[[slug]]` syntax. List page + unlinked term.
- **Open questions accumulation** — concepts appearing in 3+ pages' `## Open questions`
  sections but with no wiki page. These are the wiki's growing edge — worth a future ingest.
- **High session_count without high confidence** — pages with `session_count ≥ 5` still at
  `confidence: medium`. These should be promoted to `high`.
- **Tag sprawl** — tags used only once (often inconsistent tagging).
- **Craft references** — if wiki pages mention workflows that could be a Craft
  (e.g., "to do X, follow these 5 steps"), flag for possible `craft/` creation.

## Step 4 — Write the lint report

Create `wiki/pages/lint-YYYY-MM-DD.md` automatically:

```markdown
---
title: "Lint Report — YYYY-MM-DD"
slug: lint-YYYY-MM-DD
page_type: synthesis
tags: [maintenance, lint]
sources: []
session_count: 0
confidence: high
created: YYYY-MM-DD
updated: YYYY-MM-DD
supersedes: [lint-PREV-DATE]
superseded-by: ""
---

# Lint Report — YYYY-MM-DD

## Summary

| Metric | Value |
|--------|-------|
| Total pages | N |
| Unprocessed sessions | N |
| Sessions since last lint | N |
| Last lint | PREV-DATE or "never" |

| Severity | Count |
|----------|-------|
| 🔴 Errors | N |
| 🟡 Warnings | N |
| 🔵 Info | N |

## 🔴 Errors

### Broken links
- `[[missing-slug]]` referenced in: [[page-a]], [[page-b]]
  → Create the page or remove the link

### [other errors...]

## 🟡 Warnings

### Orphaned pages
- [[orphan-slug]] — no incoming links, confidence: medium, session_count: 1
  → Add link from [[suggested-parent]] if relevant, or mark superseded

### Contradictions
- [[page-a]] vs [[page-b]] — both cover `feishu-auth` but make opposing claims
  → Ingest a clarifying session, update one, set superseded-by on the older

### [other warnings...]

## 🔵 Info

### Open questions worth ingesting
- "feishu recurring event behavior" — mentioned in: [[page-a]], [[page-b]], [[page-c]]
  → This is a knowledge gap. Next time a session touches this, capture it.

### Potential Craft candidates
- [[batch-event-update]] describes a 5-step workflow — consider creating craft/batch-event-update/

### [other info...]

## Recommended next actions

1. [Most impactful fix — usually: ingest backlog or broken links]
2. [Second action]
3. [Third action]
```

Mark the previous lint report with `superseded-by: lint-YYYY-MM-DD`.

## Step 5 — Update index and overview

- Add the lint report to `wiki/index.md` under the index header (or update existing lint entry)
- Re-read `wiki/overview.md`. If significant new knowledge has accumulated since it was
  last written (5+ new pages), rewrite it to reflect current understanding and ask
  the user whether to apply the update.

## Step 6 — Log

```
[YYYY-MM-DD HH:MM] lint | errors: N, warnings: N, info: N | lint-YYYY-MM-DD
```

## Report to user

- Total page count and overall health (🟢 green / 🟡 yellow / 🔴 red)
- Unprocessed session backlog count
- Top 3 most impactful fixes
- Pages not updated in 90+ days with confidence < high
