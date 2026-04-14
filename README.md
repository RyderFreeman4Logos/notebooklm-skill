# NotebookLM Skill

高级 NotebookLM 数据处理与上传流水线 (万物转 Markdown + Tokuin 智能切分)。
该 Skill 专为各大 AI 代理 (Agent) 框架设计，彻底解决 NotebookLM 对超长文档的静默截断和 50 个源上限问题。

## 功能特性 (Features)
* **智能策略路由**: 根据文件类型（PDF、中文小说、代码库、字幕等）自动选择最优解析方案。
* **强制前置审查**: 无论文件页数多少，所有文件在上传前都将由底层 Token 计价器 (`tokuin`) 进行强检验，保证每次请求绝不超过模型的 100 万 Token 安全线。
* **全自动依赖自修**: 在执行环境中如遇缺失依赖（如 `nlm`、`tokuin`、`pdftk`），流水线将通过 `mise` 等包管理器自动完成后台静默安装。

## 安装与使用 (Installation & Usage)

### 方法一：通过包管理器直接导入
如果你的 Agent 环境支持标准的 NPM/NPX 指令，您可以直接执行添加命令：
```bash
npx skills add https://github.com/RyderFreeman4Logos/notebooklm-skill
```

### 方法二：给 Agent 发送直链 (推荐)
对于现代的开发端 AI 代理（如 `openclaw`, `claude code`, `gemini-cli`, `antigravity` 等），您可以直接在对话中将远端协议链接发送给它：

```text
请安装并使用这个 NotebookLM 处理技能：
https://github.com/RyderFreeman4Logos/notebooklm-skill/raw/refs/heads/main/skills/notebooklm/SKILL.md
```

Agent 在读取到该直链后，会自动拉取并应用该 Skill 的核心规则与处理流水线。

---
**License**: Apache-2.0
