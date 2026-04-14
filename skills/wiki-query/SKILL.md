---
name: wiki-query
description: >
  Query the Agent's domain wiki to recall accumulated knowledge before or during
  task execution. Use this skill when the Agent needs to remember domain patterns,
  API behaviors, past decisions, or known pitfalls relevant to the current task.
  This skill answers from wiki content ONLY — not from general training knowledge.
  Trigger when: starting a task in a domain covered by the wiki, encountering an
  API or workflow that might have documented patterns, user asks "what do you know
  about X", Agent is unsure about a domain behavior it may have seen before.
  This skill runs autonomously — do not pause for user confirmation.
user-invokable: true
args:
  - name: topic
    description: The domain topic, API, or concept to look up
    required: false
---

# wiki-query — Recall Domain Knowledge

Answers domain questions by reading wiki pages, not from training data. The wiki
reflects **this Agent's empirically accumulated understanding** of its specific
domain — it should be consulted before attempting any non-trivial domain task.

## When to use proactively

Before starting a task, quickly assess: "Could I have seen this before?"

- About to call an API this Agent has used before → query for api-ref and pitfall pages
- About to run a workflow pattern → query for pattern pages
- User mentioned a concept that sounds domain-specific → query for concept pages
- Previously failed at something similar → query for pitfall pages

A quick query costs a few tool calls. Encountering a known pitfall fresh costs
multiple turns. Default to querying when uncertain.

## Step 0 — Load wiki context

Read `SCHEMA.md` at workspace root to understand page_type taxonomy and domain.
Read `wiki/index.md` to see what the wiki contains at a glance.

## Step 1 — Parse the query topic

Extract the core concept or question. If the task context implies multiple
sub-questions, prioritize the one most likely to cause failure if unknown.

Do not start reading pages hoping to stumble on the answer — identify targets first.

## Step 2 — Identify relevant pages

From `wiki/index.md`, identify 3–6 pages most likely to be relevant:
- Match by page_type (if you're about to call an API → api-ref and pitfall first)
- Match by tags (domain tags overlap with task domain)
- Match by slug keywords

Also check `wiki/overview.md` for high-level orientation if the domain is new to you.

## Step 3 — Read relevant pages

Read each identified page. Follow `[[slug]]` links ONE level deep if a linked page
is directly relevant to the question — do not traverse deeper unless necessary.

Pay attention to:
- `confidence` level — high (3+ sessions) is reliable; medium is plausible; low is a hypothesis
- `session_count` — more sessions = more battle-tested
- `## Watch out for` / `## Contradictions` sections — these are the most operationally valuable parts

## Step 4 — Synthesize and use the knowledge

**Ground your answer in wiki content.** If the wiki has a clear answer:
- Apply the knowledge directly to the task
- Note which page(s) you're relying on: `(applying [[slug]])`
- Flag the confidence level if it's medium or low: `(⚠ confidence: medium — treat as hypothesis)`

**If two pages contradict each other:**
Surface the contradiction before proceeding:
> "[[page-a]] and [[page-b]] have conflicting information about X. I'll proceed
> with [[page-a]]'s approach (higher confidence) and flag this for wiki-update."

**If the wiki has no relevant knowledge:**
Say so explicitly rather than falling back to training data for domain specifics:
> "My wiki has no accumulated knowledge about X. I'll proceed carefully and
> treat any findings as new knowledge to ingest after this session."
(This honest gap-flagging is what lets wiki-ingest capture new knowledge later.)

## Step 5 — Note gaps (optional, brief)

If the query revealed meaningful gaps worth capturing:
```
[wiki-gap] No page on <topic>. Relevant to current task — will be captured in next ingest.
```

Do NOT create pages mid-task. Knowledge capture happens in the dedicated
WikiMaintenance session via wiki-ingest, not during UserDriven task execution.

## Step 6 — Log the query (skip if trivial)

For non-trivial queries that surfaced useful knowledge or meaningful gaps,
append to `wiki/log.md`:
```
[YYYY-MM-DD HH:MM] query | <slugs read> | "<topic summary>" [gap: <topic> if applicable]
```

Skip logging for routine lookups that found nothing useful.
