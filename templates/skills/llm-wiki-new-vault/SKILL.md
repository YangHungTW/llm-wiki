---
name: llm-wiki-new-vault
description: Create a new LLM Wiki domain vault. Accepts an optional vault name as argument.
disable-model-invocation: true
allowed-tools: Bash Read Write
---

# LLM Wiki — Create New Vault

Create a new domain vault. Vault name from argument: $ARGUMENTS

1. Find the wiki root by reading `~/.llm-wiki-config`. If not found, search common locations (`~/Wiki/.llm-wiki-config`, etc.) or ask the user.
2. Read the config to get `WIKI_ROOT`, `SCHEMA_FORMAT`, `INSTALL_PATH`, and `USE_OBSIDIAN`.
3. If no vault name was provided in the argument, ask the user what domain/topic the new vault should be. Provide examples: `personal`, `research`, `reading`, `projects`, `work`.
4. Read the installation guide at `INSTALL_PATH`.
5. Follow the **Vault Creation Guide** section (Vault Steps 1–5) using the saved config values and the templates from the `templates/` directory next to `INSTALL_PATH`.
