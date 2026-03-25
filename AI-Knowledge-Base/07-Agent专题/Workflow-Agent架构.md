# Workflow-Agent架构

> 状态：第一版草案

## 1. 文档定位

本文档用于定义 Workflow-Agent 架构，重点回答：

- 为什么 Workflow-Agent 是企业中最适合落地的第一代 Agent 形态
- Workflow 与 Agent 如何组合，既保留灵活性又保持可控性
- 近两年行业最佳实战为何普遍推荐从 workflow-first 走起

## 2. 为什么 Workflow-Agent 是企业最佳起点

在企业系统中，很多任务虽然复杂，但并不是完全未知。它们通常有：

- 大致固定的步骤
- 明确的系统边界
- 清楚的高风险节点
- 必要的人工确认要求

这类任务如果直接用完全自由 agent 去做，往往会带来：

- 不稳定
- 不可预测
- 难以审计
- 难以回放
- 成本失控

因此，Workflow-Agent 架构的价值在于：

用 workflow 约束整体边界，用 agent 提供局部智能弹性。

## 3. 近两年的关键变化

### 3.1 Workflow 正在重新成为 agent 设计主流

Anthropic 在 2024 年的文章中明确把 workflow 单列为 agentic systems 的核心类型，并且强烈建议先从简单、可组合的模式开始。

### 3.2 企业 agent 架构正在从“自由循环”走向“状态图 + 条件分支”

LangGraph 的 StateGraph、durable execution、interrupt / resume 等能力本质上都说明：

企业 agent 更像“可恢复工作流”，而不是“无限循环助手”。

### 3.3 Workflow 已不再等于传统 BPM

现代 Workflow-Agent 系统不是纯规则流，它会在节点内使用：

- LLM
- retrieval
- tools
- local planner
- evaluator

因此，它比传统 BPM 更智能，但比完全开放 Agent 更可控。

## 4. 什么是 Workflow-Agent 架构

Workflow-Agent 架构可以定义为：

由显式 workflow 提供任务主路径、状态边界和关键控制点，在特定节点上由 LLM / Planner / Retrieval / Tool 动态参与决策和执行的混合型 agent 架构。

用一句话概括：

Workflow-Agent 的本质，是把 agent 放进一个有边界的流程骨架中运行。

## 5. Workflow-Agent 解决什么问题

### 5.1 解决 Agent 过度自由的问题

用 workflow 限制路径边界，避免系统到处乱试。

### 5.2 解决状态不清晰的问题

workflow 节点和状态机让系统更容易追踪当前处于哪一步。

### 5.3 解决治理难落地的问题

审批、人工确认、风险控制、重试、回放，都更容易嵌入 workflow 节点中。

### 5.4 解决复杂任务工程化落地问题

让多步骤任务更容易被拆解、调试和评估。

## 6. Workflow-Agent 的核心组成

### 6.1 Workflow Skeleton

定义主要节点、状态和边。

### 6.2 Agentic Nodes

在部分节点中，允许模型或 planner 做局部决策。

### 6.3 Tool Layer

为每个节点提供可调用的动作能力。

### 6.4 Retrieval Layer

在需要时为节点提供知识增强。

### 6.5 Control Points

包括：

- human review
- timeout
- retry
- escalation
- stop condition

## 7. 典型 workflow 节点示例

以“配置诊断”场景为例，Workflow-Agent 可以拆为：

1. 任务理解
2. 实体抽取
3. 检索规则与案例
4. 调用校验工具
5. 汇总结果
6. 判断是否足够回答
7. 输出建议或转人工

其中：

- 第 3 步和第 6 步就很适合 agentic decision
- 其余边界仍由 workflow 控制

## 8. 为什么 Workflow-Agent 更适合企业场景

### 8.1 更可观测

每个步骤和状态转换都可记录。

### 8.2 更可审计

高风险节点可插审批、人工确认和证据保留。

### 8.3 更可恢复

长流程中断后可以从中间状态恢复。

### 8.4 更适合评测

可以评估：

- 哪个节点问题最多
- 哪类分支最常失败
- 哪个工具在什么阶段出问题

## 9. Workflow-Agent 与纯 Workflow 的区别

纯 Workflow：

- 规则控制更强
- 动态决策较少

Workflow-Agent：

- 主路径明确
- 局部节点允许模型动态决策
- 更适合复杂但仍需控制的任务

## 10. Workflow-Agent 与 Tool-Based Agent 的区别

Workflow-Agent：

- 流程骨架明确
- Agent 自由度局部化

Tool-Based Agent：

- 模型更自由地决定何时调什么工具
- 对 tool set 设计与 runtime 治理要求更高

## 11. 面向低代码平台的典型场景

### 11.1 页面生成工作流

流程：

- 理解目标
- 检索模板和组件协议
- 生成 schema 草稿
- 校验
- 修复
- 输出

### 11.2 配置纠错工作流

流程：

- 理解错误描述
- 检索规则与案例
- 调用校验工具
- 汇总诊断
- 给修复建议

### 11.3 运维辅助工作流

流程：

- 接收问题
- 查规范
- 查日志
- 查发布记录
- 汇总分析
- 转人工或输出下一步建议

## 12. 常见误区

### 12.1 认为 Workflow-Agent 不够“高级”

在企业里，稳定和可控通常比自由更重要。

### 12.2 把 workflow 节点做得过粗

这样无法真正发挥节点级 agentic decision 的价值。

### 12.3 所有节点都放开给模型自由决定

这会让 Workflow-Agent 退化成自由 Agent。

### 12.4 没有 durable execution 就做长流程 workflow-agent

中断恢复和 HITL 会非常脆弱。

## 13. 一句话总结

Workflow-Agent 架构的核心，是用显式流程骨架承载任务主路径，再在局部节点引入 LLM 与工具的动态决策能力，从而在企业场景中实现“既智能又可控”的 agentic system；这也是近两年最被行业最佳实践反复验证的落地方向之一。

## 14. 参考资料

- Anthropic, Building Effective AI Agents: [https://www.anthropic.com/engineering/building-effective-agents](https://www.anthropic.com/engineering/building-effective-agents)
- OpenAI, A practical guide to building agents: [https://openai.com/business/guides-and-resources/a-practical-guide-to-building-ai-agents/](https://openai.com/business/guides-and-resources/a-practical-guide-to-building-ai-agents/)
- LangGraph overview: [https://docs.langchain.com/oss/python/langgraph/overview](https://docs.langchain.com/oss/python/langgraph/overview)
- LangGraph durable execution: [https://docs.langchain.com/oss/python/langgraph/durable-execution](https://docs.langchain.com/oss/python/langgraph/durable-execution)
