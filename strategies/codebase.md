# 策略：代码库与技术文档项目 (Codebases & Documentation)

## 背景与痛点
用 NotebookLM 读代码或大型技术文档（如 Repomix / Aider 生成的巨型 XML 或全项目拼合 Markdown）是高频需求。代码的 Chunking 对大模型来说至关重要，一旦把一个 Class 或 Function 切成两半，模型的上下文推理就会彻底断裂。

## 核心策略：维持 AST/文件完整性

1.  **绝对禁止基于字符或 Token 的硬切分**
    *   不能像切小说那样到了 70万 Token 就立刻切一刀。代码必须保持完整性。
    *   **最小切分单元必须是“单个文件 (File)”或者“完整的 Markdown 模块”。**

2.  **如何处理超大代码库打包 (如 Repomix 产物)**
    *   如果你拿到了一个包含整个项目的巨大 `output.md`，里面的结构通常是：
        ```markdown
        ## File: src/main.rs
        ```
    *   **切分算法：**
        1.  使用 `tokuin` 实时累计 Token (`--model gemini-1.5-pro`)。
        2.  遇到 `## File: ` 标记时，检查当前累计 Token 是否超过了安全阈值（例如 **700,000 Token**）。
        3.  如果超过，就在**这个 `## File:` 标记之前**切分，将前面所有的文件打包为一个 `.md` 作为 Source。
        4.  绝不允许在一个文件的代码块内部切断。如果单一一个文件本身就超过了 70万 Token，将其独立拿出来作为一个唯一的 Source 即可（极端情况）。

3.  **添加全局上下文引导 (Global Context Injection)**
    *   对于切分后的每一部分代码 `.md`，**强烈建议在文件最开头动态注入一段简短的声明**：
        `> [系统提示] 这是项目代码库的第 X 部分。本项目主要用于实现 YYY 功能。`
    *   这能极大缓解 NotebookLM 跨源 (Cross-Source) 查询时的上下文遗忘症。

4.  **有序命名**
    *   `01_ProjectName_Code_Part1.md`
    *   `02_ProjectName_Code_Part2.md`
