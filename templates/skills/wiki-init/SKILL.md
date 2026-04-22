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
4. Ask the user to describe the **Domain Context** for the schema:
   - **Topic description** — What is this vault about? (1-2 sentences)
   - **Goals** — What do you want to achieve? (1-3 bullet points)
   - **Key questions** — What questions are you trying to answer? (1-5 bullet points)
   If the user wants to skip, use defaults from the installation guide.
5. Read the installation guide at `INSTALL_PATH`.
6. Check if `<WIKI_ROOT>/<VAULT_NAME>` already exists.
   - **If it does NOT exist:** Follow the **Vault Creation Guide** section (Vault Steps 1–5) to create a new vault from scratch.
   - **If it DOES exist:** Follow the **Vault Adoption Guide** section to adopt the existing directory as a vault.
