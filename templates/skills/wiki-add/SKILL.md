---
name: wiki-add
description: Add a file, folder, URL, or pasted text to a vault's raw/ from anywhere, then optionally trigger ingest.
allowed-tools: Bash Read Write Edit Glob Grep WebFetch
---

# Wiki Add

Arguments: $ARGUMENTS

## Parse Arguments

Format: `<vault-spec> <source>`

- `<vault-spec>` — vault name (partial match OK), or `?` to list and choose
- `<source>` — one of:
  - Local file path (e.g. `/tmp/notes.md`, `~/Downloads/report.pdf`)
  - Local folder path (e.g. `~/Downloads/articles/`)
  - URL (e.g. `https://...`)
  - Omitted — paste mode (ask user to provide content)

If no arguments, ask for vault and source separately.

## Step 1: Resolve Vault

1. Read `~/Wiki/.llm-wiki-config` to get `WIKI_ROOT` and `WIKI_LANGUAGE`.
2. List vaults:
   ```bash
   ls "$WIKI_ROOT"
   ```
3. If vault-spec is `?`, show the list and ask the user to choose.
4. Match vault-spec against directory names (partial match). If ambiguous, show matches and ask.
5. Set `VAULT=<WIKI_ROOT>/<matched-vault>`. Confirm it exists.

## Step 2: Add Source

### File

```bash
cp "<source-path>" "$VAULT/raw/"
```

Confirm the filename with the user if the destination already exists (offer to rename or skip).

### Folder

```bash
cp -r "<source-path>" "$VAULT/raw/"
```

Show the user what will be copied (file count, top-level structure) before executing:
```bash
find "<source-path>" -type f | head -20
find "<source-path>" -type f | wc -l
```
Ask for confirmation before copying.

### URL

1. Fetch the page content using WebFetch.
2. Convert to clean Markdown:
   - Strip navigation, ads, footers — keep article body, headings, code blocks, images
   - Download images referenced in the article to `$VAULT/raw/assets/` and rewrite image links to local paths
   - Preserve the page title as the H1 heading
   - Add a source block at the top:
     ```markdown
     > Source: <url>
     > Fetched: <YYYY-MM-DD>
     ```
3. Derive a filename from the page title (lowercase, hyphens, `.md`). Confirm with user.
4. Write the file to `$VAULT/raw/<filename>.md`.

### Paste mode

Ask the user to paste the content. Ask for a filename. Write to `$VAULT/raw/<filename>.md`.

## Step 3: Confirm and Optionally Ingest

1. Show what was added:
   ```
   Added to <vault>/raw/:
     <filename(s)>
   ```
2. Ask: **「要現在 ingest 嗎？」**
   - **Yes** → proceed with ingest (Step 4)
   - **No** → done. Remind user they can run `/wiki-ingest <filename>` later from within the vault.

## Step 4: Ingest (if requested)

Follow the **Single mode** steps from `wiki-ingest`:

1. Read the source document fully.
2. Discuss key takeaways with the user.
3. Create a **source** page in `wiki/` with YAML frontmatter:
   ```yaml
   ---
   title: <title>
   type: source
   created: <YYYY-MM-DD>
   updated: <YYYY-MM-DD>
   sources: [<filename>]
   tags: []
   ---
   ```
   If origin is a URL, add `origin: url` and `source_url: <url>` to frontmatter.
4. Create or update **entity** and **concept** pages for important items.
5. Update `wiki/index.md`.
6. Append to `wiki/log.md`:
   ```
   ## [YYYY-MM-DD] ingest | <source title>
   ```
7. Review existing pages for connections or contradictions.
8. Ask if the user wants to commit:
   ```bash
   cd "$VAULT" && git add -A && git commit -m "[ingest] <source title>"
   ```

## Notes

- All wiki content must be written in `WIKI_LANGUAGE` from config. Frontmatter keys stay in English.
- Never modify existing files in `raw/`. If a destination file exists, ask before overwriting.
- Cite with `[[wikilinks]]` for Obsidian navigation.
