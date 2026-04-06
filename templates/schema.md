# {{DOMAIN_NAME}} Wiki — Schema

You are an LLM agent maintaining a personal wiki. Follow these conventions precisely.

## Directory Structure

```
raw/            # Immutable source documents (never modify)
  assets/       # Images and attachments
wiki/           # LLM-maintained pages (you own this)
  index.md      # Content catalog — update on every change
  log.md        # Chronological activity log — append only
```

## Page Conventions

- All wiki pages are Markdown files in `wiki/`.
- Use `[[wikilinks]]` for cross-references between pages.
- Every page should have YAML frontmatter:

```yaml
---
title: Page Title
type: source | entity | concept | analysis | comparison
created: YYYY-MM-DD
updated: YYYY-MM-DD
sources: []          # list of source filenames this page draws from
tags: []
---
```

- Page types:
  - **source**: Summary of a single raw source document.
  - **entity**: A person, place, organization, tool, or other named thing.
  - **concept**: An idea, framework, pattern, or theme.
  - **analysis**: A synthesis, comparison, or investigation (often from queries).
  - **comparison**: Side-by-side analysis of two or more things.

## Workflows

### Ingest

When the user adds a new source to `raw/` and asks you to process it:

1. Read the source document fully.
2. Discuss key takeaways with the user.
3. Create a **source** page in `wiki/` summarizing the document.
4. Create or update **entity** and **concept** pages for important items mentioned.
5. Update `wiki/index.md` with new/changed pages.
6. Append an entry to `wiki/log.md`.
7. Review existing pages for connections, contradictions, or updates needed.

### Query

When the user asks a question:

1. Read `wiki/index.md` to find relevant pages.
2. Read the relevant pages.
3. Synthesize an answer with citations (link to wiki pages and/or raw sources).
4. Choose an appropriate output format based on the question:
   - **Markdown page** — default for most answers.
   - **Comparison table** — for side-by-side analysis.
   - **Slide deck (Marp)** — for presentation-style summaries.
   - **Chart (matplotlib/other)** — for data-driven answers.
   - **Canvas** — for visual relationship maps.
5. If the answer is substantial and reusable, offer to save it as an **analysis** or **comparison** page.
6. If saved, update `wiki/index.md` and append to `wiki/log.md`.

### Lint

When the user asks for a health check:

1. Scan all pages for:
   - Contradictions between pages.
   - Stale claims superseded by newer sources.
   - Orphan pages (no inbound links).
   - Important concepts mentioned but lacking their own page.
   - Missing cross-references.
   - Data gaps that could be filled.
2. Report findings and suggest fixes.
3. Suggest new questions to investigate and new sources to look for to fill knowledge gaps.
4. With user approval, apply fixes.
5. Append a lint entry to `wiki/log.md`.

### Create New Vault

When the user asks to create a new domain vault:

1. Read the config file at `{{WIKI_ROOT}}/.llm-wiki-config` to get `WIKI_ROOT`, `SCHEMA_FORMAT`, `INSTALL_PATH`, and `USE_OBSIDIAN`.
2. Follow the **Vault Creation Guide** section (Vault Steps 1–5) in the installation guide at `INSTALL_PATH`.
3. Use the same `WIKI_ROOT` and `SCHEMA_FORMAT` from the config.

## NotebookLM Integration

NotebookLM can be used alongside the wiki in several ways. The user decides which approach to use on a case-by-case basis.

### Approach A: NotebookLM outputs as sources

NotebookLM-generated content (notes, summaries, Audio Overview transcripts) can be ingested as wiki sources:

1. User exports content from NotebookLM (copy-paste, download transcript, etc.).
2. Save the export to `raw/` with a descriptive filename, e.g. `notebooklm-summary-topic.md`.
3. Follow the standard **Ingest** workflow.
4. In the source page frontmatter, add `origin: notebooklm` to distinguish from primary sources.

### Approach B: Shared raw sources

The same documents can be uploaded to both NotebookLM and placed in `raw/`:

- Use NotebookLM for quick interactive exploration and Q&A.
- Use the wiki for long-term structured accumulation.
- Insights discovered in NotebookLM can be brought into the wiki via Approach A.

### Approach C: NotebookLM as a triage filter

Use NotebookLM to quickly evaluate a batch of sources before committing them to the wiki:

1. Upload candidate sources to NotebookLM.
2. Use NotebookLM to explore, ask questions, and assess relevance.
3. Sources worth keeping get moved to `raw/` and ingested into the wiki.
4. Optionally, save NotebookLM's exploration notes to `raw/` as a secondary source (Approach A).

### Logging

When ingesting NotebookLM outputs, append to `wiki/log.md` with the action `notebooklm-ingest`, e.g.:

```
## [2026-04-06] notebooklm-ingest | Audio Overview transcript on topic X
```

## Working with Images

LLMs cannot natively read markdown with inline images in one pass. When a source or wiki page contains images:

1. Read the text content of the page first.
2. View referenced images separately to gain additional context.
3. Integrate insights from both text and images into the wiki page.

When images are referenced by URL, prefer downloading them to `raw/assets/` and updating the link to a local path. This prevents broken links over time.

## Scaling

At small scale (~100 sources, hundreds of pages), the `wiki/index.md` file is sufficient for navigation and query routing. As the wiki grows, consider adding a search tool:

- [qmd](https://github.com/tobi/qmd) — local markdown search with hybrid BM25/vector search and LLM re-ranking. Has both CLI and MCP server modes.
- Or build a simpler search script as needed — even a basic grep wrapper can help at moderate scale.

The agent should suggest adding search tooling when `index.md` becomes unwieldy or when queries start missing relevant pages.

## Rules

- **Write wiki content in `WIKI_LANGUAGE`** as configured in `{{WIKI_ROOT}}/.llm-wiki-config`. Frontmatter keys, filenames (`index.md`, `log.md`), and structural elements remain in English.
- **Never modify files in `raw/`.** Sources are immutable.
- **Always update `index.md`** when creating or modifying wiki pages.
- **Always append to `log.md`** for ingests, significant queries, and lint passes.
- **Use wikilinks** (`[[Page Name]]`) for all cross-references.
- **Cite sources** — every claim in the wiki should trace back to a raw source.
- **Flag contradictions** — when new data conflicts with existing pages, note it explicitly.
- **Ask before deleting** — never remove a wiki page without user confirmation.
