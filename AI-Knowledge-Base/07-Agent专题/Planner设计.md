# Planner设计

> 状态：第一版草案

## 1. 文档定位

本文档用于定义 Agent 系统中的 Planner 设计，重点回答：

- Planner 在 Agent 体系中到底承担什么职责
- Planner 与目标理解、任务模型、Workflow、Tool Routing 的边界是什么
- 近两年行业在 planner 设计上形成了哪些稳定的工程共识

## 2. 为什么 Planner 是 Agent 系统的关键中枢

很多 Agent 失败，并不是模型能力不足，而是系统在“下一步做什么”上没有稳定控制。

典型问题包括：

- 明明该先检索，结果先调用了高风险工具
- 明明该拆成多个子任务，结果直接粗暴回答
- 明明该转人工确认，结果继续自动执行
- 工具路径很多，但没有清晰选择逻辑

这说明 Agent 不只是需要一个会推理的模型，而是需要一个能把目标转化为“下一步行动计划”的规划层。

## 3. 近两年的关键变化

### 3.1 Planner 正从“Chain-of-Thought 附属能力”升级为正式架构层

随着 workflow agent、tool-based agent 和 agentic retrieval 的发展，Planner 已经不再只是 prompt 里的一段“先思考再行动”，而越来越成为：

- 任务拆解器
- 路由器
- 步骤决策器
- 检索与工具选择协调器

### 3.2 行业共识转向“简单规划优先”

Anthropic 在《Building Effective AI Agents》中给出的一个重要建议是：

从最简单可行的模式开始，只有在复杂度真正必要时再升级。

这对 Planner 设计的启发非常直接：

不要一开始就追求全局复杂自治规划，而应优先构建：

- 明确任务分类
- 局部决策
- 小步可验证规划

### 3.3 Planner 与 runtime 正在更深耦合

LangGraph 把 durable execution、state graph、resume、HITL 放在运行时核心层，说明 planner 现在不能只产出“计划文本”，而应产出可以被 runtime 真正消费的结构化行动计划。

## 4. Planner 的职责边界

### 4.1 应负责什么

- 判断当前任务是否需要拆解
- 选择下一步是检索、推理、工具调用还是人工确认
- 生成局部行动计划
- 在多种候选路径中做选择
- 决定是否终止、继续或升级处理

### 4.2 不应负责什么

- 不直接执行工具
- 不负责最终知识检索结果本身
- 不负责最终输出校验
- 不替代 runtime 的状态持久化与错误恢复

Planner 是决策层，而不是执行层。

## 5. Planner 需要解决的核心问题

### 5.1 要不要规划

不是所有任务都需要复杂规划。很多简单任务只需要：

- 直接回答
- 单次 retrieval
- 单次 tool 调用

### 5.2 先做什么

例如：

- 先 retrieval
- 先查当前状态
- 先调用工具
- 先拆子问题

### 5.3 要不要拆子任务

当任务足够复杂时，需要决定：

- 是否拆成多个小目标
- 拆分粒度多细
- 是否并行执行

### 5.4 什么时候停

Planner 应参与判断：

- 是否已有足够信息
- 是否达到成功标准
- 是否进入失败或高风险状态

## 6. 常见 Planner 模式

### 6.1 规则路由式 Planner

适用于：

- 任务类型清晰
- 路径边界固定
- 企业高控制场景

特点：

- 稳定
- 容易调试
- 可与 workflow 紧密结合

### 6.2 局部决策式 Planner

不是一次做完整计划，而是每一步只决定下一步。

这是很多生产 agent 更实用的方式。

### 6.3 先规划后执行式 Planner

先生成完整计划，再逐步执行。

适用于：

- 复杂但较长程的任务
- 需要先让人审阅计划的场景

### 6.4 规划 + 评估 + 修正模式

用于：

- 复杂任务
- 高要求质量场景
- 需要 planner 自我修正的情况

Anthropic 提到的 evaluator-optimizer pattern，在某些 agent 任务中就属于这一类思想。

## 7. 推荐的 Planner 输出结构

Planner 最好输出结构化结果，而不是只输出自然语言计划。

建议至少包含：

- nextActionType
- targetTool / targetFlow
- subTasks（可选）
- requiredEvidence
- stopConditionCheck
- confidence
- riskFlags

例如：

```json
{
  "nextActionType": "retrieve",
  "targetFlow": "diagnosis_case_retrieval",
  "requiredEvidence": ["component_rule", "error_case"],
  "riskFlags": [],
  "confidence": 0.82
}
```

## 8. Planner 与任务模型的关系

任务模型定义：

- 目标
- 约束
- 当前状态

Planner 则基于这些信息决定：

- 下一步做什么
- 是否拆任务
- 何时终止

任务模型是 planner 的输入骨架。

## 9. Planner 与 Workflow 的关系

在 workflow-first 体系中，Planner 通常更像：

- workflow router
- node transition decider
- conditional branch selector

这比完全自由 Planner 更适合企业场景。

## 10. Planner 与 RAG 的关系

Planner 在现代 agentic system 中经常直接决定：

- 是否要 retrieval
- 查什么 source
- 是否要 multi-query
- 检索结果是否足够继续

这意味着 Planner 实际上已经在深度介入 RAG 路由。

## 11. 面向低代码平台的 Planner 典型任务

### 11.1 页面生成

Planner 要决定：

- 先查模板还是先查组件协议
- 是否需要数据源定义
- 生成后是否先做 schema 校验

### 11.2 配置诊断

Planner 要决定：

- 先查规则、案例还是先查运行日志
- 是否需要继续调用工具
- 是否需要人工确认问题定位

### 11.3 运维分析

Planner 要决定：

- 先查发布记录还是先查构建日志
- 是否需要跨 source 汇总
- 是否应转人工处理

## 12. 常见误区

### 12.1 Planner 一定要先输出完整大计划

很多生产系统更适合局部决策，而不是一次性长规划。

### 12.2 Planner 输出只有自然语言

这会让 runtime 很难稳定消费。

### 12.3 Planner 不感知风险和权限

这样容易在错误边界内做出看似合理的决策。

### 12.4 过早追求复杂 planner，而忽视简单路由已足够的场景

## 13. 一句话总结

Planner 设计的核心，是把任务模型转化为可执行、可追踪、可中断的下一步决策，并在 retrieval、tool use、workflow 分支和终止条件之间做受控选择；近两年的最佳实践也越来越明确，企业最实用的 Planner 通常不是最复杂的，而是最结构化、最稳定、最便于 runtime 消费的那种。

## 14. 参考资料

- Anthropic, Building Effective AI Agents: [https://www.anthropic.com/engineering/building-effective-agents](https://www.anthropic.com/engineering/building-effective-agents)
- OpenAI, A practical guide to building agents: [https://openai.com/business/guides-and-resources/a-practical-guide-to-building-ai-agents/](https://openai.com/business/guides-and-resources/a-practical-guide-to-building-ai-agents/)
- LangGraph durable execution: [https://docs.langchain.com/oss/python/langgraph/durable-execution](https://docs.langchain.com/oss/python/langgraph/durable-execution)
