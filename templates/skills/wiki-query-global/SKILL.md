---
name: wiki-query
description: Query one or more LLM Wiki vaults from anywhere. Supports single-vault lookup and cross-vault comparison.
allowed-tools: Bash Read Write Edit Glob Grep
---

# Wiki Query

Arguments: $ARGUMENTS

## Parse Arguments

Arguments format: `<vault-spec> <question>`

- `<vault-spec>` is one of:
  - A vault name (e.g. `monaco-rails`) — matches against the vault directory name
  - A comma-separated list (e.g. `monaco-rails,payments`) — cross-vault comparison
  - `?` — list available vaults and ask the user to choose
- `<question>` is everything after the first word

If no arguments are provided, ask the user for the vault and question separately.

## Step 1: Locate Vaults

1. Read `~/Wiki/.llm-wiki-config` to get `WIKI_ROOT` and `WIKI_LANGUAGE`.
2. List available vaults:
   ```bash
   ls "$WIKI_ROOT"
   ```
3. If vault-spec is `?`, show the list and ask the user to choose one or more.
4. Otherwise, match the vault-spec against directory names (partial match is fine, e.g. `monaco` matches `vault__mainapp__monaco-rails__yield`). If ambiguous, show matches and ask.
5. Confirm all resolved vault paths exist.

## Step 2: Load Index(es)

For each vault, read `<vault-path>/wiki/index.md` to understand what pages are available.

## Step 3: Answer

### Single vault

1. Identify relevant pages from the index.
2. Read those pages.
3. Synthesize an answer with citations using `[[wikilinks]]`.
4. Choose output format:
   - **Markdown** — default
   - **Comparison table** — side-by-side analysis
   - **Marp slide deck** — presentation summary
   - **Chart (matplotlib/other)** — data-driven answers
   - **Canvas** — visual relationship maps
5. If the answer is substantial and reusable, offer to save it as an `analysis` or `comparison` page in `<vault>/wiki/` with YAML frontmatter:
   ```yaml
   ---
   title: <title>
   type: analysis
   created: <YYYY-MM-DD>
   updated: <YYYY-MM-DD>
   sources: []
   tags: []
   ---
   ```
6. If saved, update `wiki/index.md` and append to `wiki/log.md`:
   ```
   ## [YYYY-MM-DD] query | <question summary>
   ```
7. Ask the user if they want to commit. If yes:
   ```bash
   cd <vault-path> && git add -A && git commit -m "[query] <question summary>"
   ```

### Cross-vault comparison

1. For each vault, identify relevant pages and read them independently.
2. Synthesize a **comparison answer** that highlights:
   - Similarities
   - Differences
   - Gaps (one vault has information the other lacks)
3. Default output format is a comparison table or structured Markdown with per-vault sections.
4. If saving, choose which vault to save the comparison page in (ask the user), or save a copy in each.
5. Follow the same index/log/commit steps as single-vault above for each vault that receives a new page.

## Notes

- All answers must be written in `WIKI_LANGUAGE` from the config. Frontmatter keys stay in English.
- Never modify files in `raw/`. Sources are immutable.
- Cite pages with `[[wikilink]]` syntax so they are navigable in Obsidian.
