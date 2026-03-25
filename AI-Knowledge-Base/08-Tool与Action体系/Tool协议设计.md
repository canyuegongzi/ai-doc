# Tool协议设计

> 状态：第一版草案

## 1. 文档定位

本文档用于定义企业 AI 系统中的 Tool Protocol，重点回答：

- Tool 为什么不能只被理解成“给模型暴露一个函数”
- 生产级 Tool 应具备哪些正式协议字段
- 近两年行业为什么越来越强调 tool design、tool documentation、tool safeguards

## 2. 什么是 Tool 协议

Tool 协议可以理解为：

对一个可被模型、Workflow 或 Agent Runtime 调用的外部能力进行标准化描述与约束的一套契约。

它不仅描述“可以调用什么”，还描述：

- 这个工具是干什么的
- 适合在什么条件下调用
- 需要什么输入
- 会返回什么结果
- 有什么风险边界
- 权限和审计要求是什么

用一句话概括：

Tool 协议的本质，是把系统动作从“代码函数”提升为“可被 AI 稳定理解、可被运行时治理的标准能力对象”。

## 3. 为什么 Tool 协议是 Agent 系统的关键基础设施

### 3.1 模型主要依赖协议和描述理解工具，而不是源码

模型选择工具时，依赖的通常不是函数内部实现，而是：

- tool name
- description
- input schema
- examples
- constraints
- risk hints

这意味着 Tool 设计质量，直接影响模型的工具选择质量。

### 3.2 工具协议决定了 Runtime 是否可治理

如果工具只是零散函数：

- 无法统一做权限控制
- 无法统一做风险分级
- 无法统一做审计
- 无法统一做 route、retry、timeout、idempotency 管理

因此，Tool 协议实际上是 Tool Runtime 的治理入口。

### 3.3 近两年行业已经形成“tool quality 决定 agent quality”的共识

Anthropic 在 2025 年关于 tools 的工程文章里非常明确地强调：

清晰的工具命名、精确的参数解释、示例、错误设计和边界提示，都会显著影响 agent 成功率。

这说明 Tool 协议已经不再是附属细节，而是核心系统设计对象。

## 4. Tool 协议需要解决什么问题

### 4.1 能力暴露标准化

让不同系统能力以统一方式暴露给模型和 runtime。

### 4.2 动作边界清晰化

让模型知道这个工具能做什么、不能做什么。

### 4.3 风险与权限可治理

让系统能够在调用前做：

- 权限判断
- 风险检查
- 审批决策
- 参数校验

### 4.4 复用与平台化

让同一个工具既能被：

- Workflow 使用
- Tool-Based Agent 使用
- 人工操作界面使用
- 测试和 replay 系统使用

## 5. 推荐的 Tool 协议核心字段

一个生产级 Tool 协议建议至少包含以下字段：

### 5.1 基础标识字段

- `name`
- `displayName`
- `description`
- `category`
- `version`

### 5.2 输入输出契约字段

- `inputSchema`
- `outputSchema`
- `examples`
- `errorSchema`

### 5.3 运行控制字段

- `timeoutMs`
- `retryPolicy`
- `idempotent`
- `sideEffectLevel`
- `maxConcurrency`

### 5.4 治理字段

- `riskLevel`
- `authScope`
- `tenantIsolationMode`
- `requiresHumanApproval`
- `auditLevel`

### 5.5 可发现性字段

- `tags`
- `applicableScenes`
- `preconditions`
- `antiPatterns`
- `toolHints`

### 5.6 执行接入字段

- `handlerRef`
- `provider`
- `routeKey`
- `availabilityPolicy`

## 6. 推荐的协议形态

一个简化示例：

```ts
interface AgentTool {
  name: string;
  description: string;
  category: 'query' | 'retrieve' | 'validate' | 'generate' | 'execute';
  version: string;
  inputSchema: JSONSchema;
  outputSchema?: JSONSchema;
  riskLevel: 'low' | 'medium' | 'high';
  authScope?: string[];
  idempotent?: boolean;
  timeoutMs?: number;
  requiresHumanApproval?: boolean;
  examples?: Array<{
    input: unknown;
    output?: unknown;
    note?: string;
  }>;
  handlerRef: string;
}
```

