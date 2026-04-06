# LLM Wiki

A pattern for building personal knowledge bases maintained by LLM agents.

## What is this?

Instead of retrieving from raw documents at query time (like RAG), the LLM **incrementally builds and maintains a persistent wiki** — a structured, interlinked collection of markdown files. When you add a new source, the LLM reads it, extracts key information, and integrates it into existing wiki pages. The knowledge compounds over time rather than being re-derived on every query.

You never write the wiki yourself. The LLM does all the summarizing, cross-referencing, and bookkeeping. You curate sources, ask questions, and direct the analysis.

## Architecture

Each wiki vault has three layers:

```
vault/
├── raw/              # Your source documents (immutable)
│   └── assets/       # Images and attachments
├── wiki/             # LLM-maintained pages
│   ├── index.md      # Content catalog
│   └── log.md        # Activity log
├── CLAUDE.md         # Schema file (or AGENTS.md, schema.md)
└── .gitignore
```

- **Raw sources** — articles, papers, notes, images. The LLM reads but never modifies these.
- **The wiki** — LLM-generated markdown pages with wikilinks, frontmatter, and citations. Includes source summaries, entity pages, concept pages, analyses, and comparisons.
- **The schema** — instructions that tell the LLM how to operate the wiki (conventions, workflows, rules).

## How to use

### Setup

1. Clone this repo or copy it to your machine.
2. Open a terminal in the repo directory with your LLM agent (Claude Code, Codex, OpenCode, etc.).
3. Tell the agent: **"Follow INSTALL.md to set up LLM Wiki"**
4. The agent will walk you through an interactive setup — choosing a language, wiki location, schema format, and creating your first domain vault.

### Daily use

**Ingest a source:** Drop a file into `raw/` and tell the agent to process it. The agent reads it, discusses key takeaways with you, creates wiki pages, and updates cross-references.

**Ask a question:** Ask anything about your wiki's contents. The agent searches the index, reads relevant pages, and synthesizes an answer with citations. Good answers can be saved back as wiki pages.

**Health check:** Ask the agent to lint the wiki. It finds contradictions, orphan pages, missing cross-references, and suggests new sources to look for.

**Create a new vault:** Tell the agent to create a new domain vault at any time (e.g., "Create a new wiki vault for `reading`").

### Optional tools

- **Obsidian** — browse the wiki with graph view, wikilink navigation, and plugins (Dataview, Marp).
- **Obsidian Web Clipper** — browser extension to clip web articles directly into `raw/`.
- **NotebookLM** — use alongside the wiki for quick exploration, then bring insights back into the wiki.
- **qmd** — local markdown search engine for when the wiki grows large.

## Supported LLM agents

The schema file works with any LLM agent that reads project instructions:

| Agent | Schema file |
|-------|-------------|
| Claude Code | `CLAUDE.md` |
| OpenAI Codex | `AGENTS.md` |
| OpenCode / others | `schema.md` (tool-agnostic) |

## Credits

Inspired by [Andrej Karpathy's LLM Wiki gist](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f) and Vannevar Bush's Memex (1945).

---

# LLM Wiki

一個用 LLM 代理建立和維護個人知識庫的模式。

## 這是什麼？

不同於 RAG 在每次查詢時從原始文件中檢索，LLM 會**逐步建立並維護一個持久的 wiki** — 一個結構化、互相連結的 markdown 檔案集合。當你加入新的來源時，LLM 會讀取它、提取關鍵資訊，並整合到現有的 wiki 頁面中。知識隨時間累積，而非每次查詢都重新推導。

你永遠不需要自己寫 wiki。LLM 負責所有的摘要、交叉引用和記錄工作。你負責策劃來源、提出問題、引導分析方向。

## 架構

每個 wiki vault 有三層：

```
vault/
├── raw/              # 你的來源文件（不可變）
│   └── assets/       # 圖片和附件
├── wiki/             # LLM 維護的頁面
│   ├── index.md      # 內容目錄
│   └── log.md        # 活動日誌
├── CLAUDE.md         # Schema 檔案（或 AGENTS.md、schema.md）
└── .gitignore
```

- **Raw sources** — 文章、論文、筆記、圖片。LLM 只讀取，不修改。
- **Wiki** — LLM 產生的 markdown 頁面，包含 wikilinks、frontmatter 和引用。包括來源摘要、實體頁面、概念頁面、分析和比較。
- **Schema** — 告訴 LLM 如何操作 wiki 的指令（慣例、工作流程、規則）。

## 使用方式

### 安裝

1. Clone 這個 repo 或複製到你的電腦。
2. 用你的 LLM 代理（Claude Code、Codex、OpenCode 等）在 repo 目錄開啟終端機。
3. 告訴代理：**「照 INSTALL.md 來安裝 LLM Wiki」**
4. 代理會引導你完成互動式設定 — 選擇語言、wiki 位置、schema 格式，並建立第一個 domain vault。

### 日常使用

**匯入來源：** 把檔案放到 `raw/`，告訴代理處理它。代理會讀取、跟你討論重點、建立 wiki 頁面、更新交叉引用。

**提問：** 問任何關於 wiki 內容的問題。代理搜尋索引、讀取相關頁面、合成帶引用的答案。好的答案可以存回 wiki。

**健康檢查：** 請代理 lint wiki。它會找出矛盾、孤立頁面、缺少的交叉引用，並建議新的來源。

**建立新 vault：** 隨時告訴代理建立新的 domain vault（例如：「建立一個 `reading` 的 wiki vault」）。

### 選用工具

- **Obsidian** — 用圖譜視圖、wikilink 導航和外掛（Dataview、Marp）瀏覽 wiki。
- **Obsidian Web Clipper** — 瀏覽器擴充套件，直接把網頁文章擷取到 `raw/`。
- **NotebookLM** — 搭配 wiki 做快速探索，再把洞察帶回 wiki。
- **qmd** — wiki 變大後可用的本地 markdown 搜尋引擎。

## 支援的 LLM 代理

Schema 檔案適用於任何會讀取專案指令的 LLM 代理：

| 代理 | Schema 檔案 |
|------|-------------|
| Claude Code | `CLAUDE.md` |
| OpenAI Codex | `AGENTS.md` |
| OpenCode / 其他 | `schema.md`（工具無關） |

## 致謝

靈感來自 [Andrej Karpathy 的 LLM Wiki gist](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f) 和 Vannevar Bush 的 Memex（1945）。
