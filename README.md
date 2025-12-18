# Cursor AI 参考指南（多文件版）

- [Introduction（简介）](docs/cursor-ai/01-introduction.md#introduction)
  - [Key Features（关键特性与能力）](docs/cursor-ai/01-introduction.md#key-features)
  - [Target Audience & Use Cases（适用人群与典型场景）](docs/cursor-ai/01-introduction.md#target-audience--use-cases)
- [Getting Started（快速开始）](docs/cursor-ai/02-getting-started.md#getting-started)
  - [Installation（安装）](docs/cursor-ai/02-getting-started.md#installation)
  - [Initial Setup & Configuration（初始设置与配置）](docs/cursor-ai/02-getting-started.md#initial-setup--configuration)
  - [Interface Overview（界面概览）](docs/cursor-ai/02-getting-started.md#interface-overview)
- [Core Features（核心功能）](docs/cursor-ai/03-core-features.md#core-features)
  - [AI-Powered Code Completion（AI 代码补全）](docs/cursor-ai/03-core-features.md#ai-powered-code-completion)
  - [Chat Interface（聊天界面）](docs/cursor-ai/03-core-features.md#chat-interface)
  - [Code Generation（代码生成）](docs/cursor-ai/03-core-features.md#code-generation)
  - [Multi-File Editing（多文件编辑）](docs/cursor-ai/03-core-features.md#multi-file-editing)
  - [Terminal Integration（终端集成）](docs/cursor-ai/03-core-features.md#terminal-integration)
- [Prompt Engineering & Best Practices（提示词工程与最佳实践）](docs/cursor-ai/04-prompt-engineering-best-practices.md#prompt-engineering--best-practices)
  - [Prompt Templates（提示词模板）](docs/cursor-ai/04-prompt-engineering-best-practices.md#prompt-templates)
  - [Effective vs. Ineffective Prompts（好坏提示词对比）](docs/cursor-ai/04-prompt-engineering-best-practices.md#effective-vs-ineffective-prompts)
  - [Best Practices（最佳实践）](docs/cursor-ai/04-prompt-engineering-best-practices.md#best-practices)
  - [Context Management（上下文管理）](docs/cursor-ai/04-prompt-engineering-best-practices.md#context-management)
  - [Troubleshooting（排错指南）](docs/cursor-ai/04-prompt-engineering-best-practices.md#troubleshooting)
- [Advanced Features（高级功能）](docs/cursor-ai/05-advanced-features.md#advanced-features)
  - [Custom Rules & `.cursorrules`（自定义规则）](docs/cursor-ai/05-advanced-features.md#custom-rules--cursorrules)
  - [Keyboard Shortcuts & Productivity（快捷键与效率技巧）](docs/cursor-ai/05-advanced-features.md#keyboard-shortcuts--productivity)
  - [Version Control Integration（版本控制集成）](docs/cursor-ai/05-advanced-features.md#version-control-integration)
  - [Privacy & Security Settings（隐私与安全设置）](docs/cursor-ai/05-advanced-features.md#privacy--security-settings)
- [References & Official Resources（参考与官方资源）](docs/cursor-ai/06-references.md#references--official-resources)

**Last Updated:** 2024-12-15

> This guide was created in December 2024. Cursor AI is actively developed and features may change. Always verify current information from official documentation.
>
> 本指南以 2024 年 12 月的 Cursor 功能与交互习惯为基线编写。你在实际使用时，应以 `Help`、`Settings`、以及官方文档/更新日志为准。

---

## 如何使用本指南

1. 从上面的目录进入你需要的章节。
2. 如果你在 Cursor 里实践某个提示词模板：先复制模板，再把占位符 `<...>` 替换成你项目的真实信息（错误输出、文件路径、约束）。
3. 涉及多文件改动时，优先使用“先计划后执行”的方式，并在每次应用 diff 后运行 `lint/typecheck/test` 进行验证。

## 文档结构

- 主入口：`README.md`
- 章节目录：`docs/cursor-ai/`
  - `docs/cursor-ai/01-introduction.md`
  - `docs/cursor-ai/02-getting-started.md`
  - `docs/cursor-ai/03-core-features.md`
  - `docs/cursor-ai/04-prompt-engineering-best-practices.md`
  - `docs/cursor-ai/05-advanced-features.md`
  - `docs/cursor-ai/06-references.md`
