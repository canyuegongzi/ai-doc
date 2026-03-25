# Agent任务模型

> 状态：第一版草案

## 1. 文档定位

本文档用于定义 Agent 系统中的任务模型（task model），重点回答：

- Agent 如何把用户目标转化为可执行任务
- 为什么任务模型是 Runtime、Planner、Tool 和 Memory 的共同上游
- 近两年 agent 工程实践中，任务建模为什么变得越来越重要

## 2. 为什么 Agent 必须先有任务模型

如果没有清晰任务模型，Agent 往往会出现：

- 不知道当前到底要完成什么
- 工具调用和目标脱节
- 状态推进混乱
- 终止条件不清晰
- 人工确认点难定义

这说明 Agent 系统的第一性问题，不是“先调工具”，而是“先定义任务是什么”。

## 3. 近两年的关键变化

### 3.1 Agent 设计正在从 prompt-centric 走向 task-centric

Anthropic 和 OpenAI 的最新 agent 指南都在强调：

- 先选择适合 agent 的 use case
- 先明确任务边界
- 再设计 tool / orchestration / guardrails

这意味着行业正在形成共识：

任务模型比 prompt loop 更接近 agent 成败的第一控制点。

### 3.2 Runtime 和 long-running agents 放大了任务建模的重要性

一旦任务可以跨多个步骤、多个工具甚至跨会话运行，就必须显式知道：

- 任务类型
- 当前阶段
- 需要什么输入
- 完成条件是什么
- 什么情况下应中断或升级为人工处理

### 3.3 agentic retrieval 和 tool planning 让任务模型更结构化

当 retrieval、planning、execution 都变成多阶段过程时，任务模型不再只是“一个用户请求”，而更像一个结构化对象。

## 4. 什么是 Agent 任务模型

Agent 任务模型可以理解为：

系统对“当前要完成的工作”所做的结构化表示，包括目标、约束、上下文、阶段、依赖关系、成功标准和风险等级等信息。

用一句话概括：

任务模型是 Agent Runtime 理解“当前正在做什么”的核心对象。

## 5. 任务模型应包含哪些内容

建议至少包含以下要素：

- taskId
- taskType
- goal
- inputContext
- currentState
- requiredCapabilities
- constraints
- successCriteria
- failureCriteria
- riskLevel
- humanReviewPolicy

## 6. 常见任务类型分类

### 6.1 问答型任务

目标：给出解释、回答或总结。

### 6.2 搜索导航型任务

目标：找到对象、文档、模板或案例。

### 6.3 生成型任务

目标：输出结构化草稿或文本产物。

### 6.4 诊断型任务

目标：定位问题、解释原因、给出建议。

### 6.5 执行型任务

目标：在受控边界内完成查询、创建、触发等动作。

### 6.6 复合型任务

目标：由多个子任务组成，例如“先查再生成再校验”。

## 7. 任务模型中的关键维度

### 7.1 目标维度

描述最终想达成什么结果。

### 7.2 状态维度

描述当前执行到哪一步。

### 7.3 约束维度

描述：

- 时间限制
- 权限限制
- 风险边界
- 输出格式要求

### 7.4 能力维度

描述任务需要：

- retrieval
- tools
- reasoning
- multimodal input
- structured outputs

### 7.5 终止维度

描述：

- 任务成功条件
- 失败条件
- 最大步数
- 升级人工条件

## 8. 为什么任务模型应尽量结构化

因为一旦结构化，系统才能稳定做：

- 路由
- Workflow 选择
- Tool white-list
- 状态持久化
- Trace
- Evals
- Replay

如果任务模型只是“自然语言请求本身”，后续 runtime 能力会非常脆弱。

## 9. 任务模型与 Planner 的关系

任务模型定义“任务是什么”，Planner 决定“下一步怎么做”。

两者关系类似：

- task model = 问题空间与边界
- planner = 当前步骤决策器

## 10. 任务模型与 Memory 的关系

Memory 往往存的是：

- 会话历史
- 中间步骤
- 工具结果
- 人工反馈

任务模型则更像 Runtime 当前持有的“任务主对象”。

因此，任务模型通常是 memory 的核心组织骨架之一。

## 11. 任务模型与 RAG 的关系

任务类型会直接决定 retrieval 策略。

例如：

- FAQ 任务 -> 解释型 retrieval
- 生成任务 -> 模板 / 规则 retrieval
- 诊断任务 -> 案例 / 规则 / 日志 retrieval
- 执行任务 -> 规范 / 协议 retrieval

这说明 task model 是 retrieval routing 的重要上游。

## 12. 面向低代码平台的任务模型建议

在低代码平台中，建议显式建模以下任务类型：

- component_qa
- page_schema_generation
- config_diagnosis
- logic_flow_generation
- ops_analysis
- draft_creation

并为每类任务定义：

- 必需输入
- 默认 tool set
- 风险等级
- 输出结构
- 终止条件

## 13. 常见误区

### 13.1 把所有用户请求都当成同一种 agent task

### 13.2 任务模型只有自然语言，没有结构字段

### 13.3 不定义成功 / 失败 / 中断条件

### 13.4 任务模型不与工具和 retrieval 路由联动

## 14. 一句话总结

Agent 任务模型的核心，是把用户目标和系统约束转化为可被 Runtime、Planner、Tool、Memory 和 Retrieval 共同消费的结构化任务对象；近两年的最佳实践也越来越明确，任务建模是 agent 工程从“看起来能跑”走向“真正可控可运营”的第一步。

## 15. 参考资料

- Anthropic, Building Effective AI Agents: [https://www.anthropic.com/engineering/building-effective-agents](https://www.anthropic.com/engineering/building-effective-agents)
- OpenAI, A practical guide to building agents: [https://openai.com/business/guides-and-resources/a-practical-guide-to-building-ai-agents/](https://openai.com/business/guides-and-resources/a-practical-guide-to-building-ai-agents/)
- LangGraph overview: [https://docs.langchain.com/oss/python/langgraph/overview](https://docs.langchain.com/oss/python/langgraph/overview)
