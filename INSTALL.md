# LLM Wiki — Installation Guide

This is an interactive installation playbook. An LLM agent (Claude Code, Codex, OpenCode, etc.) follows these steps with the user to set up LLM Wiki on a new machine.

**For the agent:** Follow the steps below in order. Each step marked with `[ASK]` requires user input before proceeding. Each step marked with `[VERIFY]` requires checking the result before moving on. Do not skip verification steps.

---

## Step 0: Language Preference

`[ASK]` Ask the user which language they prefer for this installation process:

- **English**
- **中文 (Chinese)**

Use the chosen language for all prompts, confirmations, and output messages for the remainder of the installation.

`[ASK]` Ask the user what language the wiki content (pages, summaries, analyses) should be written in. This is separate from the installation language above. Examples:

- **English**
- **中文 (Chinese)**
- **日本語 (Japanese)**
- Or any other language

Record this as `WIKI_LANGUAGE`. File structure (frontmatter keys, filenames like `index.md`, `log.md`) is always in English regardless of this choice.

---

## Step 1: Install Obsidian (Optional)

`[ASK]` Ask the user if they want to use Obsidian as a viewer/browser for the wiki. Explain that Obsidian is **optional** — the wiki is just markdown files and works with any editor. Obsidian adds graph view, wikilink navigation, and plugin support (Dataview, Marp, etc.).

If the user **does not want Obsidian**, record `USE_OBSIDIAN=false` and skip to Step 2.

If **yes**, ask if Obsidian is already installed.

If already installed, record `USE_OBSIDIAN=true` and skip to Step 2.

If not installed, guide the installation based on the detected platform:

### macOS

```bash
# Option A: Homebrew (preferred)
brew install --cask obsidian

# Option B: Manual download
# Download from https://obsidian.md/download
```

### Linux

```bash
# Option A: Snap
sudo snap install obsidian --classic

# Option B: Flatpak
flatpak install flathub md.obsidian.Obsidian

# Option C: AppImage
# Download from https://obsidian.md/download
```

### Windows

```powershell
# Option A: Winget
winget install Obsidian.Obsidian

# Option B: Scoop
scoop bucket add extras
scoop install obsidian

# Option C: Manual download
# Download from https://obsidian.md/download
```

`[VERIFY]` Confirm Obsidian is installed:

- macOS/Linux: `which obsidian || ls /Applications/Obsidian.app 2>/dev/null || flatpak list 2>/dev/null | grep -i obsidian || snap list 2>/dev/null | grep -i obsidian`
- Windows: `where obsidian` or check Start Menu

If verification fails but the user confirms it's installed (e.g., installed via .dmg or manual download), accept and proceed.

Record `USE_OBSIDIAN=true`.

---

## Step 2: Choose Wiki Root Directory

`[ASK]` Ask the user where wiki vaults should live on this machine.

Suggest a default based on platform:
- macOS/Linux: `~/Wiki`
- Windows: `%USERPROFILE%\Wiki`

The user may choose any path. Record this as `WIKI_ROOT`.

`[ACTION]` Create the directory if it doesn't exist:

```bash
mkdir -p <WIKI_ROOT>
```

`[VERIFY]` Confirm the directory exists and is writable:

```bash
test -d <WIKI_ROOT> && test -w <WIKI_ROOT> && echo "OK" || echo "FAIL"
```

---

## Step 3: Choose Schema Format

`[ASK]` Ask the user which schema file format to use. Explain that the schema file tells the LLM agent how to operate the wiki. The choice depends on which LLM tools the user works with:

| Option | File | Best for |
|--------|------|----------|
| A | `CLAUDE.md` | Claude Code |
| B | `AGENTS.md` | OpenAI Codex, OpenCode |
| C | `schema.md` | Tool-agnostic (manually referenced) |
| D | Multiple | Create both `CLAUDE.md` and `AGENTS.md` from the same content |

Record the choice as `SCHEMA_FORMAT`.

`[ACTION]` Write a config file to persist the user's choices for future vault creation:

