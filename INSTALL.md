# LLM Wiki — Installation Guide

This is an interactive installation playbook. An LLM agent (Claude Code, Codex, OpenCode, etc.) follows these steps with the user to set up LLM Wiki on a new machine.

**For the agent:** Follow the steps below in order. Each step marked with `[ASK]` requires user input before proceeding. Each step marked with `[VERIFY]` requires checking the result before moving on. Do not skip verification steps.

---

## Step 0: Language Preference

`[ASK]` Ask the user which language they prefer for this installation process:

- **English**
- **中文 (Chinese)**

Use the chosen language for all prompts, confirmations, and output messages for the remainder of the installation. The files created (schema, index, log, etc.) are always written in **English** regardless of this choice.

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

`[ASK]` Ask the user if they want to install a Claude Code slash command skill. Explain that this adds a global command:

- `/llm-wiki-new-vault` — quickly create a new domain vault

If **no**, skip to Step 5.

`[ACTION]` Symlink skill directories from templates to the Claude Code skills directory:

```bash
ln -s <INSTALL_DIR>/templates/skills/llm-wiki-new-vault ~/.claude/skills/llm-wiki-new-vault
```

Where `<INSTALL_DIR>` is the absolute path to the directory containing this `INSTALL.md` (e.g. `~/Tools/llm-wiki`). Using symlinks means skill updates propagate automatically when templates are updated.

`[VERIFY]` Confirm skills are installed and symlinks are valid:

```bash
test -L ~/.claude/skills/llm-wiki-new-vault && test -f ~/.claude/skills/llm-wiki-new-vault/SKILL.md && echo "[OK] llm-wiki-new-vault" || echo "[FAIL] llm-wiki-new-vault"
```

After installation, the user can type `/llm-wiki-new-vault` (or `/llm-wiki-new-vault research`) in Claude Code to create new vaults.

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

# Check Claude Code global skill (only if applicable)
if [ -L ~/.claude/skills/llm-wiki-new-vault ] && [ -f ~/.claude/skills/llm-wiki-new-vault/SKILL.md ]; then
  echo "[OK] Global skill: llm-wiki-new-vault"
fi

if $PASS; then
  echo ""
  echo "Installation complete."
  echo "Create your first vault with: /llm-wiki-new-vault <domain-name>"
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
| Wiki root | `<WIKI_ROOT>` |
| Schema format | `<SCHEMA_FORMAT>` |
| Use Obsidian | `<USE_OBSIDIAN>` |

---

---

# Vault Creation Guide

The steps below are used when creating a new domain vault. They are invoked by:

- **Claude Code:** `/llm-wiki-new-vault <domain-name>`
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
├── .claude/                # (Only if SCHEMA_FORMAT includes CLAUDE.md)
│   └── skills/             # Project-level skills
│       ├── ingest/         # /ingest — process new sources
│       ├── query/          # /query — ask questions
│       └── lint/           # /lint — health check
└── .obsidian/              # (Only if USE_OBSIDIAN=true)
    └── app.json            # Basic Obsidian settings
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
5. `.obsidian/app.json` — from `templates/obsidian-app.json` **(only if USE_OBSIDIAN=true)**

If `SCHEMA_FORMAT` includes `CLAUDE.md`, also symlink project-level skills:

```bash
mkdir -p "$VAULT/.claude/skills"
for skill in ingest query lint; do
  ln -s <INSTALL_DIR>/templates/skills/$skill "$VAULT/.claude/skills/$skill"
done
```

Where `<INSTALL_DIR>` is the absolute path to the directory containing this `INSTALL.md`.

Replace `{{DOMAIN_NAME}}` with the actual domain name in all templates.
Replace `{{WIKI_ROOT}}` with the actual wiki root path in the schema file.
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
test -f "$VAULT/.gitignore" && echo "OK" || echo "MISSING"
# Only if SCHEMA_FORMAT includes CLAUDE.md:
for skill in ingest query lint; do
  if [ -L "$VAULT/.claude/skills/$skill" ] && [ -f "$VAULT/.claude/skills/$skill/SKILL.md" ]; then
    echo "=== Skill: $skill ===" && echo "OK"
  fi
done
# Only if USE_OBSIDIAN=true:
if [ "<USE_OBSIDIAN>" = "true" ]; then
  echo "=== Obsidian config ===" && \
  test -f "$VAULT/.obsidian/app.json" && echo "OK" || echo "MISSING"
fi
```

---

## Vault Step 3: Initialize Git Repository

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
for file in wiki/index.md wiki/log.md "$SCHEMA" .gitignore; do
  if [ -f "$VAULT/$file" ]; then
    echo "[OK] $file"
  else
    echo "[FAIL] $file missing"
    PASS=false
  fi
done

# Check Obsidian config (only if USE_OBSIDIAN=true)
if [ "<USE_OBSIDIAN>" = "true" ]; then
  if [ -f "$VAULT/.obsidian/app.json" ]; then
    echo "[OK] .obsidian/app.json"
  else
    echo "[FAIL] .obsidian/app.json missing"
    PASS=false
  fi
fi

# Check git
if git -C "$VAULT" rev-parse --is-inside-work-tree > /dev/null 2>&1; then
  echo "[OK] git initialized"
else
  echo "[FAIL] git not initialized"
  PASS=false
fi

# Check Claude Code project skills (only if applicable)
for skill in ingest query lint; do
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
