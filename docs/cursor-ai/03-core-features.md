<a id="core-features"></a>
# Core Features（核心功能）

- [AI-Powered Code Completion（AI 代码补全）](#ai-powered-code-completion)
- [Chat Interface（聊天界面）](#chat-interface)
- [Code Generation（代码生成）](#code-generation)
- [Multi-File Editing（多文件编辑）](#multi-file-editing)
- [Terminal Integration（终端集成）](#terminal-integration)

[← 返回总目录](../../README.md)


下面按“你在日常开发中最常用的能力”来组织：补全、聊天、生成、多文件改动、终端闭环。

<a id="ai-powered-code-completion"></a>
## AI-Powered Code Completion（AI 代码补全）

**Cursor Tab（AI 补全）** 的价值不只是“补全下一行”，而是：在你已有上下文（当前文件、相关 import、相邻函数）下，给出多行实现、边界处理、甚至风格一致的代码片段。

- 适合：
  - 重复性样板：DTO/类型声明、表单校验、API 客户端封装。
  - 小范围逻辑：数组/对象变换、错误处理、简单状态机。
- 不适合（建议改用 Chat/Composer）：
  - 需要跨文件协调的改动（新增路由 + 控制器 + 测试 + 文档）。
  - 需要严格遵循架构边界（分层、依赖方向）的大改动。

**小技巧**

- 先写“函数签名 + 注释 + 示例输入输出”，再让补全补齐实现，通常更稳定。
- 如果补全偏离方向，撤销并补上 1-2 行关键约束（比如“不要引入新依赖”“用现有 `logger`”）。

<a id="chat-interface"></a>
## Chat Interface（聊天界面）

Chat 更适合“解释 + 方案 + 小幅改动建议”。你可以：

- 让它解释报错、解释一段代码做了什么、给出重构建议。
- 让它基于你引用的文件/选区，产出一个可执行的修改计划（plan）。
- 让它对比两种实现的权衡（可维护性、性能、可测试性、安全性）。

> **Tip:** 你越能让 AI “看到”关键上下文（相关文件、调用链、约束），它越不需要猜。猜得越少，返工越少。

<a id="code-generation"></a>
## Code Generation（代码生成）

代码生成常见有三类：

- **从 0 到 1**：生成一个新模块/脚本/组件骨架。
- **从 1 到 N**：批量生成同构代码（例如为 10 个 API 生成客户端）。
- **从 N 到 1**：把零散实现整理成规范结构（例如把 notebook 迁移为包）。

> ⚠️ **Warning:** 批量生成/批量改动会放大错误影响。你应当要求 AI：
> 1）先输出计划与受影响文件清单；2）分批提交 diff；3）每批都可运行测试/构建验证。

<a id="multi-file-editing"></a>
## Multi-File Editing（多文件编辑）

多文件任务最常见的失败原因是：需求不清晰 + 约束没说 + AI 自作主张改了架构关键点。推荐工作流：

1. 先在 Chat 里让它输出“方案 + 影响范围 + 风险点”。
2. 再切到 Composer/Agent 执行多文件改动，并要求：
   1. 保持改动最小化（minimal diff）。
   2. 逐文件解释“为什么改、改了什么、怎么验证”。
   3. 避免引入新依赖（除非你明确允许）。
3. 应用变更后立刻运行：`lint`、`typecheck`、`test`（至少其一）。

### Real-world example: 在 React 组件中修复“受控/非受控”警告（Web）

**场景**：你有一个表单组件在某些情况下 `value` 变成 `undefined`，React 报 “A component is changing an uncontrolled input to be controlled”。

**Before**

```tsx
import React from "react";

type Props = {
  value?: string;
  onChange: (v: string) => void;
};

export function NameInput({ value, onChange }: Props) {
  return (
    <input
      value={value}
      onChange={(e) => onChange(e.target.value)}
      placeholder="Your name"
    />
  );
}
```

**After（更稳健：永远传 string，且可扩展）**

```tsx
import React from "react";

type Props = {
  value?: string | null;
  onChange: (v: string) => void;
};

export function NameInput({ value, onChange }: Props) {
  return (
    <input
      value={value ?? ""}
      onChange={(e) => onChange(e.target.value)}
      placeholder="Your name"
    />
  );
}
```

你可以让 Cursor 在 Composer 中同时改动：组件、相关调用方默认值、以及补一个回归测试（如果有）。

<a id="terminal-integration"></a>
## Terminal Integration（终端集成）

终端集成的关键是把“反馈”喂回去：构建失败、测试失败、Lint/Typecheck 报错，是最强的监督信号。

1. 运行命令并复制输出（或在 Cursor 中直接引用终端输出作为上下文）。
2. 在提示词里要求：
   1. 先定位根因（root cause），再给修复方案。
   2. 给出最小 diff，并说明验证步骤。

> ⚠️ **Warning:** 不要直接执行 AI 建议的破坏性命令（如 `rm -rf`、重置分支、清空数据库）。在执行前，你应当让它解释命令影响范围，并自己确认。

### Feature matrix（功能矩阵）

| 你要达成的目标 | 推荐入口 | 你应当提供的上下文 | 你应当要求的输出 |
| --- | --- | --- | --- |
| 让 AI 补齐一段实现（10-30 行） | Cursor Tab（补全） | 函数签名、注释、示例输入输出 | 可直接接受的多行补全 |
| 解释/定位 Bug（需要推理） | Chat（对话） | 报错、堆栈、复现步骤、相关 `@file` | 根因 + 最小修复方案 + 验证命令 |
| 只改一个局部片段 | Inline Edit（就地编辑） | 选区 `@code` + 约束（不改别处） | 仅修改选区的最小改动 |
| 跨文件实现功能/迁移 | Composer/Agent（多文件） | 文件/目录清单 `@file/@folder` + 架构约束 | 计划 → 逐文件 diff → 回滚/验证步骤 |
| 修复构建/测试失败 | Terminal + Chat | 失败命令、完整输出、环境版本 | 修复点定位 + 最小 diff + 复跑命令 |

---
