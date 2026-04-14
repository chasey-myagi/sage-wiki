---
name: sage-query
description: >
  Query an LLM wiki to answer questions strictly from stored knowledge — not from
  general training data. Use this skill when the user asks a question they want
  answered from their wiki, wants to find specific information in their knowledge
  base, or explicitly asks "what does my wiki say about X".
  Trigger on: "query my wiki", "search my wiki", "what does my wiki say about",
  "ask my wiki", "sage query", "find in wiki", "wiki search", "what did I save
  about", "what do I know about" (when an active wiki exists).
  When the user asks a domain question and a SCHEMA.md is nearby, prefer answering
  from the wiki rather than from general knowledge.
---

# sage-query — Ask Your Wiki

Answers questions by reading wiki pages, not by drawing on training data. The wiki
is the source of truth. This keeps knowledge grounded in what the user has
deliberately accumulated, and surfaces gaps that are worth filling.

## Step 0 — Find the wiki

Search upward from the current directory for `SCHEMA.md`. Fallback: `~/wikis/`.
If not found, tell the user and stop.

Read `SCHEMA.md` to understand the domain and page conventions.

## Step 1 — Understand the question

If the question is ambiguous, clarify once before reading pages. Don't start
reading the whole wiki hoping to stumble on the answer.

## Step 2 — Read the index

Read `wiki/index.md` fully. Identify the 3–7 pages most likely to contain the
answer. If the index isn't detailed enough, also check `wiki/overview.md` for
high-level orientation.

## Step 3 — Read relevant pages

Read the identified pages in full. Follow one level of `[[slug]]` links if a
linked page seems directly relevant — but don't go deeper than one hop unless
the question specifically requires it.

## Step 4 — Synthesize the answer

Write an answer that:
- **Is grounded only in wiki content.** Do not answer from general knowledge,
  even if you think you know the answer. The value of the wiki is that it
  reflects *this user's accumulated understanding*, not generic LLM knowledge.
- **Cites inline.** Every claim should be followed by `([[slug]])`.
- **Notes disagreements.** If two pages contradict each other on a relevant
  point, surface it explicitly: "[[page-a]] says X, but [[page-b]] says Y —
  this may be worth resolving."
- **Is honest about gaps.** If the wiki doesn't have a good answer, say so.
  Don't pad with general knowledge.

## Step 5 — Flag gaps and suggest follow-ups

After answering, tell the user:
- Which pages were most useful
- What the wiki is missing that would have given a better answer
- Specific sources worth ingesting to fill those gaps (if you can infer them)

## Step 6 — Offer to save the insight

If the answer involved interesting synthesis that isn't captured in any single page:

> "This analysis connects [[slug-a]], [[slug-b]], and [[slug-c]] in a way that
> isn't explicit anywhere. Worth saving as `wiki/pages/<slug>.md`?"

If the user says yes:
- Write the page with `tags: [query, synthesis]` and `sources: [cited slugs]`
- Update `wiki/index.md`
- Log to `wiki/log.md`: `[date] query-saved | <slug> | "<question summary>"`

If the user says no, still log the query:
- `wiki/log.md`: `[date] query | — | "<question summary>" — not saved`

Saving valuable queries is how the wiki compounds: today's question becomes
tomorrow's starting point.
