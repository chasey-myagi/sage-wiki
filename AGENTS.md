# sage-wiki — Agent Instructions

Wiki skills for building and maintaining a domain-expert knowledge base that accumulates
from conversation sessions. Implements the Karpathy LLM Wiki pattern, adapted for
autonomous Agent operation.

## When to use which skill

| Situation | Skill to use |
|-----------|-------------|
| Setting up wiki for the first time | `wiki-init` |
| IDLE time, unprocessed sessions exist | `wiki-ingest` |
| About to start a domain task | `wiki-query` |
| Wiki feels disorganized, or after 10+ ingests | `wiki-lint` |
| Current observation contradicts a wiki page | `wiki-update` |

## Workspace layout (expected)

```
workspace/
├── SCHEMA.md              ← read this first — governs all wiki operations
├── raw/sessions/          ← source material (JSONL conversation archives)
└── wiki/
    ├── pages/             ← flat slug-named Markdown pages
    ├── index.md           ← master catalog
    ├── overview.md        ← evolving synthesis
    └── log.md             ← append-only log (source of truth for processed sessions)
```

## Core rules

1. **SCHEMA.md first** — always read it before any wiki operation
2. **wiki/log.md is the processed-session ledger** — sessions in an ingest entry are processed; never re-ingest them
3. **Don't answer from training data** — wiki-query is grounded in wiki content only
4. **Don't ingest during UserDriven tasks** — knowledge capture happens in dedicated WikiMaintenance sessions
5. **Contradictions are knowledge** — never silently overwrite conflicting claims; document both sides

## Page types

Every wiki page has a `page_type` frontmatter field:

- `pitfall` — things that failed, gotchas, undocumented limits
- `pattern` — successful workflows worth repeating
- `api-ref` — API endpoint behaviors confirmed from real sessions
- `decision` — decisions made with rationale
- `concept` — domain terminology and definitions
- `synthesis` — cross-topic analysis

## Confidence model

Confidence is driven by independent session corroboration:

- `low` — 1 session, inferred — treat as hypothesis
- `medium` — 1–2 sessions, directly observed — use with awareness
- `high` — 3+ sessions, consistent — rely on it

## Skill: wiki-init

Bootstrap the wiki structure. Call once per agent workspace.

**When:** First run, or wiki directory is missing.
**Does:** Creates `wiki/` directory tree, writes `SCHEMA.md`, seeds empty `index.md`, `log.md`, `overview.md`.
**Does not:** Require user input if defaults are acceptable.

## Skill: wiki-ingest

Extract domain knowledge from unprocessed conversation sessions.

**When:** IDLE time with unprocessed sessions in `raw/sessions/`, or manual `/wiki` trigger.
**Source:** `raw/sessions/*.jsonl` — conversation archives. Never user documents.
**Process:** Read sessions → extract pitfalls/patterns/api-refs/decisions → write wiki pages → mark processed in log.
**Limit:** Max 5 sessions per run to avoid context overflow.
**Autonomous:** No user confirmation needed.

## Skill: wiki-query

Recall accumulated domain knowledge before or during a task.

**When:** About to use a domain API, workflow, or concept that might be in the wiki.
**Answers from:** Wiki pages only — cite `([[slug]])` for every claim.
**Honest about gaps:** If wiki doesn't have it, say so; don't fill in from training data.

## Skill: wiki-lint

Health audit of the wiki.

**When:** After every 5–10 ingests, or when wiki feels stale/disorganized.
**Checks:** Broken links, orphaned pages, contradictions, stale confidence, unprocessed session backlog, implied concepts.
**Outputs:** `wiki/pages/lint-YYYY-MM-DD.md` lint report.

## Skill: wiki-update

Revise wiki pages with new evidence.

**When:** Current session observation contradicts a wiki page, or user corrects the wiki.
**Change types:** refinement / correction / supersession / contradiction.
**Downstream check:** Always check pages that link to the updated page.
