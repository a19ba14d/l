<a id="prompt-engineering--best-practices"></a>
# Prompt Engineering & Best Practices（提示词工程与最佳实践）

- [Prompt Templates（提示词模板）](#prompt-templates)
- [Effective vs. Ineffective Prompts（好坏提示词对比）](#effective-vs-ineffective-prompts)
- [Best Practices（最佳实践）](#best-practices)
- [Context Management（上下文管理）](#context-management)
- [Troubleshooting（排错指南）](#troubleshooting)

[← 返回总目录](../../README.md)


这一章是本指南的重点：你写给 Cursor 的每一句话，都会影响它“理解问题的方式”和“做出改动的范围”。高质量提示词的核心不是“写得长”，而是：

- **目标清晰**：你要它做什么、交付什么。
- **上下文充分**：它需要哪些文件/日志/复现步骤才能不靠猜。
- **约束明确**：不能改什么、必须遵守什么（版本、风格、性能、安全）。
- **验证闭环**：怎么证明改动是对的（测试/命令/场景）。

<a id="prompt-templates"></a>
## Prompt Templates（提示词模板）

下面给出 15 个“可直接复制粘贴”的模板。你只需要替换其中的占位符（`<...>`），并按需删除无关字段即可。

### Error fixing template（错误修复模板：运行时/堆栈）

```text
你是我的结对开发者。请帮我修复一个报错，并给出最小改动的补丁方案。

【目标】
- 修复错误并解释根因（root cause）
- 不要引入新依赖；保持现有公共 API 不变

【环境】
- 语言/版本：<例如 Node 20 / Python 3.11>
- 框架/版本：<例如 Next.js 14 / FastAPI 0.104>
- 运行方式：<例如 pnpm dev / pytest>

【错误信息】
<粘贴完整 error message>

【堆栈/日志】
<粘贴 stack trace 或相关日志 30-200 行>

【复现步骤】
1. <步骤 1>
2. <步骤 2>
3. <观察到的结果 vs 期望结果>

【相关代码】
- @file <路径 1>（包含报错附近 20-50 行）
- @file <路径 2>（调用链上游/下游）

【输出要求】
1. 先用要点列出根因与修复思路
2. 给出需要修改的文件清单
3. 按文件给出具体修改（可直接应用的代码片段）
4. 给出我应该运行的验证命令（lint/test）
```

### Type/build error template（错误修复模板：类型/构建失败）

```text
请修复以下 TypeScript/构建错误，并保证通过 typecheck 与 lint。

【约束】
- TypeScript 版本：<例如 5.x>
- 不要使用 any；尽量保持类型收敛
- 不要改动对外导出的类型/函数签名（除非必要，请先说明原因）

【错误输出】
<粘贴 tsc / build 的完整输出>

【相关上下文】
- @file <相关文件 1>
- @file <相关文件 2>
- 如果涉及类型推导，请同时看 import 的类型来源文件

【期望交付】
- 给出修改后的代码
- 解释为什么这样改能消除错误
- 指出潜在的边界情况/回归风险
```

### Refactor template（重构模板：提取方法/简化条件/拆分组件）

```text
请帮我对下面的代码做重构，目标是提高可读性与可测试性，同时保持行为不变。

【重构目标（可多选）】
- 提取方法（extract method）
- 简化条件分支（simplify conditionals）
- 拆分超大函数/组件（split large function/component）
- 引入更清晰的命名与早返回（early return）

【约束】
- 不改变对外行为（包括错误类型、返回值、日志语义）
- 不引入新依赖
- 保持现有公共 API

【代码位置】
- @file <路径>（请基于整文件重构，不要只看片段）

【输出要求】
1. 先总结当前代码的“坏味道”（code smells）
2. 给出分步重构计划（每一步可单独提交）
3. 给出最终代码
4. 建议补充/更新的测试点
```

### Refactor-to-pattern template（重构模板：指定模式/范式）

```text
我希望你把下面实现按指定模式重构，并解释权衡。

【指定模式】
- <例如：Strategy / Adapter / Repository / 状态机 / 依赖注入>

【约束】
- 维持现有入口函数签名
- 性能不退化（说明大 O 或主要开销）
- 给出迁移步骤（避免一次性大改）

【上下文】
- @file <涉及的核心文件>
- @folder <目录>（如果需要了解相邻模块）

【输出】
- 新的结构图/模块划分（用列表即可）
- 每个文件的职责说明
- 关键代码片段
```

### Unit test template（写测试模板：单元测试）

```text
请为以下函数/模块补充单元测试，要求覆盖正常路径与主要边界情况。

【测试框架】
- <例如：Jest / Vitest / Pytest / Go test>

【约束】
- 不要依赖真实外部服务（用 mock/fake）
- 测试应当可读、可维护，避免脆弱断言

【被测代码】
- @file <路径>
- 指出你要测试的函数/类：<名称>

【你需要输出】
1. 新增/修改的测试文件路径
2. 完整的测试代码（可直接运行）
3. 每个测试用例验证了什么
4. 还缺哪些测试（如果需要集成测试才覆盖）
```

### Integration test template（写测试模板：集成测试/API 测试）

```text
请为以下 API/流程补充集成测试，验证“从入口到数据库/外部依赖的边界”。

【测试目标】
- 覆盖成功场景 + 失败场景（鉴权失败/参数错误/资源不存在）
- 验证返回码、返回结构、以及关键副作用（例如写库）

【环境与依赖】
- 数据库：<例如 Postgres>
- 是否已有测试容器/测试数据库：<是/否>
- 测试框架：<例如 Jest + supertest>

【上下文】
- @folder <api 相关目录>
- @file <路由/控制器文件>
- @file <数据访问层文件>

【输出】
- 测试用例清单（先列出来）
- 然后给出可运行的测试实现
- 说明如何本地运行与在 CI 中运行
```

### Edge-case coverage template（写测试模板：边界/属性测试/随机化）

```text
请分析下面逻辑可能遗漏的边界情况，并补齐测试用例。

【你需要做的事】
1. 列出至少 10 个边界/异常输入（含空值、极值、非法格式、重复、乱序）
2. 选出最关键的 5-7 个写成测试
3. 如果适合，补一个属性测试/随机化测试（可选）

【上下文】
- @file <路径>
- 业务约束（域规则）：<例如金额必须 >=0，日期必须是 ISO8601>
```

### Docstring/JSDoc template（文档生成模板：docstrings / JSDoc）

```text
请为以下代码生成高质量的 docstring/JSDoc，要求包含：
- 参数解释（含单位/约束/默认值）
- 返回值/异常
- 最小可运行示例
- 边界条件说明

【风格要求】
- <例如：Google Python docstring / TypeScript JSDoc>

【上下文】
- @file <路径>
```

### README template（文档生成模板：README/使用说明）

```text
请为这个项目/模块补齐 README 的一个章节：<章节名，例如“本地开发”“部署”“故障排查”>。

【目标读者】
- <例如：新入职同事/开源用户/运维>

【约束】
- 命令必须真实可运行（不要编造脚本名）
- 如果缺少信息，请先列出你需要我补充的 3-5 个问题

【上下文】
- @file README.md
- @file package.json / pyproject.toml / Dockerfile（按项目选择）
- @folder <重要目录>
```

### API documentation template（文档生成模板：API 文档/契约）

```text
请根据现有代码生成 API 文档（面向调用方），并指出不一致/缺失的地方。

【输出格式】
- Markdown 表格：Endpoint / Method / Auth / Request / Response / Errors / Notes
- 如能推导出示例 curl，也请给出

【上下文】
- @folder <routes/controllers>
- @file <schema/validator>
```

### Security review template（代码审查模板：安全/隐私）

```text
请对以下改动做一次安全与隐私审查（像做安全 PR Review 一样），并给出修复建议。

【重点关注】
- 鉴权/越权（IDOR）
- 输入校验与注入（SQL/命令/模板）
- 敏感信息泄露（日志、错误返回、前端暴露）
- 依赖与供应链风险（新增依赖/外部请求）

【上下文】
- @diff <这次改动的 diff>（或 @file 列出改动文件）
- 系统威胁模型简述：<谁是攻击者、资产是什么、边界在哪里>

【输出】
- 风险列表（严重程度/影响/复现方式）
- 具体修复建议（代码级）
- 建议加入的安全测试/日志
```

### Performance review template（代码审查模板：性能/可维护性）

```text
请对以下代码做性能与可维护性审查，找出瓶颈与潜在回归点。

【约束】
- 不改变外部行为
- 优先通过算法/缓存/批处理优化，而不是“加机器”

【上下文】
- @file <路径>
- 性能指标：<例如 P95 延迟目标、最大数据量、QPS>
- 运行环境：<例如 Serverless / K8s>

【输出】
- 可能的热点（按影响排序）
- 具体优化建议（含复杂度/代价）
- 如何做基准测试与验证（benchmark/指标）
```

### Architecture design template（架构设计模板：系统/组件/数据流）

```text
请帮我做一个轻量系统设计（不需要长文档，但要结构清晰），并给出落地实施步骤。

【需求】
- 功能：<描述用户故事/核心需求>
- 非功能：<性能/可靠性/安全/可观测性>

【约束】
- 技术栈：<例如 React 18 + Node + Postgres>
- 不能引入的组件：<例如不允许新增消息队列>

【输出】
1. 组件划分与职责（列表）
2. 关键数据模型（表结构/类型）
3. 关键流程的数据流（用编号步骤描述）
4. 风险点与替代方案
5. 分阶段实施计划（每阶段交付物与验证方式）
```

### Database query template（数据库模板：SQL/索引/迁移）

```text
请帮我优化以下查询/表结构，并给出可验证的改进方案。

【数据库】
- 类型与版本：<例如 Postgres 15>

【当前问题】
- 症状：<慢查询/锁/高 CPU/高 IO>
- 指标：<例如 2s -> 200ms>

【输入】
- SQL：<粘贴查询>
- 表结构：<粘贴 schema 或相关 ORM model>
- EXPLAIN/执行计划：<如果有>

【输出】
- 索引建议（含为什么）
- 查询改写建议
- 迁移脚本示例（如果需要）
- 回滚策略与验证步骤
```

### API integration template（API 集成模板：接第三方服务）

```text
请帮我把第三方 API 集成到现有代码中，要求可配置、可测试、可观测。

【第三方 API】
- 文档链接：<URL>
- 鉴权方式：<API Key/OAuth>
- 速率限制：<如果有>

【约束】
- API Key 只能通过环境变量注入（不要写死）
- 需要超时、重试（带退避）、以及可观测日志（不泄露敏感信息）

【上下文】
- @folder <相关模块目录>
- @file <http client 封装/配置文件>

【输出】
1. 代码改动（含配置、客户端、调用点）
2. 单元测试（mock 第三方）
3. 失败策略（超时、429、5xx）
```

### Migration assistance template（迁移模板：升级/替换依赖/语言版本）

```text
请帮我做一次“可控”的迁移：从 <旧版本/旧库> 升级到 <新版本/新库>。

【迁移范围】
- 受影响模块：<列出目录/包>
- 不能停机/可停机：<要求>

【约束】
- 分阶段、小步提交
- 每阶段必须可回滚
- 需要更新测试与文档

【上下文】
- @file <依赖声明文件，如 package.json/requirements.txt>
- @folder <核心代码目录>

【输出】
1. 迁移计划（阶段、风险、回滚）
2. 每阶段需要修改的文件清单
3. 关键代码改动示例
4. 我应该运行的验证命令
```

---

<a id="effective-vs-ineffective-prompts"></a>
## Effective vs. Ineffective Prompts（好坏提示词对比）

下面用 6 组对比，展示“同一个需求”如何从模糊变清晰。你会发现：有效提示词通常包含 **上下文**、**约束**、**期望输出格式**，并把“猜”的空间压到最小。

### Example 1：修复报错（运行时）

| Ineffective Prompt（无效） | Effective Prompt（有效） |
| --- | --- |
| 帮我修这个报错。 | 请修复这个错误并解释根因：`TypeError: Cannot read properties of undefined (reading 'id')`。复现：1）进入 `/users`；2）点击“导出”。堆栈见下（完整粘贴）。相关文件：@file `src/pages/users.tsx`（报错行前后 50 行）、@file `src/api/users.ts`。约束：不引入新依赖；保持导出 API 返回结构不变。输出：根因→最小 diff→验证命令。 |

解释：有效版本把“错误、复现、调用链上下文、约束、输出格式”都给齐了，AI 不需要猜“在哪里、怎么复现、改到什么程度”。这会显著降低它给出“泛泛建议”或改错地方的概率。

### Example 2：重构

| Ineffective Prompt（无效） | Effective Prompt（有效） |
| --- | --- |
| 把这段代码重构一下。 | 请对 @file `src/lib/pricing.ts` 做重构：目标是提取方法、简化条件分支、提高可测试性（保持行为不变）。约束：不改 public API、不引入新依赖、保留现有日志语义。输出：1）列出坏味道；2）分步重构计划（每步可单独提交）；3）最终代码；4）建议补的测试点。 |

解释：无效提示词没有目标与边界，AI 可能进行“美化式重构”或引入破坏性改动。有效提示词把重构类型、约束与交付物明确化，使结果更可控、也更易 code review。

### Example 3：写测试

| Ineffective Prompt（无效） | Effective Prompt（有效） |
| --- | --- |
| 给我写点测试。 | 请为 @file `src/auth/jwt.ts` 的 `verifyToken()` 补单元测试（Vitest）。覆盖：1）合法 token；2）过期 token；3）签名错误；4）缺少 `sub` claim；5）时钟偏差（±30s）。约束：不访问真实网络/外部服务；mock 系统时间。输出：测试文件路径 + 完整测试代码 + 每个用例验证点。 |

解释：有效版本明确被测对象、测试框架、覆盖点与 mock 策略。AI 更容易生成能跑、且不脆弱的测试，而不是“只测 happy path”。

### Example 4：性能优化

| Ineffective Prompt（无效） | Effective Prompt（有效） |
| --- | --- |
| 这段代码太慢了，优化一下。 | 请分析 @file `src/jobs/recompute.ts` 的性能瓶颈。输入规模：每天 500 万条记录；目标：P95 < 2s。请先定位主要热点（算法/IO/序列化/查询），再给 2-3 个可落地优化方案（含复杂度与代价），并提供一个最小基准测试方法用于验证。 |

解释：性能问题必须有“输入规模与指标”，否则 AI 只能给通用建议。有效提示词要求先定位热点再提方案，并要求验证方法，让优化更可量化。

### Example 5：架构设计

| Ineffective Prompt（无效） | Effective Prompt（有效） |
| --- | --- |
| 设计一个系统。 | 需求：做一个“团队内部代码片段分享”服务。技术栈：React 18 + Node + Postgres。约束：不能新增消息队列；必须支持 SSO；隐私：片段默认私有且可审计。请输出：组件划分、数据模型、关键流程（上传/分享/权限校验）、风险点与替代方案、分阶段实施计划（每阶段可交付与验证）。 |

解释：系统设计最怕“无限发挥”。有效版本提供了业务故事、约束与安全要求，并规定输出结构，使设计可落地、可评审。

### Example 6：安全审查

| Ineffective Prompt（无效） | Effective Prompt（有效） |
| --- | --- |
| 帮我看看有没有安全问题。 | 请对这个 PR 的变更做安全审查：@diff（粘贴或引用）。重点：鉴权/越权、注入（SQL/命令）、敏感信息泄露、依赖风险。请输出：风险列表（严重程度/影响/复现）+ 代码级修复建议 + 建议补的安全测试。 |

解释：安全审查需要明确“看什么、怎么输出”。有效提示词把威胁面收敛到关键类别，并要求可执行的修复建议，避免“空泛的安全提醒”。

---

<a id="best-practices"></a>
## Best Practices（最佳实践）

这一节把“写提示词”拆成可操作的习惯：给上下文、拆任务、迭代收敛、声明约束。

### Providing context（提供上下文）

**核心原则：让 AI 看见它必须看见的内容，隐藏它不该看见的内容。**

- 代码片段给多少？
  - 小修复（单点 bug）：给 **20-50 行**，覆盖报错行前后与调用点。
  - 中等改动（重构/新增分支逻辑）：给 **整个文件**，避免它破坏局部一致性。
  - 跨模块改动（新增功能/迁移）：给 **关键入口文件 + 相邻模块目录结构**，并列出受影响文件清单。
- 什么时候用 `@file` / `@folder` / `@code`（或等价引用方式）？
  - `@file`：当你希望它基于“文件整体结构”做修改（重构、加导出、调整类型）。
  - `@folder`：当你需要它理解模块边界、相邻文件命名规律、或现有实现模式。
  - `@code`（代码选区）：当你只希望它改一小段，并且你不希望它改动更大范围。

> ⚠️ **Warning:** 不要把密钥、客户数据、访问令牌直接贴进聊天窗口。若必须提供日志，请先脱敏（mask）并删除敏感字段。

**多文件引用的推荐写法**

- 先列清单（让 AI 知道范围），再给重点文件：
  - `@file src/routes/users.ts`
  - `@file src/controllers/usersController.ts`
  - `@file src/db/usersRepo.ts`
  - `@file src/__tests__/users.test.ts`
- 明确你允许它改哪些文件、不允许改哪些：
  - “只允许改 `src/users/**` 与测试目录；不要改 `src/core/**`。”

### Divide and conquer（拆解复杂请求）

当任务复杂（涉及设计、迁移、多文件）时，最有效的方法是“先拆再做”。你可以用下面的模式：

1. **让 AI 先输出计划**（Plan），并要求你确认后再执行。
2. 把计划拆成可独立验证的阶段（每阶段都有可运行的验证命令）。
3. 每阶段只做一件事：避免“同时改功能 + 重构 + 升级依赖”。

**示例 1：把遗留路由迁移到新路由系统**

1. 盘点：列出旧路由与调用方（受影响页面/接口）。
2. 兼容层：先加一个适配层（旧接口调用新实现）。
3. 逐步迁移：一次迁一个路由 + 对应测试。
4. 删除旧代码：全部迁完再删，并加回归测试。

**示例 2：为项目加上统一日志与追踪**

1. 选标准：定义 `logger` 接口（字段、级别、trace id）。
2. 接入入口：HTTP 中间件生成/传播 trace id。
3. 替换重点路径：先替换最关键的 2-3 条链路。
4. 扩面：批量替换其余模块（分批提交）。

**示例 3：数据库迁移（新增列并回填）**

1. 第一阶段：加列（允许为空）+ 代码写入新列。
2. 第二阶段：离线回填（批处理）+ 校验一致性。
3. 第三阶段：读切换到新列（保留旧列）。
4. 第四阶段：加约束（NOT NULL/索引）+ 删除旧列（最后做）。

**示例 4：把脚本型代码整理成可发布包**

1. 抽模块：把核心逻辑变成库函数（不改行为）。
2. 补测试：补齐单元测试与样例数据。
3. 加 CLI：薄封装参数解析与调用库函数。
4. 文档化：补 README、用法与发布流程。

### Iterative refinement（迭代式收敛）

你不需要一次写出“完美提示词”。更稳的方法是：从广到窄，逐轮增加约束，直到输出可用。

**例 1：从“怎么做”到“按我项目的方式做”**

```text
（第 1 轮）解释一下这段代码为什么会 N+1 查询？
```

```text
（第 2 轮）请基于 @file src/db/ordersRepo.ts，指出具体是哪一行触发 N+1，并给出不改变返回结构的优化方案。
```

```text
（第 3 轮）约束：我们用 Postgres；不能新增依赖；必须有索引建议与 EXPLAIN 验证步骤。请给出最小 diff。
```

**例 2：从“生成代码”到“生成可审查的变更”**

```text
（第 1 轮）帮我加一个导出 CSV 的功能。
```

```text
（第 2 轮）请先输出设计：影响文件清单、API 契约、错误处理、以及测试策略；我确认后再生成代码。
```

**例 3：从“修复”到“防回归”**

```text
（第 1 轮）修复这个报错。
```

```text
（第 2 轮）请补一个回归测试覆盖该报错路径，并解释为什么测试能防止同类问题再次出现。
```

### Specifying constraints（声明约束）

下面是一些“可以直接粘贴”的约束短句，你可以按需组合：

- 语言/运行时：
  - “使用 Python 3.11 语法与标准库；不要依赖 3.12 才有的特性。”
  - “目标运行时是 Node.js 20；不要使用实验特性。”
- 框架/版本：
  - “使用 React 18 + Hooks；不要使用 class component。”
  - “使用 FastAPI；数据校验用现有的 Pydantic 模型，不要引入新库。”
- 代码风格：
  - “遵循 Airbnb JavaScript style guide；保持现有项目的 import 排序与命名。”
  - “TypeScript：不要用 `any`；优先精确类型与类型守卫。”
- 性能：
  - “目标 P95 延迟 < 200ms；避免 O(n^2)；批量化 IO。”
  - “最大数据量 500 万；避免一次性把数据读入内存。”
- 安全：
  - “所有外部输入必须校验；不要拼接 SQL；不要把敏感信息写入日志。”
  - “鉴权必须在服务端完成；不要相信前端传来的角色字段。”

> **Tip:** 约束最好写成“可验证”的句子（例如“必须通过 `pnpm test`”），而不是“写得优雅一点”。

### Real-world example: 修复 SQL 注入风险（Backend/Security）

**场景**：一个查询接口把用户输入直接拼到 SQL 里，存在 SQL 注入风险，同时错误处理也可能泄露内部细节。

**Before（危险：字符串拼接 SQL）**

```ts
import { pool } from "./db";

export async function getUserByEmail(req, res) {
  const email = req.query.email; // 未校验
  const result = await pool.query(
    `SELECT * FROM users WHERE email = '${email}'` // SQL 注入风险
  );
  return res.json(result.rows[0] ?? null);
}
```

**After（更安全：参数化查询 + 输入校验 + 最小返回字段）**

```ts
import { pool } from "./db";

export async function getUserByEmail(req, res) {
  const email = String(req.query.email ?? "").trim();
  if (!email) return res.status(400).json({ error: "email is required" });

  const result = await pool.query(
    "SELECT id, email, name FROM users WHERE email = $1 LIMIT 1",
    [email]
  );

  return res.json(result.rows[0] ?? null);
}
```

> ⚠️ **Warning:** “快速修复能跑”并不等于“安全”。当提示词涉及数据库、命令执行、模板渲染、鉴权时，你应当明确要求 AI 优先考虑注入、越权与敏感信息泄露。

你可以这样提示 Cursor：

```text
请对 @file src/routes/users.ts 的这个查询接口做安全修复：防 SQL 注入、补输入校验、限制返回字段、不要泄露内部错误。输出：最小 diff + 建议补的测试用例（至少 3 个：正常、空参数、注入尝试）。
```

---

<a id="context-management"></a>
## Context Management（上下文管理）

### When to start a new chat（何时新开对话）

- 适合 **继续同一对话** 的情况：
  - 你在同一个 bug 上迭代（补日志 → 定位 → 修复 → 加测试）。
  - 你在同一个 PR 上做审查（多轮问答围绕同一组改动）。
- 适合 **新开对话** 的情况：
  - 主题切换（从“修 bug”跳到“设计新模块”）。
  - 你发现 AI 已经带着错误前提在回答（越聊越偏）。
  - 你需要用全新上下文重新引用文件/目录（避免旧上下文干扰）。

### How to reset context（AI 迷糊时如何重置）

1. 用一句话重申目标（不要超过 2 行）。
2. 明确列出“事实”与“已验证信息”（例如：哪个函数确实会被调用、哪个字段确实为 `null`）。
3. 重新引用关键上下文（`@file`/`@folder`/日志）。
4. 要求它先复述你的问题与约束，再给方案（这是强力纠偏手段）。

可复制的“纠偏提示词”：

```text
你刚才的结论基于错误前提。请先复述：1）我真正的问题是什么；2）哪些约束必须遵守；3）你将使用哪些文件/日志作为证据。确认后再给修复方案与最小 diff。
```

### Token/length management（管理上下文长度与上限）

- 只喂“与当前问题强相关”的文件与日志：宁可少而精准，也不要把整仓库塞进去。
- 对长日志先做结构化摘要：
  - “失败发生在第 2 次重试”
  - “关键异常是 `ECONNRESET`”
  - “上游返回 429，触发指数退避”
- 大改动要求分批：
  - “先只改核心逻辑并通过测试”
  - “再做重构/清理”

### Composer vs inline chat vs terminal chat（何时用哪一种）

| 方式 | 适合任务 | 你应当要求的输出 | 常见风险 |
| --- | --- | --- | --- |
| Inline Edit（就地编辑） | 小范围改动、局部重写、函数内重构 | 直接改选区；不改别处 | 误改边界条件、遗漏调用方 |
| Chat（对话） | 解释、排查、给方案、评审 | 根因/权衡/下一步 | 输出泛泛、缺少可执行 diff |
| Composer/Agent（多文件） | 新功能、迁移、跨文件重构 | 计划→变更清单→逐文件 diff→验证步骤 | 改动范围过大、引入破坏性改动 |
| Terminal Chat（终端） | 构建/测试失败修复、脚本生成 | 命令 + 输出解释 + 修复方案 | 给出破坏性命令、忽略环境差异 |

---

<a id="troubleshooting"></a>
## Troubleshooting（排错指南）

这一节覆盖常见痛点：代码不对、上下文不懂、回答太泛、建议破坏性改动、大仓库性能问题。

### AI provides incorrect/outdated code（AI 代码不正确或过时）

- 让它声明“依据”与“假设”：
  - “请指出你依据的是哪一份文件/接口契约/版本假设。”
- 把事实喂给它：
  - 粘贴真实错误输出与版本号（例如 `node -v`、`python --version`）。
- 要求“最小可验证示例”：
  - “请给一个最小复现与修复后的最小验证脚本。”

### AI doesn't understand codebase context（AI 不理解代码库上下文）

1. 引用入口文件与路由/依赖注入点（告诉它“从哪里进来”）。
2. 引用真实调用链（上游/下游各 1 个文件）。
3. 给目录结构摘要（只列 1-2 层）。

### Responses are too generic（回答过于泛泛）

- 用结构化输出强制它具体：
  - “请按：问题→根因→修改文件→具体 diff→验证命令 输出。”
- 要求它“只给 1 个最佳方案”，并说明为何不选其他方案。
- 要求它把每条建议绑定到“哪一行/哪个函数/哪个文件”。

### AI suggests breaking changes（AI 建议破坏性改动）

> ⚠️ **Warning:** 一旦涉及“改 public API、改数据库 schema、批量替换、删除文件”，你应该把任务切成多步，并要求迁移与回滚策略。

- 让它输出影响面：
  - “受影响的调用方有哪些？请用搜索结果/引用文件列表证明。”
- 要求“兼容层/迁移期”：
  - “先加新接口并保持旧接口可用；给 deprecation 计划。”

### Performance issues with large codebases（大代码库性能/响应慢）

- 缩小上下文：只引用与当前任务相关的目录。
- 分批改动：把多文件任务拆成 2-4 个可独立验证的子任务。
- 用规则限制范围：在 `.cursorrules` 写清楚“不要跨域改动”“不要碰某些目录”。

### Real-world example: 修复慢 Pandas 处理（数据科学）

**场景**：你在 notebook 里写的处理逻辑迁到脚本后跑得很慢，怀疑是 `apply`。

**Before**

```python
import pandas as pd

def normalize_amount(x):
    if pd.isna(x):
        return 0
    return float(str(x).replace(",", ""))

df["amount_norm"] = df["amount"].apply(normalize_amount)
```

**After（向量化，减少 Python 层循环）**

```python
import pandas as pd

amount = df["amount"].astype("string")
df["amount_norm"] = (
    amount.fillna("0")
    .str.replace(",", "", regex=False)
    .astype("float64")
)
```

提示词建议写法：

```text
请把这段 Pandas 处理优化到向量化实现，输入规模 500 万行，目标是减少 Python 层 apply。给出 before/after，并说明复杂度与内存影响。
```

---
