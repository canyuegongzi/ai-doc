# 执行增强型RAG

> 状态：第一版草案

## 1. 文档定位

本文档用于定义执行增强型 RAG 的业务形态与架构特点，重点回答：

- 什么样的场景属于执行增强型 RAG
- 为什么 RAG 在 Agent 时代不再只是问答系统，而是任务执行前的重要知识模块
- 执行增强型 RAG 与 Workflow / Agent Runtime 应如何协同

## 2. 执行增强型 RAG 的定义

执行增强型 RAG 指的是：

系统在执行任务之前或执行过程中，先通过检索补充规则、案例、协议、历史记录和操作依据，再将这些知识用于后续工具调用、状态推进、人工确认或执行决策。

用一句话概括：

执行增强型 RAG 是 Agent / Workflow 的认知增强模块，而不是单独的问答模块。

## 3. 为什么这类场景近两年越来越重要

随着 Agent、Workflow 和 tool use 的发展，很多企业场景已经不是“只要回答问题”，而是：

- 先查知识，再做操作
- 先查规则，再生成配置
- 先查案例，再做诊断
- 先查 API / 平台协议，再调用系统能力

这意味着：

RAG 不是被 Agent 替代，而是在 Agent 时代变得更重要。

## 4. 近两年的关键变化

### 4.1 Agentic retrieval 直接强化了执行增强型 RAG 的价值

Azure AI Search 2026 年 agentic retrieval 文档已经明确把 retrieval 设计成多查询、多 source、结构化响应，面向 agent 和 chat app 消费。这说明 retrieval 正在被重新定位为 agent system 的正式知识层。

### 4.2 执行前 grounding 正在成为工具调用的安全与质量前提

近两年的工具调用与 agent 实践越来越清楚：

如果执行前没有可靠知识 grounding，系统很容易：

- 调错工具
- 用错参数
- 误解规则
- 在错误约束下执行

因此，执行增强型 RAG 其实也是一种 risk reduction architecture。

### 4.3 retrieval response 开始服务 runtime，而不只服务 UI

在执行增强型 RAG 中，retrieval 输出可能直接供：

- planner
- tool selector
- validator
- human review

使用，而不只是生成给用户看的一段回答。

## 5. 典型业务场景

### 5.1 调用 API 前先查协议

例如：

- 查接口说明
- 查参数约束
- 查限流规则
- 再执行调用

### 5.2 生成配置前先查规则和模板

例如：

- 先查组件协议
- 再生成配置
- 再查校验规则

### 5.3 诊断前先查案例和规范

例如：

- 先查错误案例
- 先查发布规范
- 再调用日志或校验工具

### 5.4 工作流节点执行前查前置条件

例如：

- 某一步需要满足什么规则
- 某动作是否允许在当前环境执行

## 6. 与 FAQ 型 / 生成辅助型 RAG 的区别

### 6.1 与 FAQ 型 RAG 的区别

FAQ 型 RAG 的终点通常是“回答”。

执行增强型 RAG 的终点通常是“让后续动作更正确”。

### 6.2 与生成辅助型 RAG 的区别

生成辅助型 RAG 的终点通常是“更好生成”。

执行增强型 RAG 的终点通常是“更安全、更可靠地决策与执行”。

## 7. 典型架构特点

执行增强型 RAG 通常包含：

- intent / task understanding
- retrieval planning
- knowledge retrieval
- evidence packaging
- tool selection / workflow step
- action / validation
- trace / audit

其 retrieval 输出通常会更结构化，并与 runtime 紧耦合。

## 8. 为什么执行增强型 RAG 通常需要更强的 query understanding

因为这类场景不仅要理解“用户在问什么”，还要理解：

- 当前任务在哪一步
- 当前要查的是规则、案例还是对象定义
- 检索结果用于解释、生成还是执行判断

因此，它通常比 FAQ 型 RAG 更依赖：

- intent recognition
- entity extraction
- source-aware recall
- task-aware context building

## 9. 为什么执行增强型 RAG 必须更重视 governance

因为 retrieval 结果会影响行动，因此需要更强治理：

- 权限过滤
- 来源可信度控制
- 最新性控制
- citation / evidence 保留
- trace / replay

否则错误知识会从“答错”升级为“做错”。

## 10. 面向低代码平台的典型形态

### 10.1 配置纠错

流程：

- 检索错误规则
- 检索类似案例
- 调用 schema 校验工具
- 输出修复建议

### 10.2 页面生成前规则检查

流程：

- 检索组件协议与模板
- 生成草稿
- 校验并修正

### 10.3 运维辅助执行

流程：

- 检索发布规范与历史案例
- 查询日志和状态
- 输出下一步动作建议
- 必要时进入人工确认

### 10.4 逻辑编排助手

流程：

- 检索节点定义和连接规则
- 生成工作流草稿
- 校验依赖和类型

## 11. 各大公司常见落地模式

近两年很多公司会把 RAG 放到执行链路中，典型模式包括：

- 企业 Copilot 在操作前先查知识库
- Agent 在调用 API 前先查文档
- 代码 / 运维助手在执行前先查规范与案例
- 内部平台助手在生成配置前先查协议

这些都属于执行增强型 RAG 的范畴。

## 12. 主要评测重点

建议重点评估：

- retrieval evidence 是否足以支撑后续动作
- action 前规则检索是否正确
- 是否降低错误执行率
- 是否减少无效 tool call
- citation / trace 是否完整
- 人工 review 时证据是否充分

## 13. 常见误区

### 13.1 把执行增强型 RAG 当成普通问答来做

### 13.2 检索结果不进入 runtime 结构化状态，只给模型自由看

### 13.3 不做 citation 和 evidence logging

### 13.4 retrieval 不区分规则、案例、对象定义

## 14. 一句话总结

执行增强型 RAG 的核心，是在行动前和行动中通过检索提供可追溯的知识依据，让 Workflow / Agent 的决策和执行建立在正确规则、案例和对象协议上；近两年的最新架构也越来越明确，RAG 已经不仅是问答引擎，更是 agentic system 的正式知识控制层。

## 15. 参考资料

- Azure AI Search agentic retrieval concept: [https://learn.microsoft.com/en-us/azure/search/search-agentic-retrieval-concept](https://learn.microsoft.com/en-us/azure/search/search-agentic-retrieval-concept)
- Azure AI Search RAG overview: [https://learn.microsoft.com/en-us/azure/search/retrieval-augmented-generation-overview](https://learn.microsoft.com/en-us/azure/search/retrieval-augmented-generation-overview)
- OpenAI A practical guide to building agents: [https://openai.com/business/guides-and-resources/a-practical-guide-to-building-ai-agents/](https://openai.com/business/guides-and-resources/a-practical-guide-to-building-ai-agents/)
- Anthropic Building Effective AI Agents: [https://www.anthropic.com/engineering/building-effective-agents](https://www.anthropic.com/engineering/building-effective-agents)
- Cohere Agentic RAG tutorial: [https://docs.cohere.com/v2/docs/agentic-rag](https://docs.cohere.com/v2/docs/agentic-rag)
