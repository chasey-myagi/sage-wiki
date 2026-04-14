---
name: sage-ingest
description: >
  Add documents, articles, papers, URLs, or notes to an existing LLM wiki using the
  sage-wiki pattern. Use this skill when the user wants to process source material
  into their wiki, add an article or paper to their knowledge base, or expand their
  wiki with new content.
  Trigger on: "ingest", "add to my wiki", "process this document", "add this article",
  "read and save to wiki", "sage ingest", "put this in my wiki", "add this paper".
  Also trigger when the user pastes a long document or shares a file path while an
  active wiki (SCHEMA.md) exists nearby — they probably want to ingest it.
---

# sage-ingest — Add Sources to the Wiki

Takes any source (file, URL, pasted text) and synthesizes it into interconnected
wiki pages. The goal is not to summarize the source — it is to extract durable
knowledge and weave it into the existing wiki.

## Step 0 — Find the wiki

Search upward from the current directory for `SCHEMA.md`. If not found, check
`~/wikis/` as a fallback. If still not found, tell the user to run `sage-init`
first and stop.

Read `SCHEMA.md` fully before proceeding — it governs naming, frontmatter, and
confidence levels.

## Step 1 — Accept the source

The user will provide one of:
- A file path (in `raw/` or elsewhere)
- A URL
- Pasted text

If the source is a file not yet in `raw/`, ask: "Should I copy this to `raw/<slug>.md`
for archival?" — but don't block on it.

Read the source **completely** before writing anything.

## Step 2 — Surface key insights (WAIT for user)

Before touching any files, tell the user:

> I've read `<source title>`. Here are 3–5 key insights I'd pull into the wiki:
>
> 1. ...
> 2. ...
> 3. ...
>
> Any you'd like to skip, reframe, or add?

Wait for their response. Adjust based on what they say. This step ensures the wiki
reflects what the user actually cares about, not just what the LLM finds salient.

## Step 3 — Identify pages to create or update

Read `wiki/index.md` to see what already exists. For each insight:

- **Existing page matches** → update it; note the new source in frontmatter
- **No match** → create a new page

For new pages, pick a slug that is specific enough to be unambiguous but general
enough to stay useful as the wiki grows (prefer `transformer-attention` over
`bert-layer-12-self-attention`).

## Step 4 — Write or update pages

Use this format for every page:

```markdown
---
title: "Concept Title"
slug: concept-slug
tags: [tag1, tag2]
sources: [source-slug]
confidence: high
created: YYYY-MM-DD
updated: YYYY-MM-DD
supersedes: []
superseded-by: ""
---

# Concept Title

One sentence that defines or positions this concept.

## What it is

2–4 paragraphs of synthesis. Write for future-you in 6 months who has forgotten
the source. Prioritize the "why it matters" over the "what it is" — the original
source has the details; the wiki should have the understanding.

## Key claims

- Claim one (confidence: high) — [[supporting-page]] if related
- Claim two (confidence: medium) — source attribution if needed
- Claim three

## Connections

Links to related pages with a one-line note on *how* they relate:

- [[related-slug]] — shares the same underlying mechanism
- [[another-slug]] — generalizes this concept

## Open questions

Things this source raises but doesn't resolve. These are seeds for future ingests:

- Question one
- Question two

## Sources

- `raw/source-slug.md` — key quote or insight from this source
```

**Handling contradictions:** If a new insight contradicts an existing page, do not
silently overwrite. Instead:
1. Note the contradiction in the existing page under a `## Contradictions` section
2. Set `confidence: medium` on the affected claims
3. Note it in the ingest summary at the end

**Handling supersession:** If the new source clearly supersedes an old page
(e.g., a newer paper invalidates an older finding), set `superseded-by: new-slug`
on the old page and `supersedes: [old-slug]` on the new one.

## Step 5 — Backlink audit

This step is the most commonly skipped and the most important for keeping the
wiki connected.

After writing all new/updated pages, read the 10 most recently updated existing
pages (use `wiki/log.md` to find them). For each:
- If it mentions a concept that now has a page, add `[[slug]]` to its Connections section

Also search `wiki/pages/` for pages whose tags overlap with the new material and
add backlinks there too.

## Step 6 — Update index and log

**wiki/index.md:** Add new pages under their primary tag group. Format:
```
- [[slug]] — one-line description (created: YYYY-MM-DD)
```

**wiki/log.md:** Append:
```
[YYYY-MM-DD HH:MM] ingest | <source-slug> → <new-slugs>, updated: <updated-slugs> | <contradiction note if any>
```

## Step 7 — Report to user

Tell the user:
- Pages created: list of slugs
- Pages updated: list of slugs
- Contradictions found: if any, with brief description
- Notable connections discovered: surprising links to existing knowledge
- Open questions worth addressing with a future ingest
