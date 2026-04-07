# LLM Wiki — Knowledge Base Schema

This is a personal knowledge base maintained by an LLM, following the [LLM Wiki pattern](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f). The LLM builds and maintains a persistent, interlinked wiki from raw sources. The human curates sources and asks questions; the LLM does all the bookkeeping.

## Architecture

```
knowledge-base/
├── raw/            # Immutable source documents (articles, papers, notes, images)
│   └── assets/     # Downloaded images referenced by sources
├── wiki/           # LLM-generated and maintained markdown pages
│   ├── index.md    # Content catalog — every wiki page listed with summary
│   ├── log.md      # Chronological record of all operations
│   ├── overview.md # High-level synthesis of the entire knowledge base
│   ├── sources/    # One summary page per ingested source
│   ├── entities/   # Pages for people, organizations, products, places
│   ├── concepts/   # Pages for ideas, theories, frameworks, patterns
│   └── analyses/   # Comparisons, syntheses, query results worth keeping
├── CLAUDE.md       # This file — schema and conventions
└── .obsidian/      # Obsidian vault config
```

## Conventions

### Wiki pages

- All wiki pages are markdown files with YAML frontmatter
- Frontmatter fields:
  ```yaml
  ---
  title: Page Title
  type: source | entity | concept | analysis | overview
  created: YYYY-MM-DD
  updated: YYYY-MM-DD
  tags: [tag1, tag2]
  sources: [source-filename-1, source-filename-2]  # which raw sources informed this page
  ---
  ```
- Use `[[wikilinks]]` for cross-references between wiki pages (Obsidian-compatible)
- Link format: `[[category/page-name]]` (e.g., `[[entities/openai]]`, `[[concepts/attention-mechanism]]`)
- File names: lowercase, hyphens for spaces (e.g., `transformer-architecture.md`)
- Keep pages focused — one entity/concept per page
- Include a "References" section at the bottom listing source links

### index.md

- Organized by category (Sources, Entities, Concepts, Analyses)
- Each entry: `- [[category/page-name]] — one-line summary`
- Updated on every ingest and when new pages are created
- The LLM reads this first when answering queries to find relevant pages

### log.md

- Append-only chronological record
- Entry format: `## [YYYY-MM-DD] operation | Title`
- Operations: `ingest`, `query`, `lint`, `update`, `create`
- Each entry includes a brief description of what changed
- Parseable with: `grep "^## \[" wiki/log.md | tail -5`

## Operations

### Ingest

When the user adds a new source to `raw/` and asks to ingest it:

1. Read the source document fully
2. Discuss key takeaways with the user
3. Create a summary page in `wiki/sources/`
4. Create or update relevant entity pages in `wiki/entities/`
5. Create or update relevant concept pages in `wiki/concepts/`
6. Update `wiki/index.md` with new/updated pages
7. Append an entry to `wiki/log.md`
8. Update `wiki/overview.md` if the source shifts the big picture
9. Add cross-references (`[[wikilinks]]`) across all touched pages

A single source typically touches 5-15 wiki pages. Take your time — thoroughness matters more than speed.

### Query

When the user asks a question:

1. Read `wiki/index.md` to find relevant pages
2. Read the relevant wiki pages
3. Synthesize an answer with citations to wiki pages
4. If the answer produces a valuable analysis, offer to save it as a new page in `wiki/analyses/`
5. If the query reveals gaps, suggest sources to investigate
6. Log the query in `wiki/log.md`

### Lint

When the user asks to lint/health-check the wiki:

1. Check for contradictions between pages
2. Find orphan pages (no inbound links)
3. Find mentioned-but-missing pages (referenced but don't exist)
4. Check for stale information that newer sources may have superseded
5. Verify all index entries point to existing pages
6. Suggest new questions or sources to investigate
7. Log the lint pass in `wiki/log.md`

## Guidelines

- Never modify files in `raw/` — they are immutable source documents
- Always update `index.md` and `log.md` when making wiki changes
- Prefer updating existing pages over creating new ones when the topic overlaps
- Flag contradictions explicitly rather than silently overwriting
- When uncertain about categorization, ask the user
- Images: if a source references images, note them and offer to analyze separately
- The wiki is a git repo — commit after significant operations if the user wants version history
