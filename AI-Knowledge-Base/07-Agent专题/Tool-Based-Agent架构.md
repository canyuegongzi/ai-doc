# Tool-Based-Agent架构

> 状态：第一版草案

## 1. 文档定位

本文档用于定义 Tool-Based Agent 架构，重点回答：

- 当模型开始自主选择工具时，架构应如何变化
- Tool-Based Agent 与 Workflow-Agent 的边界和适用场景是什么
- 近两年为什么行业开始更强调 tool quality、tool safeguards 和 tool documentation

## 2. 什么是 Tool-Based Agent

Tool-Based Agent 指的是：

由模型在任务执行过程中动态决定是否调用工具、调用哪些工具、以什么顺序调用，并基于工具返回结果继续决策和推进任务的 agent 形态。

用一句话概括：

Tool-Based Agent 的本质，是把工具使用权部分交给模型，同时通过 runtime 与 guardrails 维持系统边界。

## 3. 为什么 Tool-Based Agent 重要

很多任务如果没有动态工具选择，很难兼顾：

- 多 source 查询
- 路径灵活性
- 异常分支处理
- 按结果调整后续动作

例如：

- 先查规则，再查日志，再查页面状态
- 如果日志为空，就换查构建记录
- 如果规则不明确，再去查案例库

这类任务很难完全写死流程，但仍然需要受控执行。

## 4. 近两年的关键变化

### 4.1 tool quality 正在被证明是 agent 质量核心变量

Anthropic 在 2025 年关于 tools 的文章里明确强调：

工具设计、参数描述、边界文档、示例和防错设计，直接决定模型工具使用效果。

这说明 Tool-Based Agent 的成败，并不主要在“Agent prompt 多聪明”，而在“Tool 设计是否专业”。

### 4.2 tool use 已从外围能力变成主流模型能力

OpenAI、Anthropic、Gemini 等最新模型都越来越把 tool use 作为模型原生能力之一。

这意味着 Tool-Based Agent 正在从“框架技巧”变成模型能力与系统架构共同驱动的主流形态。

### 4.3 tool safeguards 成为正式治理要求

OpenAI 的 practical agent guide 已明确把工具风险评级、读写权限、可逆性和 financial impact 等因素纳入 tool safeguard 设计。

因此，Tool-Based Agent 已不可能脱离治理体系存在。

## 5. Tool-Based Agent 解决什么问题

### 5.1 动态路径选择问题

任务路径不固定，需要按中间结果决定下一步。

### 5.2 多工具协作问题

同一任务可能需要多个工具组合完成。

### 5.3 复杂任务灵活性问题

相比完全固定 workflow，Tool-Based Agent 能更灵活地应对：

- 数据缺失
- 路径分叉
- 工具失败
- 多 source 组合

## 6. Tool-Based Agent 的核心组成

### 6.1 Tool Catalog

系统中可用工具集合。

### 6.2 Tool Selection Logic

由模型或 planner 决定当前是否调用工具。

### 6.3 Tool Invocation Contract

定义：

- 输入 schema
- 输出 schema
- 权限边界
- 风险等级
- 超时和幂等策略

### 6.4 Observation Loop

工具调用后读取结果并继续决策。

### 6.5 Runtime Control

限制：

- 最大步数
- 高风险工具审批
- 重试次数
- 失败升级策略

## 7. Tool-Based Agent 与 Workflow-Agent 的区别

Workflow-Agent：

- 主要流程显式
- 工具调用通常嵌在固定节点里

Tool-Based Agent：

- 模型拥有更高的 tool routing 自主性
- 更适合路径不完全确定的任务

换句话说：

Workflow-Agent 更重流程骨架，Tool-Based Agent 更重动态动作选择。

## 8. 为什么 Tool 文档与 Tool Schema 特别关键

模型用工具不像程序员那样看源码，它主要依赖：

- 名称
- 描述
- 参数 schema
- 示例
- 边界提示

因此，Tool-Based Agent 的高质量实现往往要求：

- 工具名足够清晰
- 参数语义明确
- 相似工具边界清楚
- 示例足够好
- 输入尽量防错化

这正是 Anthropic 在 tools 实战里反复强调的重点。

## 9. Tool-Based Agent 的典型运行循环

通常可表示为：

1. 理解任务
2. 判断是否需要工具
3. 选择工具
4. 生成参数
5. 执行工具
6. 读取结果
7. 判断是否继续
8. 完成、失败或升级人工

这和简单“一个 prompt + 一个 tool call”完全不是同一个复杂度层级。

## 10. Tool-Based Agent 的适用场景

适用于：

- 多源诊断
- 运维分析
- 开发辅助查询
- 复杂数据收集与汇总
- 工具链式协作任务

不太适合：

- 高风险强约束业务流程作为第一阶段落地
- 路径完全固定、规则明确的任务

## 11. 面向低代码平台的典型场景

### 11.1 配置纠错

可能需要动态调用：

- `getPageInfo`
- `validateSchema`
- `searchErrorCase`
- `searchComponentRule`

### 11.2 运维分析

可能需要动态调用：

- `getBuildLog`
- `getReleaseRecord`
- `searchOpsRule`
- `diffConfig`

### 11.3 页面辅助生成

可能需要动态调用：

- `searchTemplate`
- `searchComponent`
- `validateSchema`
- `createDraft`

## 12. 治理重点

Tool-Based Agent 一定要重点关注：

- tool white-list
- risk tiering
- read/write separation
- timeout and retry policy
- trace and replay
- human approval for high-risk actions

没有这些边界，动态工具选择很快会变成不可控风险点。

## 13. 常见误区

### 13.1 认为有 function calling 就等于 Tool-Based Agent

真正的 Tool-Based Agent 还需要 runtime、state、policy 和 guardrails。

### 13.2 工具设计得很随意

这会让模型在选择和填参阶段不断出错。

### 13.3 不区分读工具和写工具风险

这在企业场景中非常危险。

### 13.4 没有 observation / stop control

系统容易陷入无意义循环或重复调用。

## 14. 一句话总结

Tool-Based Agent 架构的核心，是让模型在受控边界内动态选择和调用工具，以解决固定工作流难以覆盖的复杂任务；近两年的最佳实践也越来越明确，Tool-Based Agent 的上限不只由模型决定，更由工具设计质量、运行时控制和治理边界共同决定。

## 15. 参考资料

- Anthropic, Writing effective tools for agents: [https://www.anthropic.com/engineering/writing-tools-for-agents](https://www.anthropic.com/engineering/writing-tools-for-agents)
- Anthropic, Building Effective AI Agents: [https://www.anthropic.com/engineering/building-effective-agents](https://www.anthropic.com/engineering/building-effective-agents)
- OpenAI, A practical guide to building agents: [https://openai.com/business/guides-and-resources/a-practical-guide-to-building-ai-agents/](https://openai.com/business/guides-and-resources/a-practical-guide-to-building-ai-agents/)
- OpenAI, New tools and features in the Responses API: [https://openai.com/index/new-tools-and-features-in-the-responses-api/](https://openai.com/index/new-tools-and-features-in-the-responses-api/)
