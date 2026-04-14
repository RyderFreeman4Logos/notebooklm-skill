---
name: notebooklm
description: 高级 NotebookLM 数据处理与上传流水线 (万物转 Markdown + Tokuin 智能切分)。解决 NotebookLM 对超长文档的静默截断和 50 个源上限问题。
allowed-tools: Bash, read_file
---

# NotebookLM 数据集智能处理与上传流水线

本 Skill 旨在解决将超大规模/无结构数据集（如长篇中文小说、大型代码库、视频字幕集等）无损、高效地喂给 NotebookLM 的核心痛点。

传统直接上传 TXT/PDF 会面临大模型底层的**静默截断（Token 溢出）**和**检索迷失（Chunking 混乱）**问题。本流水线采用 **“万物转 Markdown -> Tokuin 精准算 Token -> 针对性语义切分 -> 跨笔记本组装”** 的架构。

## 安装与配置 (Installation & Setup)

*   **依赖自动安装 (Auto-Install):** 执行流水线过程中，若缺失任何 CLI 工具（如 `pdftk`、`qpdf`、`tokuin`、`nlm`、`pandoc` 等），请优先尝试使用 `mise use -g <plugin>:<tool>` (或相应的 `cargo install`, `pipx` 等经由 mise 的方式) 自动安装，确保流水线不中断。
*   **NLM CLI 安装:** 通过 mise 和 pipx 全局安装 `notebooklm-mcp-cli`:
    ```bash
    mise use -g pipx:notebooklm-mcp-cli
    ```
*   **NLM 认证:** 安装后必须进行登录配置才能使用：
    ```bash
    nlm login              # 自动模式（需 Chrome）
    # 或
    nlm login --manual --file cookies.txt  # 手动模式
    nlm login --check      # 验证登录状态
    ```
*   **Tokuin 安装:** 用于精准计算 Token，由于暂未发布至 crates.io，需通过源码安装（必须开启 gemini feature）：
    ```bash
    mise use -g cargo:https://github.com/nooscraft/tokuin
    ```

## 核心流水线 (The Pipeline)

当你接到为 NotebookLM 准备资料的任务时，请**严格按照以下四个步骤执行**：

### 1. 文件类型判断与策略路由 (Strategy Routing)
首先判断用户提供的资料类型。**不要将所有的切分策略硬编码在记忆里，请按需读取对应的策略说明文件**。
根据资料类型，读取 `strategies/` 目录下对应的文件获取详细操作指南：

*   **PDF 书籍/文档:** 读取 `strategies/pdf-documents.md` (尽量避免破坏性转 MD，可能保留物理切分)
*   **中文长篇小说 (TXT/EPUB):** 读取 `strategies/cn-novel.md`
*   **日文轻小说/外文翻译:** 读取 `strategies/jp-light-novel.md`
*   **代码库/文档项目:** 读取 `strategies/codebase.md`
*   **视频字幕集 (SRT/VTT):** 读取 `strategies/video-subs.md`

### 2. 格式转换与清洗 (Markdown-First)
*   **非 PDF 源：** 无论是 TXT 还是 EPUB，一律想办法转换为 **Markdown (.md)**。
*   利用 Markdown 的 Header（`#`, `##` 等）来建立语义边界，这能极大帮助 NotebookLM 建立高质量的 Chunking。
*   清理掉无意义的空行和乱码。

### 3. Tokuin 智能计价与安全切分 (Token-Aware Chunking)
NotebookLM 的单源 Token 极限大约对应 50万“字”（底层 100万 Token 的上下文窗口）。为了绝对安全，防止模型静默截断：
*   **强制前置检查:** 无论来源是 TXT、EPUB 还是 PDF，在上传前**必须**提取其纯文本，并使用 `tokuin` 进行严格的 Token 审查。特别是 PDF，绝不能想当然认为页数少就不超标，**必须**过一遍计价器，绝对避免超过字数限制。
*   **核心工具 `tokuin`:** 如果系统未安装，由于该工具暂未发布至 crates.io，请通过源码安装：`mise exec -- cargo install --git https://github.com/nooscraft/tokuin`
*   **安全阈值:** 使用 `tokuin <file> --model gemini-1.5-pro` 计算 Token 数。将单个 Markdown 源文件强制控制在 **70万 - 80万 Token** 之下（宁可没触碰上限导致浪费，也要将截断风险降到可以忽略）。
*   **切分原则:** 严格按照步骤 1 中读取到的策略进行切分，**绝不能从段落或句子中间切断**。切分后的文件要有序命名（例如 `01_卷一_第1-10章.md`, `02_卷一_第11-20章.md`），以保留时间线元数据。

### 4. 突破上限：跨笔记本组装 (Notebook Spanning)
NotebookLM 每个笔记本有 **50 个源 (Sources)** 的硬性物理上限。
*   如果拆分后的 `.md` 或 `.pdf` 文件总数超过 48 个，**必须跨多个 Notebook 上传**。
*   按卷或逻辑块归类（例如 `修仙传_Notebook_01`, `修仙传_Notebook_02`）。
*   **注意：** 执行完毕后，务必向用户或调用方（如 openclaw/claude code/gemini-cli）清晰列出所有的 Notebook 名称或 ID 列表，并提醒他们跨卷查询时需要切换 Notebook。

## 常用 NLM CLI 备忘
```bash
nlm notebook list
nlm notebook create "Title"
nlm source add <id> --file "01_part.md"
nlm login --check
```
