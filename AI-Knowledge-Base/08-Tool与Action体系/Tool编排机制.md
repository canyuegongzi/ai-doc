# Tool编排机制

> 状态：第一版草案

## 1. 文档定位

本文档用于定义 Tool Orchestration 机制，重点回答：

- 当系统拥有越来越多工具后，应该如何组织它们协同执行
- Tool 编排与 Planner、Workflow、Runtime、Registry 的边界是什么
- 近两年为什么行业开始从单次 tool call 走向更完整的 action orchestration

## 2. 什么是 Tool 编排

Tool 编排可以理解为：

在一个任务执行过程中，对多个工具的选择、顺序、依赖、条件分支、错误恢复和结果汇总进行系统化组织的机制。

它关注的不是“某个工具怎么调”，而是：

- 什么时候该调哪个工具
- 哪些工具可以串行
- 哪些工具可以并行
- 哪些结果会成为后续输入
- 失败后该重试、换路还是升级人工

用一句话概括：

Tool 编排的本质，是把离散工具调用组织成可控的任务执行链路。

## 3. 为什么需要 Tool 编排

### 3.1 企业任务通常不是单工具可完成的

很多真实任务都会涉及多个工具协作，例如：

- 先查知识规则
- 再查目标页面状态
- 再校验 Schema
- 再生成修复方案
- 最后创建草稿或提交审批

如果没有编排机制，系统就容易变成：

- 工具调用顺序混乱
- 重复查询
- 上下文断裂
- 错误恢复无规则

### 3.2 工具数量增加后，必须从“函数调用”升级为“动作链管理”

一旦系统拥有几十到上百个工具：

- 仅靠模型自由选择会显著增加不稳定性
- 仅靠静态 Workflow 又会失去灵活性

因此，更成熟的系统通常会采用：

- Workflow 骨架
- Planner 局部决策
- Tool 编排层做执行协调

### 3.3 近两年的 runtime 设计正在强化 orchestration

无论是 LangGraph 的 state graph，还是 OpenAI / Anthropic 对 agent system 的实践，都越来越强调：

真正的生产系统需要显式编排，而不是“模型不断输出 tool call”。

## 4. Tool 编排要解决什么问题

### 4.1 顺序控制

决定哪些工具必须先后执行。

### 4.2 依赖传递

让上一步结果稳定成为下一步输入。

### 4.3 并行优化

对可独立工具并发执行，降低总体延迟。

### 4.4 分支与回退

根据中间结果选择不同路径。

### 4.5 失败恢复

在工具失败时决定：

- retry
- fallback
- alternative tool
- human review

### 4.6 结果汇总

将多工具结果结构化合并，供 Planner、Generator 或用户消费。

## 5. 近两年的关键变化

### 5.1 从“单次 tool use”走向“多步 action orchestration”

早期很多系统只关注模型能否正确调一个工具；而现在的重点已经转向：

- 多工具链式执行
- retrieval + tools 混合链路
- 带状态的执行图
- 受控的长运行任务

### 5.2 编排正在从 prompt 内隐逻辑转向显式 runtime 逻辑

过去很多系统把编排规则写在 prompt 里；现在更成熟的做法是：

- prompt 负责局部决策提示
- runtime 负责图结构、状态、分支和错误处理

### 5.3 工具编排越来越强调 typed state 和 structured handoff

也就是说：

工具之间不再只是“文本传文本”，而是通过结构化中间状态和结果对象衔接。

## 6. 推荐的 Tool 编排模式

### 6.1 串行编排

最常见，适合存在明显前后依赖的任务。

例如：

- `getPageInfo -> validatePageSchema -> generatePatch`

### 6.2 并行编排

适合多个信息源可独立查询的场景。

例如：

- 同时查询发布记录、构建日志、配置差异

### 6.3 条件分支编排

根据工具结果选择后续路径。

例如：

- 校验通过则继续发布
- 校验失败则进入修复链路

### 6.4 Router + Specialist 编排

先决定任务路由，再进入不同的工具子链路。

### 6.5 Planner-driven 编排

由 Planner 决定下一步工具，但执行顺序、状态流转仍由 Runtime 控制。

