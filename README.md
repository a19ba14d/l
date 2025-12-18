# Cursor AI 参考指南（多文件版）

---

## 🚨⚠️ Important Warnings（重要警告 / 必读）

> [!WARNING]
> 🚨 **Security Warnings（安全警告）**
> - **AI 生成/修改的代码可能包含安全漏洞**（如注入、鉴权/权限绕过、XSS/CSRF、SSRF、不安全反序列化等）。
> - **部署前必须逐行验证与加固**：按你团队的安全规范进行威胁建模、代码审计与安全测试。
> - **严禁在提示词、代码、截图、日志或示例中暴露敏感数据/凭证**（API Key、Token、私钥、密码、Cookie、连接串等）；一旦泄露应立即吊销/轮换。
> - 使用 **最小权限（least privilege）**、隔离环境与密钥管理（环境变量/Secret Manager），避免把机密写入仓库。
> - 对 AI 建议的 **依赖包/脚本命令/基础设施改动** 进行供应链、权限与许可证风险评估。

> [!WARNING]
> ⚠️ **Large-Scale Code Generation Risks（大规模代码生成风险）**
> - **大段自动生成代码可能降低可读性与可维护性**：风格不一致、重复实现、架构漂移、隐藏复杂度。
> - 在规模化改动下更容易引入 **隐蔽 Bug、边界条件缺陷与技术债**，后期修复成本会指数上升。
> - **大量生成的 diff 更难审查与调试**；建议小步迭代、可回滚提交、分阶段验证，确保你能解释每一处变更。
> - 生成代码可能带来 **性能/资源开销问题**（低效算法、N+1 查询、无界循环/重试、过度分配）；上线前务必基准测试与 Profiling。

> [!WARNING]
> ✅ **Code Review & Testing Requirements（代码审查与测试要求）**
> - **所有生成或修改的代码在用于生产环境前 MUST 经过严格人工审查。**
> - 自动化代码生成 **不能替代人类责任与把关**；请将输出视作来自“不可信第三方”的代码。
> - **不理解就不要上线**：确保你理解控制流、失败模式、权限边界与安全假设。
> - **测试是必需的**：运行 `lint/typecheck/unit/integration`，为关键路径补齐测试覆盖，并在预发/灰度环境验证。

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
