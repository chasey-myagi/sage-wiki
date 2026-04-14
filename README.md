# sage-wiki

A Claude Code skill plugin for building and maintaining an LLM-powered personal knowledge wiki.

Implements the [Karpathy LLM Wiki pattern](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f) with improvements from the community (confidence scoring, supersession tracking, structured contradiction handling from [LLM Wiki v2](https://gist.github.com/rohitg00/2067ab416f7bbe447c1977edaaa681e2)).

## Core idea

Don't use the LLM as a search engine over your raw documents. Use it as a knowledge engineer who reads, synthesizes, cross-references, and maintains a living wiki that compounds over time.

> "The tedious part of maintaining a knowledge base is not the reading or the thinking — it's the bookkeeping. LLMs excel at this maintenance burden that humans abandon." — Karpathy

## Skills

| Skill | What it does |
|-------|-------------|
| `sage-init` | Bootstrap the wiki structure and `SCHEMA.md` |
| `sage-ingest` | Process source documents into wiki pages |
| `sage-query` | Answer questions from wiki content (not general knowledge) |
| `sage-lint` | Health check: broken links, orphans, contradictions, gaps |
| `sage-update` | Revise pages with source citations and downstream checks |

## Wiki structure

```
your-wiki/
├── SCHEMA.md          ← conventions file (created by sage-init)
├── raw/               ← immutable source documents
├── wiki/
│   ├── pages/         ← flat slug-named Markdown pages
│   ├── index.md       ← master catalog
│   ├── overview.md    ← evolving synthesis
│   └── log.md         ← append-only operation log
└── assets/
```

## Improvements over [wiki-skills](https://github.com/kfchou/wiki-skills)

- **Confidence levels** (`high` / `medium` / `low`) on every page and claim
- **Supersession tracking** — `supersedes` / `superseded-by` frontmatter fields for when new knowledge invalidates old
- **Contradiction handling** — explicit `## Contradictions` sections rather than silent overwrites
- **Step 2 user consultation** in ingest — surface key insights and wait for user input before writing
- **Downstream impact check** in update — flags pages that rely on a changed claim

## Usage

After installing, just describe what you want:

```
"Set up a new wiki for my ML research"
→ Claude uses sage-init

"Ingest this paper into my wiki: raw/attention-is-all-you-need.pdf"
→ Claude uses sage-ingest

"What does my wiki say about sparse attention?"
→ Claude uses sage-query

"Run a health check on my wiki"
→ Claude uses sage-lint

"The wiki is wrong about BERT — update it with this new paper"
→ Claude uses sage-update
```

## License

MIT