## 7. Tool 编排与 Planner 的关系

Planner 负责：

- 决定当前下一步做什么
- 选择路径或工具类别

Tool 编排负责：

- 执行具体工具链路
- 管理依赖、状态、重试、分支

换句话说：

Planner 更像决策器，编排层更像执行协调器。

## 8. Tool 编排与 Workflow 的关系

Workflow 提供的是更高层的流程骨架。

Tool 编排则更聚焦在某个节点或某段执行链内，如何组织多个工具协作。

可以理解为：

- Workflow 是任务级编排
- Tool Orchestration 是动作级编排

两者通常不是替代关系，而是嵌套关系。

## 9. 推荐的编排输出 / 中间态设计

建议每一步编排结果都写入结构化中间状态，例如：

- stepId
- toolName
- inputSummary
- outputRef
- status
- retryCount
- latency
- riskFlags
- nextCandidates

这样做的价值在于：

- 便于 replay
- 便于审计
- 便于 planner 再决策
- 便于并行步骤汇总

## 10. 编排层应支持哪些运行能力

建议至少支持：

### 10.1 依赖图表达

显式表达哪些步骤依赖前置结果。

### 10.2 并行执行控制

控制并发上限、超时和结果收敛。

### 10.3 条件分支

根据结果选择不同后续节点。

### 10.4 fallback 与替代工具

某个工具不可用时，允许切到替代工具或退化路径。

### 10.5 审计与回放挂钩

每次编排决策和工具执行都可追踪。

## 11. 面向低代码平台的典型编排示例

### 11.1 配置纠错链路

可能的工具链：

- `getPageInfo`
- `searchComponentRule`
- `validatePageSchema`
- `searchErrorCase`
- `generateSchemaPatch`
- `createPageDraft`

其中：

- 规则搜索和案例搜索可并行
- patch 生成依赖校验结果
- draft 创建前可能需 HITL

### 11.2 页面生成链路

可能的工具链：

- `searchPageTemplate`
- `searchComponentMaterial`
- `getDataModelSpec`
- `generatePageSchema`
- `validatePageSchema`
- `createPageDraft`

### 11.3 运维分析链路

可能的工具链：

- `getReleaseHistory`
- `getBuildLogSummary`
- `diffTenantConfig`
- `searchOpsRule`
- `generateRootCauseSummary`

其中多个查询步骤可并行。

## 12. 常见误区

### 12.1 把工具编排完全交给模型自由决定

这在复杂任务中很容易导致重复、遗漏和不稳定路径。

### 12.2 把所有链路写死成静态 Workflow

会牺牲对中间结果的动态响应能力。

### 12.3 中间状态不结构化

后续汇总、回放、评测都会变得困难。

### 12.4 不做并行收敛设计

会导致多源工具调用结果难以统一消费。

### 12.5 编排层不感知风险和权限

会让高风险动作在错误分支里被意外执行。

## 13. 一句话总结

Tool 编排机制的核心，是把多个离散工具调用组织成有依赖、有分支、有恢复能力的动作链路，并通过 Runtime 与 Planner 协同把任务执行稳定下来；近两年的最佳实践也越来越明确，真正可生产的 Agent 系统依赖的不是“模型会调多少工具”，而是“系统能否把这些工具编排成可靠流程”。

## 14. 参考资料

- LangGraph overview: [https://docs.langchain.com/oss/python/langgraph/overview](https://docs.langchain.com/oss/python/langgraph/overview)
- LangGraph durable execution: [https://docs.langchain.com/oss/python/langgraph/durable-execution](https://docs.langchain.com/oss/python/langgraph/durable-execution)
- Anthropic, Building Effective AI Agents: [https://www.anthropic.com/engineering/building-effective-agents](https://www.anthropic.com/engineering/building-effective-agents)
- OpenAI, A practical guide to building agents: [https://openai.com/business/guides-and-resources/a-practical-guide-to-building-ai-agents/](https://openai.com/business/guides-and-resources/a-practical-guide-to-building-ai-agents/)
- LlamaIndex workflows: [https://docs.llamaindex.ai/en/stable/module_guides/workflow/](https://docs.llamaindex.ai/en/stable/module_guides/workflow/)
