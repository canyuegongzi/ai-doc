# Agent总览

> 状态：第一版草案

## 1. 文档定位

本文档用于从架构视角总结 Agent 系统，回答：

- 什么是企业 AI 系统中的 Agent
- Agent 与 Workflow、Tool、RAG、Memory、Runtime 的关系是什么
- 近两年行业对 Agent 的理解为什么发生了明显变化
- 为什么企业最应该建设的是“可控 Agent 系统”，而不是“最自由 Agent”

## 2. 为什么 Agent 这两年成为主线话题

随着大模型在以下能力上的持续提升：

- reasoning
- tool use
- long context
- multimodal understanding
- code generation

AI 系统的应用目标也从“回答问题”逐渐扩展到“完成任务”。

这类任务通常包括：

- 多步骤信息检索
- 工具调用
- 状态推进
- 错误恢复
- 人工协作
- 最终交付结果

因此，Agent 不再只是一个研究概念，而逐步成为很多企业 AI 系统的正式架构形态。

## 3. 近两年的关键变化

### 3.1 行业正在从“Agent 神话”回到“Agent 工程”

Anthropic 在 2024 年底的《Building Effective AI Agents》中给出了一个非常重要的判断：

最成功的生产系统通常不是依赖复杂框架，而是基于简单、可组合的模式来构建 agentic systems。

这代表行业正在形成共识：

Agent 不是神秘智能体，而是一种工程系统。

### 3.2 Workflow 与 Agent 的边界被重新讲清楚

Anthropic 明确区分：

- Workflows：预定义代码路径中的 LLM 与工具编排
- Agents：由模型动态主导流程和工具使用的系统

这一定义很重要，因为它让企业可以更清楚地知道：

什么时候该做 workflow，什么时候才需要更高自由度的 agent。

### 3.3 Runtime、Memory、Human-in-the-loop 已成为正式基础设施

LangGraph 把 durable execution、human-in-the-loop、memory、streaming 直接作为 agent runtime 的核心能力来强调，这说明 agent 系统的重点正在从“prompt loop”转向“状态化运行时”。

### 3.4 Agent 与 RAG 正在深度耦合

当前越来越多场景中的 Agent 都不是“脱离知识工作的 Agent”，而是依赖：

- retrieval
- tools
- memory
- planning

组成的 augmented LLM。

这和 Anthropic 的“augmented LLM”定义高度一致。

## 4. 什么是 Agent

可以将 Agent 定义为：

一个围绕目标持续感知、决策、调用外部能力、观察反馈并推进任务状态的 AI 执行系统。

它通常具备以下特征：

- 目标导向
- 多步骤执行
- 工具使用
- 状态管理
- 环境反馈闭环
- 可中断或可终止

用一句话概括：

Agent 的本质，是一个让模型在边界内持续做决策并推进任务完成的系统，而不是单次回答接口。

## 5. Agent 解决什么问题

### 5.1 多步骤任务问题

很多企业任务并不是一句话回答就结束，而是需要一系列步骤协作完成。

### 5.2 动态路径问题

有些任务路径无法完全提前写死，系统需要根据中间结果决定下一步。

### 5.3 工具协作问题

模型必须借助工具访问外部系统、执行查询、触发动作。

### 5.4 状态闭环问题

任务执行中需要记住：

- 当前步骤
- 中间结果
- 已调用工具
- 错误与重试
- 是否需要人工确认

## 6. Agent 不解决什么问题

### 6.1 不替代知识工程

没有高质量知识底座，Agent 很容易在错误知识上行动。

### 6.2 不替代治理边界

Agent 越强，越需要 guardrails、权限控制、审批和审计。

### 6.3 不意味着必须高度自治

很多企业场景里，最佳方案不是最大自由度，而是更稳的 workflow + agent 混合模式。

## 7. Agent 的核心组成

### 7.1 目标理解

识别用户或系统当前要完成什么任务。

### 7.2 Planner / 决策层

决定下一步做什么，是否拆分子任务，是否要检索或调用工具。

### 7.3 Tool / Action 层

