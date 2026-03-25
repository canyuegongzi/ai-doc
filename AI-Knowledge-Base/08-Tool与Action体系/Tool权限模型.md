# Tool权限模型

> 状态：第一版草案

## 1. 文档定位

本文档用于定义 Agent / Workflow 系统中的 Tool Authorization Model，重点回答：

- 为什么 Tool 权限不能只依赖后端接口原有权限
- Tool 级权限、数据级权限、执行级权限应如何分层设计
- 近两年为什么行业越来越强调 least privilege、read/write separation 和 policy-aware tool use

## 2. 为什么 Tool 权限模型是基础设施

在传统系统里，权限控制往往发生在：

- 页面菜单
- API 网关
- 后端服务
- 数据库访问层

但在 Agent 系统里，模型会把自然语言目标转成工具调用意图，这带来了新的问题：

- 模型是否看到了不该看到的工具
- 模型是否能替用户调用超出权限边界的动作
- 同一个工具是否能在不同环境、不同租户下执行不同范围的动作
- 是否允许模型直接触发写操作

这说明 Tool 权限模型不能只是沿用传统接口权限，而必须成为 AI Runtime 的正式控制层。

用一句话概括：

Tool 权限模型的本质，是把“谁能调用什么工具、在什么条件下调用、可以作用于哪些对象”显式建模成运行时可执行策略。

## 3. Tool 权限模型要解决什么问题

### 3.1 工具可见性控制

不是所有工具都应暴露给所有用户、所有任务、所有 Agent。

### 3.2 工具调用授权

即使工具可见，也不意味着当前上下文一定允许执行。

### 3.3 数据作用范围控制

调用工具时，需要限制其作用对象，例如：

- 当前租户
- 当前应用
- 当前页面
- 当前工作空间

### 3.4 高风险动作拦截

对高风险工具或高风险参数，应触发更严格的权限或 HITL 机制。

## 4. 近两年的关键变化

### 4.1 tool use 正从“能力开放”转向“能力约束”

随着模型原生 function calling / tool use 能力成熟，行业最佳实践越来越强调：

把工具暴露给模型并不难，难的是正确裁剪工具边界。

### 4.2 least privilege 在 Agent 场景里更加重要

生产环境下的 Agent 通常应遵守：

- 默认最小权限
- 默认只读优先
- 写操作需要更强校验
- 高风险动作需要审批或双重确认

### 4.3 权限判断正在从 API 层前移到 Tool Discovery 层

更成熟的系统会在工具发现阶段就先裁剪：

- 当前身份不应看到的工具
- 当前场景不应暴露的工具
- 当前状态不应开放的写工具

这比等到调用时再拒绝更稳定。

## 5. Tool 权限模型的推荐分层

建议至少分成四层：

### 5.1 可见性权限

决定当前模型 / Agent 是否能看到这个工具。

### 5.2 调用权限

决定当前上下文是否允许发起这次调用。

### 5.3 数据权限

决定工具调用时可作用于哪些资源。

### 5.4 执行权限

决定这次调用能否真正落地执行，而不是只允许 dry-run、risk-check 或草稿生成。

## 6. 推荐的权限判断维度

Tool 权限建议至少考虑以下维度：

### 6.1 用户身份维度

- userId
- role
- department
- permission set
- operator type

### 6.2 组织边界维度

- tenantId
- workspaceId
- appId
- environment

### 6.3 工具维度

- tool name
- tool category
- risk level
- read/write type
- approval requirement

### 6.4 输入参数维度

- target resource
- targetEnvironment
- delete / overwrite flags
- cross-scope identifiers

### 6.5 运行时维度

- 当前任务类型
- 当前状态机节点
- 当前是否人工接管
- 当前是否已有审批令牌

## 7. 推荐的授权链路

更稳的授权链路通常不是“执行前单点判断”，而是多段式：

1. Tool Registry 按身份和场景裁剪可见工具
2. Planner / Runtime 基于任务状态筛选候选工具
3. 调用前进行参数级授权判断
4. 对高风险动作触发 HITL / approval
5. 执行层再次做后端强校验

