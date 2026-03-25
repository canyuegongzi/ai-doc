# Tool注册中心

> 状态：第一版草案

## 1. 文档定位

本文档用于定义 Tool Registry / Tool Catalog 的设计，重点回答：

- 为什么企业需要统一的 Tool 注册中心，而不是让每个 Agent 私自绑定工具
- 注册中心应承担哪些平台职责
- 近两年为什么 MCP、tool catalogs、tool discovery 正在成为更正式的系统能力

## 2. 什么是 Tool 注册中心

Tool 注册中心可以理解为：

统一管理系统内所有可被模型、Workflow、Agent Runtime 调用工具的目录、元数据、版本、权限、运行策略和发现能力的平台组件。

它不是简单的“工具列表”，而是：

- Tool Catalog
- Tool Metadata Store
- Tool Discovery Service
- Tool Governance Entry

的组合体。

用一句话概括：

Tool 注册中心的本质，是 Agent 时代的动作能力目录与治理控制面。

## 3. 为什么企业需要 Tool 注册中心

### 3.1 工具一旦增多，就不能靠代码硬编码维护

早期 Demo 往往直接在 Agent 代码里写死：

- 可用工具集合
- tool schema
- 调用入口
- prompt 描述

但当系统发展到几十上百个工具时，会立刻出现：

- 难以发现已有工具
- 相似工具边界冲突
- 工具版本失控
- 风险策略无法统一
- 多 Agent 之间重复维护

### 3.2 Tool Registry 是平台复用和治理的前提

如果没有统一注册中心：

- Workflow 不能复用同一套工具元数据
- Agent Runtime 无法统一做 routing 和 policy check
- 审计系统无法拿到稳定 tool identity
- replay/evals 难以绑定具体工具版本

### 3.3 近两年行业正把工具发现与协议标准化前置

MCP 的发展，以及 OpenAI / Anthropic / LangChain 生态越来越强调 tools catalog，说明工具不再只是“模型附带可调函数”，而是正在演进成平台级标准对象。

## 4. Tool 注册中心的核心职责

### 4.1 Tool Catalog 管理

维护全量工具清单及其元数据。

### 4.2 Tool Discovery

按场景、权限、风险、能力类型向模型或 runtime 提供可用工具集合。

### 4.3 版本管理

记录工具版本、协议变更、兼容性和废弃状态。

### 4.4 治理策略挂载

关联：

- auth scope
- risk policy
- HITL policy
- retry policy
- timeout policy
- audit level

### 4.5 运行时接入信息管理

统一管理：

- handlerRef
- routeKey
- provider
- execution mode
- availability status

## 5. 注册中心应管理哪些元数据

建议至少管理以下维度：

### 5.1 标识信息

- toolId
- name
- displayName
- version
- ownerTeam
- lifecycleStatus

### 5.2 语义信息

- description
- category
- tags
- applicableScenes
- preconditions
- antiPatterns

### 5.3 契约信息

- inputSchema
- outputSchema
- errorSchema
- examples

### 5.4 治理信息

- riskLevel
- authScope
- requiresHumanApproval
- tenantIsolationMode
- auditLevel

### 5.5 运行信息

- timeoutMs
- retryPolicy
- idempotent
- executionMode
- provider
- routeKey

### 5.6 运营信息

- usageStats
- successRate
- avgLatency
- failureRate
- lastUpdatedAt

## 6. 注册中心应如何支持 Tool Discovery

注册中心不应只是让开发者查文档，也应支持运行时发现工具。

推荐支持以下发现方式：

### 6.1 按场景发现

例如：

- schema generation
- configuration diagnosis
- ops analysis
- page publish assistant

### 6.2 按风险等级发现

例如：

- 只提供 read-only tools
- 只提供 low / medium risk tools

### 6.3 按权限边界发现

根据当前用户、租户、应用、工作空间动态裁剪工具集合。

### 6.4 按任务阶段发现

在不同 runtime state 下暴露不同工具。

例如：

- planning 阶段可见检索类工具
- execution 阶段可见执行类工具
- review 阶段仅可见审批相关工具

