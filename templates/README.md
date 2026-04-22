# {{DOMAIN_NAME}} Wiki

> This vault is powered by [LLM Wiki](https://github.com/cdc-internal/llm-wiki) — a framework for building personal knowledge bases maintained by LLM agents.
>
> 此 Vault 由 [LLM Wiki](https://github.com/cdc-internal/llm-wiki) 驅動 — 一個由 LLM 代理維護的個人知識庫框架。

---

## Getting Started / 開始使用

### Prerequisites / 前置需求

1. Install [Claude Code](https://claude.ai/code).
   安裝 [Claude Code](https://claude.ai/code)。

2. Install [LLM Wiki](https://github.com/cdc-internal/llm-wiki) — follow the `INSTALL.md` in the main repo to set up wiki root, config, and global skills.
   安裝 [LLM Wiki](https://github.com/cdc-internal/llm-wiki) — 依照主倉庫中的 `INSTALL.md` 設定 wiki root、config 及全域 skill。

### First Time Setup / 首次設定

After cloning or receiving this vault, run the following command **once** in Claude Code to set up local skills, Obsidian config, and other machine-specific files:
Clone 或取得此 vault 後，在 Claude Code 中執行以下命令**一次**，以建立本機的 skill symlink、Obsidian 設定等：

```
/wiki-init {{DOMAIN_NAME}}
```

This detects the existing vault and adopts it — creating any missing structure without overwriting your data.
此命令會偵測已存在的 vault 並進行採用 — 只補建缺少的結構，不會覆蓋現有資料。

### Adding Sources / 新增來源

Drop files (articles, notes, PDFs, etc.) into the `raw/` folder, then ask the agent to process them:
將檔案（文章、筆記、PDF 等）放入 `raw/` 資料夾，然後請代理處理：

```
/wiki-ingest
```

### Asking Questions / 提問

Ask questions about your knowledge base:
對知識庫提問：

```
/wiki-query what are the key themes across all sources?
```

### Health Check / 健康檢查

Run a lint pass to find gaps, contradictions, and orphan pages:
執行檢查以找出缺漏、矛盾和孤立頁面：

```
/wiki-lint
```

---

## Project Skills / 專案技能

These slash commands are available when Claude Code is running inside this vault:
在此 vault 中執行 Claude Code 時，可使用以下斜線命令：

| Command / 命令 | Description / 說明 |
|---|---|
| `/wiki-ingest` | Process new source files in `raw/` and update the wiki. 處理 `raw/` 中的新來源檔案並更新 wiki。 |
| `/wiki-query <question>` | Ask a question — the agent searches the wiki and synthesizes an answer. 提問 — 代理會搜尋 wiki 並綜合回答。 |
| `/wiki-lint` | Run a health check on the wiki: find contradictions, orphans, and gaps. 對 wiki 進行健康檢查：找出矛盾、孤立頁面和缺漏。 |
| `/wiki-codemap` | Generate a codemap of an external codebase and save it as a wiki source. 產生外部程式碼庫的 codemap 並儲存為 wiki 來源。 |

### Global Skill / 全域技能

To create a new vault from anywhere in Claude Code:
在 Claude Code 任何位置建立新的 vault：

```
/wiki-init {{DOMAIN_NAME}}
```

---

## Directory Structure / 目錄結構

```
raw/            # Immutable source documents (never modify) / 不可變的來源文件（勿修改）
  assets/       # Images and attachments / 圖片與附件
wiki/           # LLM-maintained pages / LLM 維護的頁面
  index.md      # Content catalog / 內容目錄
  log.md        # Activity log / 活動日誌
```

---

## Links / 連結

- **Main repo / 主倉庫:** [cdc-internal/llm-wiki](https://github.com/cdc-internal/llm-wiki)
- **Installation guide / 安裝指南:** See `INSTALL.md` in the main repo.