```bash
cat > <WIKI_ROOT>/.llm-wiki-config <<EOF
WIKI_ROOT=<WIKI_ROOT>
SCHEMA_FORMAT=<SCHEMA_FORMAT>
INSTALL_PATH=<path to this INSTALL.md>
LANGUAGE=<LANGUAGE>
WIKI_LANGUAGE=<WIKI_LANGUAGE>
USE_OBSIDIAN=<true|false>
EOF
```

`[VERIFY]` Confirm the config file was written:

```bash
cat <WIKI_ROOT>/.llm-wiki-config
```

---

## Step 4: Install Claude Code Skills (Optional)

Skip this step if `SCHEMA_FORMAT` does not include `CLAUDE.md`.

`[ASK]` Ask the user if they want to install Claude Code slash command skills. Explain that this adds these global commands:

- `/wiki-init` — create a new domain vault
- `/wiki-query` — query any vault from anywhere (with vault selection)
- `/wiki-add` — add a file, folder, URL, or pasted text to any vault's `raw/`

If **no**, skip to Step 5.

`[ACTION]` Symlink skill directories from templates to the Claude Code skills directory:

```bash
ln -s <INSTALL_DIR>/templates/skills/wiki-init ~/.claude/skills/wiki-init
ln -s <INSTALL_DIR>/templates/skills/wiki-query-global ~/.claude/skills/wiki-query
ln -s <INSTALL_DIR>/templates/skills/wiki-add ~/.claude/skills/wiki-add
```

Where `<INSTALL_DIR>` is the absolute path to the directory containing this `INSTALL.md` (e.g. `~/Tools/llm-wiki`). Using symlinks means skill updates propagate automatically when templates are updated.

Note: `wiki-query-global` is the user-level query skill (vault-aware). The project-level `wiki-query` (used inside a vault directory) lives in `templates/skills/wiki-query` and is symlinked per-vault automatically during vault creation.

`[VERIFY]` Confirm skills are installed and symlinks are valid:

```bash
test -L ~/.claude/skills/wiki-init  && test -f ~/.claude/skills/wiki-init/SKILL.md  && echo "[OK] wiki-init"  || echo "[FAIL] wiki-init"
test -L ~/.claude/skills/wiki-query && test -f ~/.claude/skills/wiki-query/SKILL.md && echo "[OK] wiki-query" || echo "[FAIL] wiki-query"
test -L ~/.claude/skills/wiki-add   && test -f ~/.claude/skills/wiki-add/SKILL.md   && echo "[OK] wiki-add"   || echo "[FAIL] wiki-add"
```

After installation:
- `/wiki-init <domain-name>` — create a new vault
- `/wiki-query <vault> <question>` or `/wiki-query ?` — query from anywhere
- `/wiki-add <vault> <source>` — add sources from anywhere

---

## Step 5: Final Verification

`[VERIFY]` Run a final check to confirm the installation:

```bash
WIKI_ROOT="<WIKI_ROOT>"
PASS=true

echo "=== LLM Wiki Installation Verification ==="

# Check wiki root
if [ -d "$WIKI_ROOT" ] && [ -w "$WIKI_ROOT" ]; then
  echo "[OK] Wiki root: $WIKI_ROOT"
else
  echo "[FAIL] Wiki root missing or not writable"
  PASS=false
fi

# Check config
if [ -f "$WIKI_ROOT/.llm-wiki-config" ]; then
  echo "[OK] Config file"
else
  echo "[FAIL] Config file missing"
  PASS=false
fi

# Check Claude Code global skills (only if applicable)
for skill in wiki-init wiki-query wiki-add; do
  if [ -L ~/.claude/skills/$skill ] && [ -f ~/.claude/skills/$skill/SKILL.md ]; then
    echo "[OK] Global skill: $skill"
  fi
done

if $PASS; then
  echo ""
  echo "Installation complete."
  echo "Create your first vault with: /wiki-init <domain-name>"
  echo "Or tell your LLM agent: \"Create a new wiki vault for <domain-name>\""
else
  echo ""
  echo "Some checks failed. Review the output above."
fi
```

---

## Summary of User Choices

