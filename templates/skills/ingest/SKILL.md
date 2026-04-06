---
name: ingest
description: Process a new source document from raw/ into the wiki. Accepts an optional filename as argument.
disable-model-invocation: true
allowed-tools: Bash Read Write Edit Glob Grep
---

# Ingest Source

Process source document(s) into the wiki. Argument: $ARGUMENTS

Supports two modes:
- **Single**: `/ingest filename.md` — process one source interactively.
- **Batch**: `/ingest batch` or `/ingest all` — process all unprocessed files in `raw/` with less supervision.

## Single mode (default)

1. If no filename was provided, list files in `raw/` and ask the user which source to ingest. Exclude `raw/assets/`.
2. Read the source document fully.
3. Discuss key takeaways with the user.
4. Create a **source** page in `wiki/` summarizing the document. Include YAML frontmatter:
   ```yaml
   ---
   title: <title>
   type: source
   created: <today>
   updated: <today>
   sources: [<filename>]
   tags: []
   ---
   ```
   If the source originates from NotebookLM (filename contains `notebooklm`, or user confirms), add `origin: notebooklm` to the frontmatter.
5. Create or update **entity** and **concept** pages for important items mentioned. Each page should also have frontmatter. A single source may touch 10–15 wiki pages.
6. Update `wiki/index.md` with new/changed pages. Each entry follows: `- [[Page Name]] — one-line summary (YYYY-MM-DD, N sources)`
7. Append an entry to `wiki/log.md` using format: `## [YYYY-MM-DD] ingest | <source title>`
8. Review existing wiki pages for connections, contradictions, or updates needed. Flag contradictions explicitly.
9. Use `[[wikilinks]]` for all cross-references between pages.

## Batch mode

1. List all files in `raw/` (excluding `raw/assets/`).
2. Cross-reference with `wiki/index.md` to find files that have not been ingested yet.
3. For each unprocessed file, run steps 2–9 from single mode with less interaction — summarize key takeaways instead of discussing them, and proceed without waiting for user input on each source.
4. After all files are processed, present a summary of what was ingested and what pages were created or updated.
