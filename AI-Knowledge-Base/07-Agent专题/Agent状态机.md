# Agent状态机

> 状态：第一版草案

## 1. 文档定位

本文档用于定义 Agent 系统中的状态机设计，重点回答：

- 为什么生产级 Agent 不应只是 while loop 调模型
- 状态机如何帮助 Agent 系统实现可观测、可恢复、可中断、可审计
- 近两年 LangGraph 等运行时实践为什么越来越强调 stateful / durable execution

## 2. 为什么 Agent 需要状态机

如果 Agent 只是：

- 调模型
- 看结果
- 再调模型
- 循环

那么系统通常很快会遇到以下问题：

- 当前到底执行到哪一步不清楚
- 中间结果难以恢复
- 错误后只能从头再来
- 人工确认很难插入
- tracing 很难做
- replay 很难做

这说明生产级 Agent 的核心不只是推理能力，而是状态可管理性。

## 3. 近两年的关键变化

### 3.1 stateful runtime 已成为 agent 系统的主流方向

LangGraph 把 durable execution、checkpoints、resume、interrupt 直接定义为长期运行 agent 的核心能力。这表明行业已经不再把 agent 视为一次性模型调用，而是视为长运行的状态机系统。

### 3.2 HITL 和 long-running tasks 推动状态机前置

一旦系统支持：

- 人工确认
- 长时任务
- 任务恢复
- 多轮工具调用

就必须显式记录状态，而不能只靠临时上下文。

### 3.3 状态机正在成为 Agent trace、eval、replay 的共同基础

如果没有稳定状态节点，后续很多治理能力都无法真正落地。

## 4. 什么是 Agent 状态机

Agent 状态机可以理解为：

用一组显式状态、状态转换条件和执行动作，来管理 Agent 在整个任务生命周期中的运行过程。

用一句话概括：

状态机的本质，是把 Agent 从隐式行为系统变成显式流程系统。

## 5. 状态机解决什么问题

### 5.1 任务生命周期可见化

让系统清楚知道当前任务处于：

- 理解阶段
- 检索阶段
- 工具执行阶段
- 校验阶段
- 人工确认阶段
- 完成或失败阶段

### 5.2 错误恢复

可以从特定状态恢复，而不是一律从头开始。

### 5.3 人工介入

人工确认、审批、补充输入都可以作为显式状态存在。

### 5.4 审计与回放

状态转换可以成为 trace 和 replay 的天然骨架。

## 6. 推荐的状态集合

具体状态应根据系统调整，但一个通用 Agent 通常可考虑：

- `INIT`
- `UNDERSTAND`
- `PLAN`
- `RETRIEVE`
- `ACT`
- `OBSERVE`
- `VALIDATE`
- `HUMAN_REVIEW`
- `RETRY`
- `COMPLETE`
- `FAILED`
- `ESCALATE`

### 6.1 INIT

初始化任务与上下文。

### 6.2 UNDERSTAND

做目标理解和任务模型构建。

### 6.3 PLAN

由 planner 决定下一步路径。

### 6.4 RETRIEVE

执行 retrieval 或知识补全。

### 6.5 ACT

调用 tools 或执行动作。

### 6.6 OBSERVE

读取工具结果或环境反馈。

### 6.7 VALIDATE

验证当前结果是否满足约束。

### 6.8 HUMAN_REVIEW

进入人工确认、审批或补充输入阶段。

### 6.9 RETRY

在可恢复错误下重新尝试。

### 6.10 COMPLETE / FAILED / ESCALATE

对应正常结束、失败结束和升级处理。

## 7. 状态转换应关注什么

建议每次状态转换至少记录：

- fromState
- toState
- trigger
- inputSummary
- outputSummary
- toolInfo（如有）
- errorInfo（如有）
- timestamp
- traceId

这样后续 tracing、replay 和 eval 才有足够基础。

## 8. 为什么状态机优于隐式循环

### 8.1 更可观测

### 8.2 更可恢复

### 8.3 更容易插 HITL

### 8.4 更容易定义终止条件

### 8.5 更适合治理和审计

## 9. 状态机与 Runtime 的关系

Runtime 负责：

- 执行状态转换
- 持久化状态
- 处理中断和恢复
- 协调工具和模型调用

状态机负责：

- 定义任务生命周期骨架
- 明确转移逻辑
- 约束系统行为边界

## 10. 状态机与 Planner 的关系

Planner 决定“下一步应该做什么”，状态机决定“系统允许进入哪些阶段，以及如何从一个阶段转到另一个阶段”。

因此，Planner 和状态机是协同关系，而不是替代关系。

## 11. 面向低代码平台的状态机示例

### 11.1 配置诊断 Agent

可能状态：

- INIT
- UNDERSTAND
- RETRIEVE_RULES
- ACT_VALIDATE_SCHEMA
- OBSERVE_RESULT
- GENERATE_DIAGNOSIS
- HUMAN_REVIEW（可选）
- COMPLETE

### 11.2 页面生成 Agent

可能状态：

- INIT
- UNDERSTAND_GOAL
- RETRIEVE_TEMPLATE
- RETRIEVE_COMPONENT_RULES
- GENERATE_DRAFT
- VALIDATE_SCHEMA
- REPAIR
- COMPLETE

### 11.3 运维辅助 Agent

可能状态：

- INIT
- UNDERSTAND_ISSUE
- RETRIEVE_POLICY
- ACT_GET_LOG
- ACT_GET_RELEASE_RECORD
- OBSERVE
- SUMMARIZE
- ESCALATE / COMPLETE

## 12. 常见误区

### 12.1 状态机太细，导致流程僵硬且难维护

### 12.2 状态机太粗，失去可观测与可恢复价值

### 12.3 不记录状态转换上下文

### 12.4 把错误恢复、HITL、审计放在状态机外部零散处理

## 13. 一句话总结

Agent 状态机的核心，是把任务生命周期拆成显式可管理状态，并围绕状态转换建立持久化、恢复、HITL、trace 和审计能力；近两年的最佳实践也越来越明确，生产级 agent 不是隐式循环，而是 stateful runtime 驱动的显式执行系统。

## 14. 参考资料

- LangGraph durable execution: [https://docs.langchain.com/oss/python/langgraph/durable-execution](https://docs.langchain.com/oss/python/langgraph/durable-execution)
- LangGraph overview: [https://docs.langchain.com/oss/python/langgraph/overview](https://docs.langchain.com/oss/python/langgraph/overview)
- Anthropic, Effective harnesses for long-running agents: [https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents)