After installation, record these values for reference (the agent should print this):

| Setting | Value |
|---------|-------|
| Language | `<LANGUAGE>` |
| Wiki language | `<WIKI_LANGUAGE>` |
| Wiki root | `<WIKI_ROOT>` |
| Schema format | `<SCHEMA_FORMAT>` |
| Use Obsidian | `<USE_OBSIDIAN>` |

---

---

# Vault Creation Guide

The steps below are used when creating a new domain vault. They are invoked by:

- **Claude Code:** `/wiki-init <domain-name>`
- **Other LLM agents:** "Create a new wiki vault for `<domain-name>`"
- **First-time setup:** After completing the installation steps above

The agent should read `<WIKI_ROOT>/.llm-wiki-config` to get `WIKI_ROOT`, `SCHEMA_FORMAT`, `INSTALL_PATH`, `LANGUAGE`, and `USE_OBSIDIAN` before proceeding.

---

## Vault Step 1: Choose Domain Name

`[ASK]` If no domain name was provided, ask the user. Provide examples:

- `personal` — goals, health, psychology, self-improvement
- `research` — deep dives on specific topics
- `reading` — books, articles, papers
- `projects` — technical projects, side projects
- `work` — professional/business knowledge

The user may choose any name. Record this as `DOMAIN_NAME`.

---

## Vault Step 2: Scaffold Vault

`[ACTION]` Create the vault directory structure:

```
<WIKI_ROOT>/<DOMAIN_NAME>/
├── raw/                    # Immutable source documents
│   └── assets/             # Images and attachments
├── wiki/                   # LLM-maintained wiki pages
│   ├── index.md            # Content catalog
│   └── log.md              # Chronological activity log
├── <SCHEMA_FILE>           # Schema file
├── .gitignore              # Git ignore rules
├── README.md               # Vault usage guide (bilingual EN/ZH)
├── .claude/                # (Only if SCHEMA_FORMAT includes CLAUDE.md)
│   └── skills/             # Project-level skills
│       ├── wiki-ingest/    # /wiki-ingest — process new sources
│       ├── wiki-query/     # /wiki-query — ask questions
│       ├── wiki-lint/      # /wiki-lint — health check
│       └── wiki-codemap/   # /wiki-codemap — generate codebase notes
└── .obsidian/              # (Only if USE_OBSIDIAN=true)
    ├── app.json            # Basic Obsidian settings
    └── graph.json          # Graph view (exclude raw/ from graph)
```

Commands:

```bash
VAULT="<WIKI_ROOT>/<DOMAIN_NAME>"
mkdir -p "$VAULT"/{raw/assets,wiki}
# Only create .obsidian/ if USE_OBSIDIAN=true
if [ "<USE_OBSIDIAN>" = "true" ]; then
  mkdir -p "$VAULT/.obsidian"
fi
```

Write the following files using the templates in `templates/`:

1. `wiki/index.md` — from `templates/index.md`
2. `wiki/log.md` — from `templates/log.md`
3. Schema file — from `templates/schema.md`, written to the chosen filename(s)
4. `.gitignore` — from `templates/gitignore`
5. `README.md` — from `templates/README.md`
6. `.obsidian/app.json` — from `templates/obsidian-app.json` **(only if USE_OBSIDIAN=true)**
7. `.obsidian/graph.json` — from `templates/graph.json` **(only if USE_OBSIDIAN=true)**

If `SCHEMA_FORMAT` includes `CLAUDE.md`, also symlink project-level skills:

```bash
mkdir -p "$VAULT/.claude/skills"
for skill in wiki-ingest wiki-query wiki-lint wiki-codemap; do
  ln -s <INSTALL_DIR>/templates/skills/$skill "$VAULT/.claude/skills/$skill"
done
```

Where `<INSTALL_DIR>` is the absolute path to the directory containing this `INSTALL.md`.

Replace `{{DOMAIN_NAME}}` with the actual domain name in all templates.
Replace `{{WIKI_ROOT}}` with the actual wiki root path in the schema file and README.
Replace `{{DATE}}` with today's date (YYYY-MM-DD) in the log file.

