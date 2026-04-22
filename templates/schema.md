# {{DOMAIN_NAME}} Wiki — Schema

You are an LLM agent maintaining a personal wiki. Follow these conventions precisely.

## Domain Context

<!-- Filled in during vault creation. The LLM uses this to guide ingest emphasis, page creation decisions, and query synthesis. -->

**Topic:** {{DOMAIN_DESCRIPTION}}

**Goals:** {{GOALS}}

**Key questions:**
{{KEY_QUESTIONS}}

This context should evolve. When the user's focus shifts or new themes emerge, update this section to reflect the current direction.

## Directory Structure

```
raw/            # Immutable source documents (never modify)
  assets/       # Images and attachments
wiki/           # LLM-maintained pages (you own this)
  index.md      # Content catalog — update on every change
  log.md        # Chronological activity log — append only
```

### Filename convention for `raw/`

When the LLM saves files to `raw/`, use a datetime prefix: `YYYY-MM-DD-<descriptive-name>.md` (e.g. `2026-04-07-project-api-codemap.md`). This tracks when the source was added. Files added manually by the user (e.g. via Obsidian Web Clipper) do not need to follow this convention.

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

## Page Creation Policy

When to create a new page vs. mention in passing:

- **Entity page**: Create when the entity appears in 2+ sources, or is central to the domain, or the user explicitly requests it. A single mention in passing does not warrant its own page — note it inline instead.
- **Concept page**: Create when a theme or pattern emerges across multiple sources, or when a concept is core to the domain's goals/key questions.
- **Merge vs. split**: Prefer fewer, richer pages over many thin ones. If two concepts are tightly coupled and always discussed together, a single page covering both is better. Split only when a page grows unwieldy or the concepts diverge.
- **When in doubt**: Mention in an existing page with a `<!-- TODO: consider dedicated page -->` comment. The next lint pass will surface these.

## Page Structure Guide

### Source page

```markdown
## Summary
2-3 paragraph overview of the source document.

## Key Takeaways
- Bullet list of the most important points.

## Connections
How this source relates to existing wiki pages (with [[wikilinks]]).

## Open Questions
Questions raised by this source that aren't yet answered in the wiki.
```

### Entity page

```markdown
## Overview
What/who this entity is, in 1-2 paragraphs.

## Key Facts
- Structured bullet list of important attributes.

## Appearances
Which sources mention this entity and in what context (with citations).

## Related
Links to related entities and concepts ([[wikilinks]]).
```

### Concept page

```markdown
## Definition
What this concept means, in plain language.

## Key Aspects
- The important dimensions or components of this concept.

## Evidence & Sources
What the sources say about this concept (with citations).

## Connections
How this concept relates to other concepts and entities in the wiki.
```

### Analysis / Comparison page

```markdown
## Question
The question or comparison being addressed.

## Findings
The synthesized answer, with citations to wiki pages and raw sources.

## Implications
What this means for the domain's goals or key questions.
```

## Contradiction Policy

When new information conflicts with existing wiki pages:

1. **Do not silently overwrite.** Preserve both the old and new claims.
2. **Mark explicitly** using a callout in the affected page(s):
   ```markdown
   > [!warning] Contradiction
   > [[Source A]] claims X, but [[Source B]] claims Y. (flagged YYYY-MM-DD)
   ```
3. **Recency default**: Newer sources are generally more reliable, but this is a default — not a rule. Domain-specific judgment applies (e.g. a primary source may outweigh a newer secondary one).
4. **Log significant contradictions** in `wiki/log.md`:
   ```
   ## [YYYY-MM-DD] contradiction | <brief description>
   ```
5. **Resolution**: When the user or a later source resolves a contradiction, remove the callout and note the resolution inline. Update the log.

## Tag Conventions

Tags help with Dataview queries and wiki navigation. Use these base tags and extend as the domain requires:

### Status tags
- `#status/draft` — page is incomplete or needs review
- `#status/reviewed` — user has confirmed the content
- `#status/stale` — content may be outdated (flagged by lint)

### Domain tags
<!-- Add domain-specific tags here as the wiki grows. Examples: -->
<!-- - #topic/architecture, #topic/security -->
<!-- - #type/interview, #type/paper, #type/blog -->

The LLM should suggest new tags when recurring themes emerge that aren't captured by existing ones. Propose additions in lint reports or during ingest.

## Workflows

### Ingest

When the user adds a new source to `raw/` and asks you to process it:

1. Read the source document fully.
2. Discuss key takeaways with the user.
3. Create a **source** page in `wiki/` summarizing the document. Follow the **Page Structure Guide** above.
4. Create or update **entity** and **concept** pages for important items mentioned. Follow the **Page Creation Policy** to decide what warrants its own page.
5. Update `wiki/index.md` with new/changed pages.
6. Append an entry to `wiki/log.md`.
7. Review existing pages for connections, contradictions, or updates needed. Follow the **Contradiction Policy** when conflicts arise.
8. Assign appropriate tags from the **Tag Conventions**.

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
5. If the answer is substantial and reusable, offer to save it as an **analysis** or **comparison** page. Follow the **Page Structure Guide**.
6. If saved, update `wiki/index.md` and append to `wiki/log.md`.

### Lint

When the user asks for a health check:

1. Scan all pages for:
   - Contradictions between pages (see **Contradiction Policy**).
   - Stale claims superseded by newer sources — mark with `#status/stale`.
   - Orphan pages (no inbound links).
   - Important concepts mentioned but lacking their own page (check `<!-- TODO: consider dedicated page -->` comments).
   - Missing cross-references.
   - Data gaps that could be filled.
   - Pages missing tags or using inconsistent tags.
2. Report findings and suggest fixes.
3. Suggest new questions to investigate and new sources to look for to fill knowledge gaps.
4. With user approval, apply fixes.
5. Append a lint entry to `wiki/log.md`.

### Create or Adopt Vault

When the user asks to create a new domain vault or adopt an existing directory:

1. Read the config file at `{{WIKI_ROOT}}/.llm-wiki-config` to get `WIKI_ROOT`, `SCHEMA_FORMAT`, `INSTALL_PATH`, and `USE_OBSIDIAN`.
2. Check if the target directory already exists.
   - **New vault:** Follow the **Vault Creation Guide** (Vault Steps 1–5) in the installation guide at `INSTALL_PATH`.
   - **Existing directory:** Follow the **Vault Adoption Guide** (Adopt Steps 1–6) in the installation guide at `INSTALL_PATH`.
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
- **Flag contradictions** — follow the **Contradiction Policy** above.
- **Ask before deleting** — never remove a wiki page without user confirmation.
- **Evolve the schema** — this document should grow with the wiki. When new conventions emerge (e.g. new page types, domain-specific tags, changed policies), update this schema to reflect them.
