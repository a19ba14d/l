<a id="advanced-features"></a>
# Advanced Features（高级功能）

- [Custom Rules & `.cursorrules`（自定义规则）](#custom-rules--cursorrules)
- [Keyboard Shortcuts & Productivity（快捷键与效率技巧）](#keyboard-shortcuts--productivity)
- [Version Control Integration（版本控制集成）](#version-control-integration)
- [Privacy & Security Settings（隐私与安全设置）](#privacy--security-settings)

[← 返回总目录](../../README.md)


这一章覆盖“把 Cursor 变成你团队的工具，而不是随机代码生成器”的关键能力：规则、快捷键、Git、隐私安全。

<a id="custom-rules--cursorrules"></a>
## Custom Rules & `.cursorrules`（自定义规则）

**`.cursorrules`** 是放在项目根目录的规则文件，用来告诉 Cursor 在回答/改代码时应当遵守的长期约束（例如风格、目录边界、测试规范、禁止事项）。截至 2024 年 12 月，这是一种常见的“项目级 AI 指令”方式。

> **Note:** Cursor 后续版本可能提供更细粒度的规则系统（例如项目规则目录等）。但无论形式如何，核心思路不变：把“团队约定”写成机器可读的提示，从而减少每次重复说明。

**示例：`.cursorrules`（可直接复制）**

```text
# Project Rules for Cursor (Dec 2024 baseline)

## Code Style
- TypeScript: no `any`, prefer type guards.
- Prefer early returns and small pure functions.
- Keep diffs minimal; do not reformat unrelated code.

## Architecture Boundaries
- Do not import from `src/core/**` into `src/features/**` (dependency direction must remain).
- Database access must go through `src/db/**` repository layer.

## Testing
- New logic must include tests.
- Prefer Vitest; mock external services; avoid snapshot tests unless necessary.

## Security
- Never log secrets, tokens, passwords, or full request bodies.
- Validate all external inputs; avoid raw SQL string concatenation.

## Deliverables
- Always provide: explanation, changed files list, verification commands.
```

> ⚠️ **Warning:** 不要把内部机密（客户数据、访问密钥、专有算法细节）写进 `.cursorrules` 并提交到公共仓库。规则文件会作为提示上下文使用，可能被发送给模型服务端（取决于你的隐私设置与部署方式）。

### Real-world example: 用规则避免“到处改格式”

如果你发现 AI 经常顺手把整个文件重排/格式化，导致 PR diff 巨大，你可以在 `.cursorrules` 加一条硬约束：

```text
- Do not reformat unrelated code. Only change lines necessary for the task.
```

这类规则通常能显著降低“看不清真正改了什么”的 review 成本。

<a id="keyboard-shortcuts--productivity"></a>
## Keyboard Shortcuts & Productivity（快捷键与效率技巧）

Cursor 的快捷键与 VS Code 类似，但 AI 动作通常有独立绑定。由于不同系统/版本可能不同，这里给出“常见默认 + 你应该如何确认”的组合建议：

1. 打开快捷键设置：`Preferences: Open Keyboard Shortcuts`（或在设置中搜索）。
2. 搜索关键词：`Cursor`、`Chat`、`Composer`、`Inline`、`AI`，确认当前绑定。
3. 把你最常用的 2-3 个动作设为顺手按键：
   1. 打开 Chat
   2. Inline Edit 当前选区
   3. 打开 Composer/Agent（多文件）

**效率技巧（不依赖特定快捷键）**

- 选中一段代码再提问：强制 AI 聚焦在你选区，减少跑题。
- 先让它“列清单”，再让它“动手改”：尤其是多文件任务。
- 要求它“生成验证命令”：例如 `pnpm test`、`pytest -q`、`go test ./...`。

### Real-world example: 生成 GitHub Actions 工作流（DevOps）

**需求**：Node 项目在 PR 时跑 lint + test。

**Before（常见问题：无缓存/慢、命令不一致）**

```yaml
name: CI
on:
  pull_request:
    branches: [main]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
      - run: npm ci
      - run: npm run lint
      - run: npm test
```

**After（更快更稳：使用包管理缓存/锁定安装）**

```yaml
name: CI
on:
  pull_request:
    branches: [main]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v4
        with:
          version: 9
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: pnpm
      - run: pnpm install --frozen-lockfile
      - run: pnpm lint
      - run: pnpm test
```

你可以让 Cursor 根据你的 `package.json` 脚本名自动生成，并要求它不要编造不存在的命令。

<a id="version-control-integration"></a>
## Version Control Integration（版本控制集成）

Cursor 继承 VS Code 的 Git 工作流，并在 2024 年末常见地提供一些 AI 辅助能力（例如生成 commit message、根据 diff 做审查建议等）。推荐实践：

1. 用 AI 生成变更后，先在 diff 视图逐文件审查。
2. 在提交前运行最小验证集合（lint/typecheck/test）。
3. 让 AI 帮你写 commit message，但你要确保信息真实、范围准确。

> ⚠️ **Warning:** 让 AI 自动生成 commit message 时，不要把敏感信息写进提交历史（例如客户数据、密钥片段、内部 URL）。提交历史通常长期保留且传播范围更大。

<a id="privacy--security-settings"></a>
## Privacy & Security Settings（隐私与安全设置）

AI 工具的隐私风险主要来自两类：**代码/数据被发送到外部服务**、以及**日志/提示词中包含敏感信息**。你的应对策略应该是“工具设置 + 使用习惯”双管齐下。

- 设置层面（以实际版本选项为准）：
  - 明确是否允许发送代码片段到模型服务端。
  - 明确是否启用代码库索引/云端特性。
  - 在公司环境遵循合规要求（例如仅允许特定模型提供方/自托管）。
- 使用习惯层面：
  - 在粘贴日志前做脱敏：token、cookie、邮箱、手机号、客户标识。
  - 把密钥放在环境变量/密钥管理，不要写入代码与提示词。
  - 对“执行命令/改基础设施”的建议保持更高审查强度。

### Real-world example: 安全地读取 API Key（Backend）

**Before（危险：写死密钥）**

```javascript
export const client = new SomeApiClient({
  apiKey: "sk-1234567890",
});
```

**After（安全：环境变量 + 失败提示）**

```javascript
const apiKey = process.env.SOME_API_KEY;
if (!apiKey) {
  throw new Error("Missing SOME_API_KEY in environment");
}

export const client = new SomeApiClient({ apiKey });
```

---