`[VERIFY]` Confirm vault structure:

```bash
VAULT="<WIKI_ROOT>/<DOMAIN_NAME>"
echo "=== Directory structure ===" && \
find "$VAULT" -type f | sort && \
echo "=== Schema file ===" && \
test -f "$VAULT/<SCHEMA_FILE>" && echo "OK" || echo "MISSING" && \
echo "=== Index ===" && \
test -f "$VAULT/wiki/index.md" && echo "OK" || echo "MISSING" && \
echo "=== Log ===" && \
test -f "$VAULT/wiki/log.md" && echo "OK" || echo "MISSING" && \
echo "=== Gitignore ===" && \
test -f "$VAULT/.gitignore" && echo "OK" || echo "MISSING" && \
echo "=== README ===" && \
test -f "$VAULT/README.md" && echo "OK" || echo "MISSING"
# Only if SCHEMA_FORMAT includes CLAUDE.md:
for skill in wiki-ingest wiki-query wiki-lint wiki-codemap; do
  if [ -L "$VAULT/.claude/skills/$skill" ] && [ -f "$VAULT/.claude/skills/$skill/SKILL.md" ]; then
    echo "=== Skill: $skill ===" && echo "OK"
  fi
done
# Only if USE_OBSIDIAN=true:
if [ "<USE_OBSIDIAN>" = "true" ]; then
  echo "=== Obsidian app.json ===" && \
  test -f "$VAULT/.obsidian/app.json" && echo "OK" || echo "MISSING" && \
  echo "=== Obsidian graph.json ===" && \
  test -f "$VAULT/.obsidian/graph.json" && echo "OK" || echo "MISSING"
fi
```

---

## Vault Step 3: Initialize Git Repository (New Vaults Only)

Skip this step when adopting an existing vault that already has a `.git/` directory.

`[ACTION]` Initialize a git repo and make the first commit:

```bash
cd "<WIKI_ROOT>/<DOMAIN_NAME>"
git init
git add -A
git commit -m "Initial vault setup for <DOMAIN_NAME>"
```

`[VERIFY]` Confirm git is working:

```bash
cd "<WIKI_ROOT>/<DOMAIN_NAME>" && git log --oneline -1
```

Expected: one commit with the initial setup message.

---

## Vault Step 4: Open in Obsidian (Optional)

Skip this step if `USE_OBSIDIAN=false`.

`[ASK]` Ask the user if they want to open the vault in Obsidian now.

If **yes**, instruct them to:
1. Open Obsidian
2. Click "Open folder as vault"
3. Select `<WIKI_ROOT>/<DOMAIN_NAME>`

### Recommended Settings

- **Settings → Files and links → Attachment folder path**: should already be set to `raw/assets` via `app.json`.

### Recommended Hotkeys

- **Settings → Hotkeys** → search "Download attachments for current file" → bind to a hotkey (e.g. `Ctrl+Shift+D`). This downloads all images in a clipped article to local disk so the LLM can view them directly.

### Recommended Community Plugins (Optional)

- **Dataview** — runs queries over page frontmatter. Useful if wiki pages have YAML frontmatter with tags, dates, etc.
- **Marp Slides** — renders Marp-format markdown as slide decks. Useful for generating presentations from wiki content.

### Obsidian Web Clipper (Optional)

`[ASK]` Ask the user if they want to install the Obsidian Web Clipper browser extension. Explain that it is a **separate browser extension** (not an Obsidian plugin) that converts web articles to markdown, useful for quickly getting sources into `raw/`.

If **yes**, instruct them to install it from their browser's extension store or from https://obsidian.md/clipper.

**Multiple vaults:** When clipping, Web Clipper shows a vault selector dropdown. To clip into a specific domain vault:

1. Each domain vault must have been opened in Obsidian at least once ("Open folder as vault").
2. When clipping, select the target vault from the dropdown.
3. Set the folder path to `raw/` so clipped articles land in the right place.

### Tips

- **Graph view** (`Ctrl/Cmd+G`) — visualize the wiki structure, see which pages are hubs, which are orphans, and how topics connect.

