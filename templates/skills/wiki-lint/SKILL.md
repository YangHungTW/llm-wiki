---
name: wiki-lint
description: Health-check the wiki for contradictions, orphan pages, missing cross-references, and data gaps.
disable-model-invocation: true
allowed-tools: Bash Read Write Edit Glob Grep WebSearch WebFetch
---

# Lint Wiki

Run a health check on the wiki.

**Important:** Read `WIKI_LANGUAGE` from the vault's config (via `<WIKI_ROOT>/.llm-wiki-config`). Reports and any new/updated wiki pages must be written in this language. Frontmatter keys remain in English.

1. Read `wiki/index.md` to get a full list of pages.
2. Read all wiki pages and scan for:
   - Contradictions between pages.
   - Stale claims superseded by newer sources.
   - Orphan pages (no inbound links from other pages).
   - Important concepts mentioned but lacking their own page.
   - Missing cross-references that should exist.
   - Data gaps that could be filled with new sources or a web search. Use `WebSearch`/`WebFetch` to look up information that could fill identified gaps, and offer to add findings as new wiki pages.
3. Report findings to the user in a clear summary.
4. Suggest new questions to investigate and new sources to look for to fill knowledge gaps.
5. With user approval, apply fixes (update pages, add cross-references, create missing pages).
6. Append a lint entry to `wiki/log.md` using format: `## [YYYY-MM-DD] lint | <summary of findings>`
7. If any wiki pages were created or updated, ask the user if they want to commit. If yes, run: `git add -A && git commit -m "[lint] <summary of findings>"`
