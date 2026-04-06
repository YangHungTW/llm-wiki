---
name: wiki-query
description: Ask a question against the wiki and get a synthesized answer with citations.
disable-model-invocation: true
allowed-tools: Bash Read Write Edit Glob Grep
---

# Query Wiki

Question: $ARGUMENTS

**Important:** Read `WIKI_LANGUAGE` from the vault's config (via `<WIKI_ROOT>/.llm-wiki-config`). Answers and any saved wiki pages must be written in this language. Frontmatter keys remain in English.

1. If no question was provided, ask the user what they want to know.
2. Read `wiki/index.md` to find relevant pages.
3. Read the relevant wiki pages.
4. Synthesize an answer with citations (link to wiki pages and/or raw sources using `[[wikilinks]]`).
5. Choose an appropriate output format based on the question:
   - **Markdown page** — default for most answers.
   - **Comparison table** — for side-by-side analysis.
   - **Slide deck (Marp)** — for presentation-style summaries.
   - **Chart (matplotlib/other)** — for data-driven answers.
   - **Canvas** — for visual relationship maps.
6. If the answer is substantial and reusable, offer to save it as an **analysis** or **comparison** page in `wiki/` with proper YAML frontmatter.
7. If saved, update `wiki/index.md` and append to `wiki/log.md` using format: `## [YYYY-MM-DD] query | <question summary>`
8. If any wiki pages were created or updated, ask the user if they want to commit. If yes, run: `git add -A && git commit -m "[query] <question summary>"`