---

## Vault Step 5: Final Verification

`[VERIFY]` Run a final check to confirm the vault is ready:

```bash
VAULT="<WIKI_ROOT>/<DOMAIN_NAME>"
SCHEMA="<SCHEMA_FILE>"
PASS=true

echo "=== Vault Verification: <DOMAIN_NAME> ==="

# Check directory structure
for dir in raw raw/assets wiki; do
  if [ -d "$VAULT/$dir" ]; then
    echo "[OK] $dir/"
  else
    echo "[FAIL] $dir/ missing"
    PASS=false
  fi
done

# Check Obsidian directory (only if USE_OBSIDIAN=true)
if [ "<USE_OBSIDIAN>" = "true" ]; then
  if [ -d "$VAULT/.obsidian" ]; then
    echo "[OK] .obsidian/"
  else
    echo "[FAIL] .obsidian/ missing"
    PASS=false
  fi
fi

# Check files
for file in wiki/index.md wiki/log.md "$SCHEMA" .gitignore README.md; do
  if [ -f "$VAULT/$file" ]; then
    echo "[OK] $file"
  else
    echo "[FAIL] $file missing"
    PASS=false
  fi
done

# Check Obsidian config (only if USE_OBSIDIAN=true)
if [ "<USE_OBSIDIAN>" = "true" ]; then
  for ofile in .obsidian/app.json .obsidian/graph.json; do
    if [ -f "$VAULT/$ofile" ]; then
      echo "[OK] $ofile"
    else
      echo "[FAIL] $ofile missing"
      PASS=false
    fi
  done
fi

# Check git
if git -C "$VAULT" rev-parse --is-inside-work-tree > /dev/null 2>&1; then
  echo "[OK] git initialized"
else
  echo "[FAIL] git not initialized"
  PASS=false
fi

# Check Claude Code project skills (only if applicable)
for skill in wiki-ingest wiki-query wiki-lint wiki-codemap; do
  if [ -L "$VAULT/.claude/skills/$skill" ] && [ -f "$VAULT/.claude/skills/$skill/SKILL.md" ]; then
    echo "[OK] project skill: $skill"
  fi
done

if $PASS; then
  echo ""
  echo "Vault ready at: $VAULT"
else
  echo ""
  echo "Some checks failed. Review the output above."
fi
```

---

---

# Vault Adoption Guide

The steps below are used when adopting an **existing directory** as an LLM Wiki vault. This is for directories that already contain content (e.g. markdown notes, Obsidian vaults, raw documents) and the user wants to bring them under LLM Wiki management without losing existing data.

This guide is invoked when `/wiki-init <name>` detects that `<WIKI_ROOT>/<name>` already exists.

The agent should read `<WIKI_ROOT>/.llm-wiki-config` to get `WIKI_ROOT`, `SCHEMA_FORMAT`, `INSTALL_PATH`, `LANGUAGE`, and `USE_OBSIDIAN` before proceeding.

---

## Adopt Step 1: Inventory Existing Content

`[ACTION]` Scan the existing directory and report what's already present:

```bash
VAULT="<WIKI_ROOT>/<DOMAIN_NAME>"

echo "=== Existing Structure ==="
echo "--- Directories ---"
find "$VAULT" -maxdepth 2 -type d | sort
echo "--- Files (top 2 levels) ---"
find "$VAULT" -maxdepth 2 -type f | sort
echo "--- File count ---"
find "$VAULT" -type f | wc -l
```

`[ASK]` Show the user the inventory and ask:

1. **Which existing files should go into `raw/`?** (e.g. "everything in `notes/`", "all the `.md` files in the root", "leave them where they are")
2. **Are there any existing wiki-style pages that should go into `wiki/`?** (e.g. index files, summaries the user has already written)
3. **Should any files be left in place as-is?** (e.g. existing `.obsidian/` config, other tool configs)

---

## Adopt Step 2: Create Missing Structure

`[ACTION]` Create only the directories and files that don't already exist. Never overwrite existing files.

