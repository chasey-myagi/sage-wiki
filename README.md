# sage-wiki

Wiki skills for [Sage](https://github.com/chasey-myagi/sage) — the local domain-expert Agent platform.

Implements the [Karpathy LLM Wiki pattern](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f) with improvements from the community (confidence scoring, supersession tracking, structured contradiction handling), **adapted for Sage's Agent-driven autonomous operation**.

## Core idea

Don't use the LLM as a search engine over raw documents. Use it as a knowledge engineer who reads conversation sessions, extracts durable domain patterns, and maintains a living wiki that compounds over time.

> "The tedious part of maintaining a knowledge base is not the reading or the thinking — it's the bookkeeping. LLMs excel at this maintenance burden that humans abandon." — Karpathy

In Sage, the Agent IS the knowledge engineer. Knowledge accumulates automatically from conversation history — no manual document ingestion required.

## How it works in Sage

```
UserDriven session completes
  → DaemonLoop archives conversation to raw/sessions/<id>.jsonl
  → IDLE: DaemonLoop triggers WikiMaintenance
  → Agent uses wiki-ingest to process unprocessed sessions
  → Domain patterns, pitfalls, API behaviors extracted → wiki/pages/
  → Next UserDriven task: Agent uses wiki-query to recall accumulated knowledge
```

## Skills

| Skill | Called by | What it does |
|-------|-----------|-------------|
| `wiki-init` | `sage init --agent` | Bootstrap workspace wiki structure and `SCHEMA.md` |
| `wiki-ingest` | DaemonLoop IDLE (automatic) | Extract domain knowledge from `raw/sessions/` JSONL archives |
| `wiki-query` | Agent during task execution | Recall accumulated knowledge before/during domain tasks |
| `wiki-lint` | Manual `/wiki lint` or after 10 ingests | Health check: broken links, orphans, session backlog, gaps |
| `wiki-update` | Agent when contradiction detected | Revise pages with new session evidence and downstream checks |

## Workspace structure (inside `/workspace`)

```
/workspace/
├── SCHEMA.md              ← conventions file (governs all wiki skills)
├── AGENT.md               ← Agent cognitive framework (injected into system prompt)
├── memory/
│   └── MEMORY.md          ← memory index (injected into system prompt)
├── raw/
│   └── sessions/          ← conversation archives (written by DaemonLoop; never edit)
├── wiki/
│   ├── pages/             ← flat slug-named Markdown pages
│   ├── index.md           ← master catalog grouped by page_type
│   ├── overview.md        ← evolving domain synthesis
│   └── log.md             ← append-only log (source of truth for processed sessions)
├── assets/
├── craft/                 ← Agent-managed reusable artifacts
└── metrics/               ← TaskRecord JSON files
```

## Page types

Sage wiki pages are domain-specific, not general knowledge:

| `page_type` | What it captures |
|------------|-----------------|
| `pitfall` | Things that failed — gotchas, undocumented limits, error patterns |
| `pattern` | Successful workflows — reusable sequences that achieve domain goals |
| `api-ref` | API endpoint behaviors confirmed from real sessions (not docs) |
| `decision` | Architectural/design decisions made in sessions, with rationale |
| `concept` | Domain concept definitions and terminology |
| `synthesis` | Cross-topic analysis connecting multiple pages |

## Confidence model

Confidence is driven by independent session corroboration, not source quality:

| Confidence | Threshold | Meaning |
|------------|-----------|---------|
| `low` | session_count = 1, inferred | Hypothesis — treat carefully |
| `medium` | session_count = 1–2, observed | Plausible — use with awareness |
| `high` | session_count ≥ 3, consistent | Battle-tested — rely on it |

## Key differences from general-purpose wiki skills

| Generic wiki | Sage wiki |
|---|---|
| Source: papers, URLs, user documents | Source: `raw/sessions/` JSONL (conversation archives) |
| Ingest: user-triggered, interactive | Ingest: automatic during DaemonLoop IDLE |
| Query: user asks questions | Query: Agent recalls knowledge during task execution |
| Location: user-chosen | Location: fixed at `/workspace/wiki/` |
| Confidence: source quality | Confidence: session_count corroboration |
| Page types: generic | Page types: pitfall/pattern/api-ref/decision/concept/synthesis |

## License

MIT
