---
name: wiki-codemap
description: Read a codebase or file and generate understanding notes saved to raw/. Accepts a path as argument.
disable-model-invocation: true
allowed-tools: Bash Read Write Edit Glob Grep
---

# Codemap

Generate understanding notes from code. Path from argument: $ARGUMENTS

**Important:** Read `WIKI_LANGUAGE` from the vault's config (via `<WIKI_ROOT>/.llm-wiki-config`). Notes must be written in this language. Frontmatter keys and filenames remain in English.

## Steps

1. If no path was provided, ask the user for the path to a codebase directory or a specific file.
2. Determine the scope:
   - **Directory**: explore the codebase structure, key files, dependencies, architecture.
   - **File**: read the file and analyze its purpose, logic, and relationships.
3. For a directory, explore systematically:
   - Read project config files (package.json, Cargo.toml, go.mod, pyproject.toml, etc.)
   - Identify the directory structure and key modules
   - Read important files (entry points, core logic, config, types/interfaces)
   - Understand the architecture, data flow, and design decisions
4. Generate a markdown document covering:
   - **Overview** — what the project/file does, in plain language
   - **Architecture** — how it's structured, key components and their roles
   - **Key files** — what each important file does
   - **Dependencies** — external libraries and why they're used
   - **Data flow** — how data moves through the system
   - **Design decisions** — notable patterns, trade-offs, or conventions
   - For a single file: focus on purpose, logic, inputs/outputs, and how it fits in the larger system
5. Save the document to `raw/` with datetime prefix:
   ```
   raw/YYYY-MM-DD-<project-or-file-name>-codemap.md
   ```
6. Ask the user if they want to ingest the codemap into the wiki now (run `/wiki-ingest` on the saved file).
7. Ask the user if they want to commit the changes. If yes, run: `git add -A && git commit -m "[codemap] <project-or-file-name>"`