## 7. 为什么 Tool 注册中心不应只做静态配置

现代 Agent 系统中的工具发现是动态的。

同一个工具是否可见，取决于：

- 当前场景
- 用户身份
- 环境
- 风险等级
- 工具健康状态
- 策略开关

因此，更合理的注册中心通常包含：

- 静态元数据存储
- 动态策略裁剪
- 运行时 availability check

## 8. Tool 注册中心与 Runtime 的关系

Runtime 通常依赖注册中心完成：

- tool lookup
- tool filtering
- schema retrieval
- governance policy lookup
- version binding
- execution routing

也就是说：

注册中心是“工具是什么”的来源，runtime 是“工具怎么被执行”的地方。

## 9. Tool 注册中心与 Prompt / Model 的关系

在工具使用场景中，并不是所有注册工具都会直接暴露给模型。

更合理的做法是：

1. runtime 先从 registry 里取候选工具
2. 再根据当前任务、状态和权限做裁剪
3. 只把必要工具协议传给模型

这样能显著降低：

- 选择混乱
- 上下文膨胀
- 高风险工具误暴露

## 10. Tool 注册中心与版本治理

一个成熟注册中心至少应支持：

- tool version
- schema compatibility
- deprecation status
- replacement mapping
- run-to-version binding

否则后续：

- replay 无法稳定还原
- eval 样本无法对齐
- prompt 中的工具描述会漂移

## 11. 面向低代码平台的注册中心设计建议

建议按平台对象组织工具目录，例如：

- 页面类工具
- 组件类工具
- schema 类工具
- 数据源类工具
- 运维类工具
- 发布类工具

同时对每类工具增加：

- 适用平台场景
- 是否生产敏感
- 是否需要人工审批
- 是否支持自动修复
- 依赖哪些后端系统

例如：

- `validatePageSchema`
- `searchComponentMaterial`
- `getPublishHistory`
- `createPageDraft`
- `checkReleaseRisk`

## 12. Tool 注册中心应暴露哪些接口

建议至少包含：

- `registerTool`
- `updateTool`
- `deprecateTool`
- `listTools`
- `searchTools`
- `getToolByName`
- `resolveAvailableTools`
- `getToolPolicy`
- `getToolSchema`

在企业内部，也可以进一步提供：

- `getToolMetrics`
- `listDeprecatedTools`
- `findToolAlternatives`

## 13. 常见误区

### 13.1 把注册中心做成一张静态表

这样无法支撑 discovery、动态裁剪和治理策略。

### 13.2 让每个 Agent 单独维护工具清单

很快会造成工具元数据碎片化和重复维护。

### 13.3 不记录 tool version

后续 replay、审计、评测都难以稳定对齐。

### 13.4 注册中心只管协议，不管运行状态

如果不感知 availability 和健康度，runtime 很难做稳定路由。

### 13.5 所有工具都直接暴露给模型

这会显著增加工具选择难度和风险暴露面。

## 14. 一句话总结

Tool 注册中心的核心，是把企业内不断增长的工具集合组织成可发现、可治理、可版本化、可按场景动态裁剪的动作目录系统；近两年的最佳实践也越来越明确，Agent 系统要做平台化，必须先把 Tool Catalog 做成正式基础设施。

## 15. 参考资料

- Model Context Protocol, overview: [https://modelcontextprotocol.io/introduction](https://modelcontextprotocol.io/introduction)
- Model Context Protocol, servers concept: [https://modelcontextprotocol.io/docs/concepts/servers](https://modelcontextprotocol.io/docs/concepts/servers)
- Anthropic, Building Effective AI Agents: [https://www.anthropic.com/engineering/building-effective-agents](https://www.anthropic.com/engineering/building-effective-agents)
- Anthropic, Writing effective tools for agents: [https://www.anthropic.com/engineering/writing-tools-for-agents](https://www.anthropic.com/engineering/writing-tools-for-agents)
- OpenAI Responses API tools guide: [https://platform.openai.com/docs/guides/tools](https://platform.openai.com/docs/guides/tools)
