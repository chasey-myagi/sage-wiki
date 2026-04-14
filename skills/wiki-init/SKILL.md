---
name: wiki-init
description: >
  Initialize or reset the wiki structure inside a Sage Agent workspace.
  Use this skill when setting up a new agent workspace for the first time,
  or when the wiki directory is missing/corrupt.
  Normally called automatically by `sage init --agent <name>`.
  Manual trigger: "initialize wiki", "reset wiki structure", "wiki init",
  "set up wiki", "create wiki".
  Do NOT call this if wiki/index.md already exists and has pages — use wiki-lint
  instead to assess the health of an existing wiki.
user-invokable: false
---

# wiki-init — Bootstrap the Wiki Structure

Initializes the wiki directory structure inside the current Agent workspace.
The workspace root is `/workspace` inside the sandbox (maps to
`~/.sage/agents/<name>/workspace/` on the host).

## Pre-flight

1. Check if `SCHEMA.md` exists at workspace root (`/workspace/SCHEMA.md`).
   - If yes → ask whether to reinitialize (overwrites `SCHEMA.md` only) or abort.
   - If no → proceed silently (normal first-run).

2. Read `agent.yaml` if accessible, to extract `name` and `description` fields
   for seeding SCHEMA.md. If not accessible, use reasonable defaults.

## Directory structure to create

```
/workspace/
├── SCHEMA.md              ← conventions + wiki root pointer (this file governs all wiki skills)
├── AGENT.md               ← Agent cognitive framework (injected into system prompt; create empty)
├── memory/
│   └── MEMORY.md          ← memory index (injected into system prompt; create empty)
├── raw/
│   └── sessions/          ← conversation archives (JSONL, written by DaemonLoop; never edit)
├── wiki/
│   ├── pages/             ← flat slug-named Markdown pages (all knowledge lives here)
│   ├── index.md           ← master catalog grouped by page_type and tag
│   ├── overview.md        ← evolving high-level synthesis, rewritten during lint passes
│   └── log.md             ← append-only operation log (source of truth for processed sessions)
├── assets/                ← attachments (images, PDFs, etc.)
├── craft/                 ← Agent-managed reusable artifacts (SOP / scripts / templates)
└── metrics/               ← TaskRecord JSON files (written by MetricsCollector; never edit)
```

`wiki/pages/` must stay **flat** — no subdirectories. All pages are lowercase
hyphen-separated slug filenames (e.g. `feishu-calendar-rate-limit.md`).

## Write SCHEMA.md

```markdown
# Workspace Schema

## Agent Identity
- **Name:** <from agent.yaml or ask>
- **Domain:** <from agent.yaml description, or ask: "What domain does this agent serve?">
- **Created:** <today's date>
- **Wiki root:** `wiki/`
- **Session archive:** `raw/sessions/`

## Wiki page types
Each wiki page has a `page_type` frontmatter field:

| page_type | When to create |
|-----------|---------------|
| `pitfall` | Something failed or caused unexpected behavior — document the gotcha and workaround |
| `pattern` | A successful approach worth repeating — capture the method and why it works |
| `api-ref` | API endpoint, parameter, or response format details confirmed from sessions |
| `decision` | An architectural or design decision made in a session, with rationale |
| `concept` | Domain concept definition — what something IS in this domain |
| `synthesis` | Cross-topic analysis connecting multiple pages |

## Page frontmatter (required)
```yaml
---
title: "Human-Readable Title"
slug: page-slug
page_type: pitfall        # pitfall | pattern | api-ref | decision | concept | synthesis
tags: [tag1, tag2]
sources: [session-id-1]   # session IDs from raw/sessions/ that support this page
session_count: 1          # how many independent sessions confirmed this knowledge
confidence: medium        # high (3+ sessions) | medium (1-2 sessions) | low (inferred)
created: YYYY-MM-DD
updated: YYYY-MM-DD
supersedes: []
superseded-by: ""
---
```

## Confidence levels
- **high** — 3 or more sessions independently confirm this; well-established pattern
- **medium** — 1–2 sessions; plausible but not battle-tested
- **low** — inferred from context or single ambiguous signal; treat as hypothesis

## Link syntax
Use `[[slug]]` for internal links. Every claim should cite its supporting session.

## Naming conventions
- Page slugs: `lowercase-hyphen-separated`
- Session slugs: match the JSONL filename in `raw/sessions/` (without extension)
- Tags: lowercase, singular, domain-specific (e.g. `feishu`, `calendar`, `rate-limit`)

## Log entry format (wiki/log.md)
```
[YYYY-MM-DD HH:MM] <operation> | <sessions or slugs affected> | <brief note>
```

Sessions listed in log.md ingest entries are considered **processed**.
wiki-ingest uses this log as the source of truth — do not edit it manually.
```

## Seed files

**wiki/index.md**
```markdown
# Wiki Index

_Last updated: <today>_

## Pitfalls

_No pages yet._

## Patterns

_No pages yet._

## API References

_No pages yet._

## Decisions

_No pages yet._
```

**wiki/overview.md**
```markdown
# Domain Overview

_This file is maintained by wiki-lint. It will be written after the first lint pass._
```

**wiki/log.md**
```markdown
# Operation Log

[<today>] init | — | Wiki initialized
```

## Finishing up

Tell the user (or DaemonLoop):
- Wiki structure created at `/workspace/wiki/`
- Sessions will be ingested automatically from `raw/sessions/` during IDLE
- Run `wiki-lint` after first few ingests to check health
