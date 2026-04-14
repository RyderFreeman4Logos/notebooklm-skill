# 策略：PDF 书籍与文档 (PDF Documents)

## 背景与痛点
虽然我们的核心理念是“万物转 Markdown”，但由于 PDF 的版式通常非常复杂（双栏、图片穿插、复杂的页眉页脚），使用常规工具强制将 PDF 转为 Markdown 极易丢失重要信息，或者产生大量错乱的无结构文本，甚至消耗巨大的 Token（如果依赖 VLM API 转换）。

## 核心策略

1.  **首选：直接物理切分 PDF (最安全、最低成本)**
    *   **无需转 Markdown：** 如果源文件已经是排版精良的 PDF，**强烈建议不要转 Markdown**（不要消耗无谓的 Token）。NotebookLM 底层依赖 Google Document AI 处理 PDF，能完美利用物理分页（Page Breaks）作为 Chunking 边界。
    *   **拆分方式：** 直接根据页码拆分 PDF。
    *   **安全阈值：** 考虑到 PDF 的解析元数据和潜在的中日文字符密集度，建议将单卷 PDF 控制在 **300 页 - 400 页**以内（保守估计以防撞上隐形 Token 墙）。
    *   **工具推荐：** 使用 `qpdf` (`pip install pymupdf`) 或系统自带的 `pdftk` 进行页面范围切分。
        ```bash
        pdftk input.pdf cat 1-300 output part1.pdf
        pdftk input.pdf cat 301-600 output part2.pdf
        ```

2.  **备选：如果用户强烈要求提取纯文本**
    *   如果用户明确表示该 PDF 只有纯文本内容且必须做语义合并，可以使用轻量级工具提取文本，然后转入 `cn-novel` 或 `codebase` 的处理流程。
    *   **警告：** 这极易翻车，必须提醒用户确认提取的 MD 没有乱码。

3.  **命名规范**
    *   切分后的 PDF 应当有序命名：`01_书名_页001-300.pdf`, `02_书名_页301-600.pdf`。
