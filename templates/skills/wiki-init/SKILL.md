---
name: wiki-init
description: Create a new LLM Wiki domain vault or adopt an existing directory as a vault. Accepts an optional vault name as argument.
disable-model-invocation: true
allowed-tools: Bash Read Write
---

# LLM Wiki — Create or Adopt Vault

Create a new domain vault, or adopt an existing directory as a vault. Vault name from argument: $ARGUMENTS

1. Find the wiki root by reading `~/.llm-wiki-config`. If not found, search common locations (`~/Wiki/.llm-wiki-config`, etc.) or ask the user.
2. Read the config to get `WIKI_ROOT`, `SCHEMA_FORMAT`, `INSTALL_PATH`, and `USE_OBSIDIAN`.
3. If no vault name was provided in the argument, ask the user what domain/topic the new vault should be. Provide examples: `personal`, `research`, `reading`, `projects`, `work`.
4. Read the installation guide at `INSTALL_PATH`.
5. Check if `<WIKI_ROOT>/<VAULT_NAME>` already exists.
   - **If it does NOT exist:** Follow the **Vault Creation Guide** section (Vault Steps 1–5) to create a new vault from scratch.
   - **If it DOES exist:** Follow the **Vault Adoption Guide** section to adopt the existing directory as a vault.