真正落地时，建议把协议字段再按：

- 发现层
- 调用层
- 治理层
- 运行层

做分层组织。

## 7. Tool 协议应如何分类

### 7.1 按动作性质分类

- query tool
- retrieval tool
- validation tool
- generation tool
- execution tool

### 7.2 按副作用等级分类

- read-only
- reversible write
- non-reversible write
- external side effect

### 7.3 按权限边界分类

- public-in-tenant
- app-scoped
- workspace-scoped
- admin-only
- cross-tenant restricted

### 7.4 按调用方式分类

- sync
- async job
- streaming
- long-running task

## 8. 工具描述应该如何写

### 8.1 名称要精确

避免：

- `processData`
- `handlePage`
- `doAction`

更合理的是：

- `getPageDraft`
- `validatePageSchema`
- `searchComponentSpec`
- `createPublishTicket`

### 8.2 描述要写清“何时用、何时不用”

描述不应只写“该工具用于 xxx”，还应写：

- 适用场景
- 输入前提
- 不适用场景
- 可能的风险

### 8.3 示例要贴近真实任务

示例不是锦上添花，而是帮助模型更稳定选对工具和填对参数的重要部分。

## 9. Tool 协议与 Runtime 的关系

Runtime 会基于协议做：

- tool discovery
- schema validation
- permission check
- approval gating
- timeout / retry
- trace / audit
- replay / mock execution

因此，协议越完整，runtime 的治理能力就越容易标准化。

## 10. Tool 协议与 Prompt / Model 的关系

现代 tool use 不只是把协议发给模型，还包括：

- 提示模型如何理解工具边界
- 给出选择工具的优先级
- 在 prompt 中写明读写区别与风险提示
- 用 structured outputs / function calling 承载调用意图

换句话说：

Tool 协议是数据契约，Prompt 是模型消费契约，两者共同决定工具使用效果。

## 11. 面向低代码平台的 Tool 协议设计建议

建议重点围绕平台对象抽象工具，而不是围绕底层服务细节直接暴露。

更合理的工具命名方式例如：

- `searchPageTemplate`
- `getComponentMaterialSpec`
- `validateSchemaBinding`
- `createPageDraft`
- `checkPublishRisk`
- `getBuildFailureSummary`

每个工具应明确：

- 服务哪个平台对象
- 是否只读
- 是否可跨租户
- 是否需要人工确认
- 是否允许自动修复

## 12. 常见误区

### 12.1 把 Tool 协议等同于函数签名

函数签名只解决调用问题，不解决治理问题。

### 12.2 工具名和描述含糊不清

会直接提高模型选错工具和填错参数的概率。

### 12.3 不区分只读工具和执行工具

会让高风险动作难以统一控制。

### 12.4 缺少版本信息

工具变更后，旧 prompt、旧 replay、旧 eval 样本都会变得不稳定。

### 12.5 不写 examples 和 anti-patterns

这会显著降低模型对工具边界的理解质量。

## 13. 一句话总结

Tool 协议设计的核心，是把企业系统中的外部能力抽象成可发现、可理解、可校验、可治理、可审计的标准动作对象；近两年的最佳实践也越来越明确，Agent 成功率不仅取决于模型，更取决于工具协议是否专业、清晰、稳定。

## 14. 参考资料

- Anthropic, Writing effective tools for agents: [https://www.anthropic.com/engineering/writing-tools-for-agents](https://www.anthropic.com/engineering/writing-tools-for-agents)
- Anthropic, Building Effective AI Agents: [https://www.anthropic.com/engineering/building-effective-agents](https://www.anthropic.com/engineering/building-effective-agents)
- OpenAI, New tools and features in the Responses API: [https://openai.com/index/new-tools-and-features-in-the-responses-api/](https://openai.com/index/new-tools-and-features-in-the-responses-api/)
- OpenAI Responses API tools guide: [https://platform.openai.com/docs/guides/tools](https://platform.openai.com/docs/guides/tools)
- Model Context Protocol, specification: [https://modelcontextprotocol.io/specification/2025-06-18](https://modelcontextprotocol.io/specification/2025-06-18)
