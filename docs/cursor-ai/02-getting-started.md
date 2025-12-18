<a id="getting-started"></a>
# Getting Started（快速开始）

- [Installation（安装）](#installation)
- [Initial Setup & Configuration（初始设置与配置）](#initial-setup--configuration)
- [Interface Overview（界面概览）](#interface-overview)

[← 返回总目录](../../README.md)


<a id="installation"></a>
## Installation（安装）

1. 打开 Cursor 官网下载页（通常提供 macOS / Windows / Linux 安装包）。
2. 安装并启动 Cursor；首次启动通常会引导你登录/创建账号。
3. （可选）从 VS Code 迁移设置：
   1. 导入主题、快捷键、扩展（若版本支持一键迁移）。
   2. 检查 `Settings` 中与 AI、隐私、索引相关的选项是否符合你的团队规范。
4. （可选）启用命令行打开能力：安装/启用 `cursor` 命令后，你可以在项目目录执行 `cursor .` 快速打开。

> ⚠️ **Warning:** 如果你选择“自带 API Key/自定义模型提供方”，务必避免把 Key 写进仓库文件（如 `.env` 误提交）。优先使用系统钥匙串、CI 的密钥管理、或环境变量（如 `OPENAI_API_KEY`、`ANTHROPIC_API_KEY`）并配置 `.gitignore`。

<a id="initial-setup--configuration"></a>
## Initial Setup & Configuration（初始设置与配置）

1. 打开一个现有仓库或新建项目（建议先用一个你熟悉的项目练手）。
2. 等待索引/扫描（如果版本需要对代码库建立索引）：这会提升“全仓理解”与跨文件改动的质量。
3. 配置基础偏好：
   1. 语言服务器/格式化：例如启用 `ESLint`、`Prettier`、`ruff`、`black`、`gofmt` 等。
   2. AI 行为：是否允许联网检索、是否允许上传片段、是否启用更激进的多文件自动改动等。
   3. 安全：在公司环境下，优先开启隐私相关选项并遵循合规要求。
4. 为项目添加 AI 规则（推荐）：在仓库根目录创建 `.cursorrules`（见高级功能章节）。

<a id="interface-overview"></a>
## Interface Overview（界面概览）

Cursor 的布局整体继承 VS Code：左侧活动栏/侧边栏、中央编辑区、底部面板（终端/问题/输出）。AI 相关交互通常集中在以下入口（不同版本按钮位置可能略有差异）：

- **Chat（对话面板）**：用于问答、解释代码、请求修改建议、生成方案等。
- **Inline Edit（就地编辑）**：在编辑器中对选中代码提出“把这段改成…”并直接应用。
- **Composer/Agent（多文件任务）**：适合“加一个功能并改动多个文件”“迁移一个模块”等任务，通常会先规划再执行，并给出可审查 diff。
- **终端与输出**：把错误日志、构建输出、测试失败信息作为上下文喂给 AI，能显著提升修复准确率。

> **Tip:** 在提问前先做两件事：1）明确目标（你希望最终怎样）；2）明确约束（不能改什么、必须遵守什么）。这两点比“多说几句话”更能提升结果质量。

---
