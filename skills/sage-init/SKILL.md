---
name: sage-init
description: >
  Initialize a new LLM-powered personal wiki using the Karpathy/sage-wiki pattern.
  Use this skill when a user wants to create, bootstrap, or set up a new knowledge
  base, personal wiki, second brain, zettelkasten, or research wiki.
  Trigger on phrases like: "create a wiki", "set up my wiki", "initialize a knowledge
  base", "new wiki", "start a wiki", "wiki init", "sage init", "bootstrap wiki".
  If the user mentions wanting a persistent knowledge base that compounds over time,
  proactively suggest this skill even if they don't use the word "wiki".
---

# sage-init — Bootstrap a New Wiki

Sets up the wiki directory structure and writes the convention file that all other
sage-wiki skills depend on.

## Pre-flight

Before doing anything, check for an existing `SCHEMA.md` by searching upward from
the current directory. If found, ask the user whether they want to reinitialize
(this will overwrite `SCHEMA.md`) or just inspect the existing setup.

## Questions to ask (in order, stop if user says "defaults are fine")

1. **Location** — Where should the wiki live? (default: `./wiki`)
2. **Domain** — What subject area is this wiki for? (e.g. "ML research", "company notes") — used to seed the SCHEMA description
3. **Source types** — What kinds of sources will you ingest? (papers, web articles, meeting notes, books, code, other)

## Directory structure to create

```
<wiki-root>/
├── SCHEMA.md          ← conventions file (see below)
├── raw/               ← immutable source documents — never modified
├── wiki/
│   ├── pages/         ← flat directory, all pages live here as slugs
│   ├── index.md       ← catalog of all pages, grouped by tag
│   ├── overview.md    ← evolving high-level synthesis of the domain
│   └── log.md         ← append-only operation log
└── assets/            ← images, PDFs, attachments referenced in pages
```

`wiki/pages/` must stay **flat** — no subdirectories. Pages are named with
lowercase hyphen-separated slugs (e.g. `attention-mechanism.md`).

## Write SCHEMA.md

Customize the template below using the user's answers. This file is the single
source of truth for all other skills; keep it accurate.

```markdown
# Wiki Schema — <domain>

## Identity
- **Domain:** <domain from question 2>
- **Owner:** <ask or leave blank>
- **Created:** <today's date>
- **Source types:** <from question 3>

## Directory layout
- `raw/` — immutable source documents; never edit these
- `wiki/pages/` — flat slug-named pages (e.g. `attention-mechanism.md`)
- `wiki/index.md` — master catalog, updated after every ingest/update
- `wiki/overview.md` — evolving synthesis; rewritten during lint passes
- `wiki/log.md` — append-only log of all operations
- `assets/` — referenced images and attachments

## Page frontmatter (required)
```yaml
---
title: "Human-Readable Title"
slug: page-slug
tags: [tag1, tag2]
sources: [source-slug-1]   # slugs of raw/ sources that support this page
confidence: high            # high | medium | low
created: YYYY-MM-DD
updated: YYYY-MM-DD
supersedes: []              # slugs this page replaces
superseded-by: ""           # slug of newer page if this one is outdated
---
```

## Confidence levels
- **high** — multiple independent sources agree; well-established claim
- **medium** — single source, or sources with minor disagreement
- **low** — speculative, single potentially-biased source, or from prior general knowledge

## Link syntax
Use `[[slug]]` for internal links. Every claim should link to its supporting page.
When creating a new page, scan at least 10 existing pages and add backlinks to it.

## Naming conventions
- Page slugs: `lowercase-hyphen-separated` (filename = slug + `.md`)
- Source slugs: same convention, e.g. `attention-is-all-you-need`
- Tags: lowercase, singular nouns preferred

## Log entry format
```
[YYYY-MM-DD HH:MM] <operation> | <slugs affected> | <brief note>
```
```

## Seed files

Write these minimal starter files:

**wiki/index.md**
```markdown
# Wiki Index

_Last updated: <today>_

## Pages by tag

_No pages yet. Run `sage-ingest` to add your first source._
```

**wiki/overview.md**
```markdown
# Domain Overview — <domain>

_This file is maintained by `sage-lint`. It will be written after the first lint pass._
```

**wiki/log.md**
```markdown
# Operation Log

[<today>] init | — | Wiki initialized
```

## Finishing up

Tell the user:
- Where the wiki was created
- To drop source files into `raw/` and run `sage-ingest` to start building it
- That `SCHEMA.md` is the config file — editing it changes behavior for all operations