换句话说：

Tool 权限模型不应替代后端权限，而应在 AI 运行时前置一层更细颗粒度的动作控制。

## 8. 常见权限策略

### 8.1 默认只读策略

在没有明确授权时，只暴露 read-only tools。

### 8.2 写工具显式授权策略

执行类工具必须额外满足：

- 角色允许
- 参数合法
- 环境允许
- 风险通过

### 8.3 环境隔离策略

例如：

- dev/test 可自动执行
- prod 必须审批

### 8.4 资源作用域策略

例如只允许对：

- 当前租户资源
- 当前应用资源
- 当前页面资源

发起动作。

### 8.5 时间或频率策略

例如：

- 某些工具在发布窗口外不可执行
- 某些批量动作需要限频或限额

## 9. Tool 权限模型与 Schema 的关系

权限不只是看工具名，也应看参数。

例如同一个 `publishPage` 工具：

- `targetEnvironment=test` 可能允许自动执行
- `targetEnvironment=prod` 可能必须 HITL

再例如：

- `deleteMode=soft` 与 `deleteMode=hard` 的权限要求不应相同

这说明参数 Schema 与权限模型必须协同设计。

## 10. Tool 权限模型与 Risk / HITL 的关系

### 10.1 与 Risk 分级的关系

风险等级越高，权限校验越严格。

### 10.2 与 HITL 的关系

权限模型应能定义：

- 哪类角色可直接执行
- 哪类角色需要审批
- 哪类角色只能发起建议、不能执行

### 10.3 与 Audit 的关系

每次授权决定都应被审计，尤其是：

- 权限放行
- 权限拒绝
- 人工 override
- 审批通过

## 11. 面向低代码平台的 Tool 权限设计建议

建议围绕平台对象和环境边界设计。

典型策略例如：

- `searchComponentMaterial`：租户内只读开放
- `validatePageSchema`：应用内开放
- `createPageDraft`：应用编辑者可执行
- `publishPage`：仅发布角色可发起，prod 需审批
- `deletePage`：高风险，仅管理员 + HITL
- `diffTenantConfig`：受限，仅运维角色可见

对于跨租户、跨环境、批量修改类工具，建议默认不开放给普通 Agent 自动执行。

## 12. 常见误区

### 12.1 只依赖后端 API 权限

这样无法解决模型侧工具暴露和任务态裁剪问题。

### 12.2 工具可见即默认可执行

可见性和执行权是两个层次。

### 12.3 不做参数级权限判断

很多高风险动作不是工具名决定的，而是参数决定的。

### 12.4 不区分 read 和 write

这会导致工具集合暴露过宽，风险上升很快。

### 12.5 不记录授权决策过程

后续审计和复盘会失去关键证据。

## 13. 一句话总结

Tool 权限模型的核心，是在 Agent Runtime 中把工具可见性、调用授权、数据范围和执行权限显式化，并与参数 Schema、风险分级、HITL 和审计体系联动；近两年的最佳实践也越来越明确，企业级 Tool 系统的成熟度，很大程度上取决于它是否真正做到了最小权限和分层授权。

## 14. 参考资料

- OpenAI, Building guardrails for agents: [https://openai.com/index/building-guardrails-for-agents/](https://openai.com/index/building-guardrails-for-agents/)
- OpenAI, A practical guide to building agents: [https://openai.com/business/guides-and-resources/a-practical-guide-to-building-ai-agents/](https://openai.com/business/guides-and-resources/a-practical-guide-to-building-ai-agents/)
- Anthropic, Building Effective AI Agents: [https://www.anthropic.com/engineering/building-effective-agents](https://www.anthropic.com/engineering/building-effective-agents)
- Anthropic, Writing effective tools for agents: [https://www.anthropic.com/engineering/writing-tools-for-agents](https://www.anthropic.com/engineering/writing-tools-for-agents)