执行查询、操作、校验、通知等外部动作。

### 7.4 Observation 层

获取环境反馈和工具结果。

### 7.5 State / Memory 层

保存任务状态、历史动作和上下文。

### 7.6 Runtime / Control 层

管理循环、终止条件、失败恢复、人工确认等。

## 8. Agent 与 Workflow 的关系

在企业实践中，Workflow 与 Agent 最合理的关系通常是：

- Workflow 负责主流程和边界
- Agent 负责局部动态决策

这也是为什么当前行业最佳实战通常建议：

先 workflow，后更自由的 agent。

## 9. Agent 与 RAG 的关系

Agent 和 RAG 不是两个彼此替代的方向。

更合理的关系是：

- RAG 负责认知增强
- Agent 负责执行增强
- RAG 是 Agent 的知识底座
- Agent 是 RAG 的行动扩展

## 10. Agent 与 Tool 的关系

Agent 若不具备 Tool 能力，通常只能停留在建议层。

Tool 负责：

- 查询真实状态
- 调用外部系统
- 验证生成结果
- 触发受控动作

因此，Tool 质量和 Tool 文档质量，往往直接决定 Agent 质量。

## 11. Agent 与 Runtime 的关系

很多人把 Agent 理解为“模型 + 循环”，但真正生产级的 Agent 更接近：

- 状态机
- 调度器
- 工具协议
- trace
- memory
- HITL
- error recovery

这意味着 Runtime 才是复杂 Agent 系统真正稳定性的基础设施。

## 12. 典型 Agent 模式

### 12.1 Workflow-first Agent

最适合企业早期落地。

### 12.2 Tool-based Agent

适合多工具协作、受控动态决策。

### 12.3 Planner-Executor Agent

适合复杂任务拆解后再执行。

### 12.4 Multi-Agent

适合明确存在角色分工、任务复杂度足够高的场景。

但企业中不宜过早使用。

## 13. 为什么企业最需要的是“可控 Agent”

因为企业真正关心的通常不是：

- 它看起来多聪明

而是：

- 能否在边界内可靠完成任务
- 能否被审计
- 能否被回放
- 能否被中断
- 高风险动作能否被拦住

这也是 OpenAI 和 Anthropic 最新指南都反复强调 guardrails、tool safeguards、HITL 的原因。

## 14. 面向低代码平台如何理解 Agent

在低代码平台中，Agent 更适合服务于：

- 页面生成工作流
- 配置诊断流程
- 逻辑编排辅助
- 平台运维辅助
- 受控操作建议与执行

而不应被理解成“一个随便聊天的大模型”。

它更像：

平台智能助手 + 任务协作器 + 受控执行系统

## 15. 常见误区

### 15.1 把 Agent 等同于无限循环的大模型

### 15.2 把 Agent 等同于工具调用本身

### 15.3 一开始就追求自治而忽略 Runtime 和治理

### 15.4 认为 Agent 会自动理解企业领域知识

## 16. 一句话总结

Agent 总览的核心结论是：当前主流企业 Agent 架构，已经从“模型循环调用工具”的模糊概念，演进为“augmented LLM + runtime + tools + memory + governance”的工程系统；而行业最佳实践也越来越明确，企业最需要的不是最自由的 agent，而是最可控、最可追踪、最能稳定完成任务的 agentic system。

## 17. 参考资料

- Anthropic, Building Effective AI Agents: [https://www.anthropic.com/engineering/building-effective-agents](https://www.anthropic.com/engineering/building-effective-agents)
- OpenAI, A practical guide to building agents: [https://openai.com/business/guides-and-resources/a-practical-guide-to-building-ai-agents/](https://openai.com/business/guides-and-resources/a-practical-guide-to-building-ai-agents/)
- LangGraph overview: [https://docs.langchain.com/oss/python/langgraph/overview](https://docs.langchain.com/oss/python/langgraph/overview)
- LlamaIndex workflows: [https://docs.llamaindex.ai/en/stable/module_guides/workflow/](https://docs.llamaindex.ai/en/stable/module_guides/workflow/)