```bash
VAULT="<WIKI_ROOT>/<DOMAIN_NAME>"

# Create directories only if missing
[ -d "$VAULT/raw" ]        || mkdir -p "$VAULT/raw"
[ -d "$VAULT/raw/assets" ] || mkdir -p "$VAULT/raw/assets"
[ -d "$VAULT/wiki" ]       || mkdir -p "$VAULT/wiki"
```

For each of these files, **only create if missing**:

| File | Template | Action if exists |
|------|----------|-----------------|
| `wiki/index.md` | `templates/index.md` | Skip — user may have existing content |
| `wiki/log.md` | `templates/log.md` | Skip — append adoption entry later |
| Schema file | `templates/schema.md` | Skip — user may have existing CLAUDE.md/AGENTS.md |
| `.gitignore` | `templates/gitignore` | Merge — append missing entries without duplicating |
| `README.md` | `templates/README.md` | Skip — user may have existing README |

If `USE_OBSIDIAN=true`:

```bash
mkdir -p "$VAULT/.obsidian"
# Only write config files if missing
[ -f "$VAULT/.obsidian/app.json" ]   || cp templates/obsidian-app.json "$VAULT/.obsidian/app.json"
[ -f "$VAULT/.obsidian/graph.json" ] || cp templates/graph.json "$VAULT/.obsidian/graph.json"
```

If `SCHEMA_FORMAT` includes `CLAUDE.md`, create skill symlinks (only if missing):

```bash
mkdir -p "$VAULT/.claude/skills"
for skill in wiki-ingest wiki-query wiki-lint wiki-codemap; do
  [ -L "$VAULT/.claude/skills/$skill" ] || ln -s <INSTALL_DIR>/templates/skills/$skill "$VAULT/.claude/skills/$skill"
done
```

Replace `{{DOMAIN_NAME}}`, `{{WIKI_ROOT}}`, and `{{DATE}}` in any newly created template files.

---

## Adopt Step 3: Relocate Existing Files (If Requested)

`[ACTION]` Based on the user's answers in Adopt Step 1, move files to their new locations. Use `mv` (not `cp`) to avoid duplication:

```bash
# Example: move user-specified files into raw/
# mv "$VAULT/<source-path>" "$VAULT/raw/"

# Example: move user-specified wiki pages into wiki/
# mv "$VAULT/<page-path>" "$VAULT/wiki/"
```

**Rules:**
- Always confirm moves with the user before executing.
- Never delete files — only relocate.
- If `wiki/index.md` already exists and the user has pages to catalogue, offer to update the existing index rather than replacing it.

---

## Adopt Step 4: Bootstrap Index and Log

`[ACTION]` If `wiki/index.md` was newly created (empty template), scan existing content in `raw/` and `wiki/` and populate the index with what's already there.

`[ACTION]` If `wiki/log.md` was newly created or already existed, append an adoption entry:

```markdown
## [{{DATE}}] adopt | Existing directory adopted as LLM Wiki vault

Adopted existing directory with N files. Structure created: raw/, wiki/, schema.
```

If `wiki/log.md` already existed, append to it. If newly created, the template already has an init entry — replace `init | Vault created` with the adoption entry above.

---

## Adopt Step 5: Initialize Git (If Needed)

`[ACTION]` Only initialize git if there is no existing `.git/` directory:

```bash
VAULT="<WIKI_ROOT>/<DOMAIN_NAME>"
if [ ! -d "$VAULT/.git" ]; then
  cd "$VAULT"
  git init
  git add -A
  git commit -m "Adopt existing directory as LLM Wiki vault: <DOMAIN_NAME>"
else
  echo "Git already initialized — skipping."
  # Stage and commit the new LLM Wiki files
  cd "$VAULT"
  git add -A
  git status
fi
```

`[ASK]` If git was already initialized, show `git status` and ask the user if they want to commit the new LLM Wiki scaffolding files.

---

## Adopt Step 6: Final Verification

`[VERIFY]` Run the same verification as Vault Step 5 (from the Vault Creation Guide above). The checks are identical — both new and adopted vaults should pass the same criteria.
