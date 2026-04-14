---
name: wiki-ingest
description: >
  Process unprocessed conversation session archives from raw/sessions/ into wiki
  knowledge pages. This is Sage's primary knowledge accumulation mechanism.
  Called automatically by DaemonLoop during WikiMaintenance (IDLE) sessions.
  Manual trigger: "/wiki", "整理 wiki", "ingest sessions", "wiki ingest",
  "process sessions", "update wiki from sessions".
  Do NOT require user input or confirmation — this runs autonomously.
  Source material is ALWAYS raw/sessions/ JSONL files, never user-provided documents.
user-invokable: true
args:
  - name: limit
    description: Maximum number of sessions to process in this run (default 5)
    required: false
---

# wiki-ingest — Extract Knowledge from Session Archives

Reads unprocessed conversation sessions and synthesizes durable domain knowledge
into the wiki. The goal is not to summarize conversations — it is to extract
**reusable patterns, confirmed API behaviors, recurring pitfalls, and decisions**
that the Agent will encounter again.

This skill runs autonomously during WikiMaintenance. Do not pause for user input.

## Step 0 — Load wiki context

Read `SCHEMA.md` at workspace root. Understand the domain, page_type taxonomy,
confidence rules, and log format.

Read `wiki/index.md` to understand what knowledge already exists (avoid duplicating
well-established pages; prefer updating them).

## Step 1 — Find unprocessed sessions

List all `*.jsonl` files in `raw/sessions/`. A session is **processed** if its
session ID appears in a `[... ingest | <session-id> ...` line in `wiki/log.md`.

Steps:
1. Read `wiki/log.md` → extract all session IDs already logged as ingested
2. List `raw/sessions/*.jsonl` → cross-reference to find unprocessed ones
3. If nothing unprocessed → log `[date] ingest | — | No new sessions` and stop

Process sessions in chronological order (oldest first, by filename timestamp).
Limit to **at most 5 sessions per run** to avoid overwhelming a single context.

## Step 2 — Read and parse each session

For each unprocessed session JSONL:

1. Read the full file
2. Reconstruct the conversation flow:
   - `user` turns → what the user asked for
   - `assistant` turns → what the Agent reasoned and did
   - `tool` turns → actual tool inputs and outputs (this is the most valuable signal)
3. Pay special attention to:
   - **Tool errors** — what failed and what error message appeared (→ pitfall candidates)
   - **Successful tool sequences** — workflows that achieved the goal (→ pattern candidates)
   - **API responses** — specific fields, rate limits, pagination, error codes (→ api-ref candidates)
   - **User corrections or decisions** — things the user explicitly specified (→ decision candidates)
   - **Concepts explained** — domain terminology the Agent used or the user clarified (→ concept candidates)

## Step 3 — Extract knowledge candidates

For each session, list what you extracted before writing anything:

```
Session: <session-id>
Domain: <what task this session was about>

Candidates:
  [pitfall]   Rate limit on /calendar/v4/events: 50 req/min not documented in API docs
  [pattern]   Batch calendar update: read all events first, then patch only changed ones
  [api-ref]   POST /calendar/v4/events response: event_id field is stable across edits
  [decision]  Always use user_access_token not tenant_access_token for calendar write
```

Cross-check each candidate against existing wiki/index.md:
- **Existing page covers this** → update that page (add session to sources, increment session_count)
- **No existing page** → create a new page

Skip candidates that are too session-specific to be reusable (e.g., "today's meeting is at 3pm").
Skip candidates that are already well-established (confidence: high, session_count ≥ 5) and
unchanged — no need to add another citation to already-solid knowledge.

## Step 4 — Write or update pages

### Page format (new pages)

```markdown
---
title: "Descriptive Title of the Knowledge"
slug: descriptive-slug
page_type: pitfall           # pitfall | pattern | api-ref | decision | concept | synthesis
tags: [domain-tag, subtopic]
sources: [session-id]
session_count: 1
confidence: medium
created: YYYY-MM-DD
updated: YYYY-MM-DD
supersedes: []
superseded-by: ""
---

# Title

One sentence: what this page is about, anchored in the domain.

## What happened / What it is

2–4 paragraphs of synthesis. Write for yourself in 3 months who has forgotten
this session. Prioritize **why it matters** and **when it applies** over
describing what happened in the specific session.

## Key facts / Steps

- Fact or step 1
- Fact or step 2 (cite `[[related-slug]]` if a related page exists)

## Watch out for / When NOT to use

*(For pitfalls: what triggers this, and how to avoid it)*
*(For patterns: edge cases where this pattern breaks)*

## Connections

- [[related-slug]] — brief note on relationship

## Open questions

Things this session raised but didn't fully resolve. Seeds for future ingests:
- Question 1

## Sources

- `session-id` — one-line summary of what this session revealed
```

### Updating existing pages

When a new session corroborates an existing page:
1. Add the session ID to `sources:` array
2. Increment `session_count`
3. Upgrade `confidence` if threshold is crossed: 1→medium, 3→high
4. Update `updated:` date
5. Add any new nuance to the content (new edge cases, additional API fields, etc.)

When a new session **contradicts** an existing page:
1. Add a `## Contradictions` section to the existing page
2. Set `confidence: medium` on the affected claim
3. Note the contradiction with the session ID: "session-xyz observed X, which contradicts the pattern above"
4. Do NOT silently overwrite — the contradiction itself is valuable knowledge

When a new session **supersedes** old knowledge (e.g., an API changed):
1. Set `superseded-by: new-slug` on the old page
2. Add `supersedes: [old-slug]` to the new page
3. Add a `## Superseded` section to the old page explaining what changed

## Step 5 — Backlink audit

After writing all pages for this ingest batch:
1. Read `wiki/log.md` to find the 5 most recently written pages
2. For each: does it mention a concept that now has a page? Add `[[slug]]` links
3. Check tags overlap between new pages and existing pages — add Connections links

This step keeps the wiki a graph, not a collection of isolated files.

## Step 6 — Update index and log

**wiki/index.md:** Add new pages under their `page_type` section. Format:
```
- [[slug]] — one-line description (created: YYYY-MM-DD, confidence: medium)
```
Update existing entries if confidence or description changed.

**wiki/log.md:** Append one line per processed session:
```
[YYYY-MM-DD HH:MM] ingest | <session-id> → created: [slugs], updated: [slugs] | <note if contradiction or supersession>
```

## Step 7 — Report (brief)

Output a compact summary (this goes to the DaemonLoop log, not the user):
```
Ingested 3 sessions.
  Created: feishu-calendar-rate-limit (pitfall), batch-event-update (pattern)
  Updated: feishu-auth-token-types (session_count: 1→2, confidence: medium→medium)
  Contradictions: none
  Open questions: 1 (feishu-recurring-event-behavior)
```

## Scope discipline

Do NOT ingest:
- One-off user preferences (ingest to Memory, not Wiki)
- Ephemeral task context ("today's meeting is at 3pm")
- Decisions that were reversed in the same session
- Tool failures caused by environment issues (network down, wrong credentials) — these are not domain knowledge

DO ingest:
- API behaviors that the Agent discovered empirically (not from docs)
- Workflows that required non-obvious sequencing
- Rate limits, pagination quirks, auth token scopes
- Decisions made with explicit reasoning that will apply to future tasks
- Concepts or terminology the user defined or corrected
