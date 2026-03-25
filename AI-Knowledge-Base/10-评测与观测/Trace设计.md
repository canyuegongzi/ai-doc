# Trace设计

> 状态：第一版草案

## 1. 文档定位

本文档用于定义 AI 系统中的 Trace 设计，重点回答：

- 为什么 AI 系统需要 Trace，而不是只靠普通日志
- Trace、审计日志、回放、指标监控之间的边界是什么
- 近两年为什么 tracing 正成为 Agent / RAG / Tool 系统的标准基础设施

## 2. 什么是 Trace

Trace 可以理解为：

对一次 AI 请求从入口到结束的完整执行链路进行结构化记录和串联的机制，用来描述：

- 这个请求经过了哪些阶段
- 每一步做了什么
- 输入输出是什么
- 花了多少时间
- 消耗了多少资源
- 最终结果是如何形成的

通常一个 Trace 由多个 Span 组成，每个 Span 表示一段具体工作单元，例如：

- LLM 调用
- retrieval
- rerank
- tool call
- guardrail check
- HITL decision

用一句话概括：

Trace 的本质，是把 AI 系统的隐式执行过程变成可观察、可定位、可比较的显式链路图。

## 3. 为什么 AI 系统特别需要 Trace

### 3.1 AI 失败通常不是单点失败

当结果出错时，问题可能出在：

- query rewrite
- 检索召回
- 上下文装配
- 模型推理
- tool 参数生成
- tool 执行失败
- HITL 决策
- 动作验证失败

如果没有 Trace，团队很难判断错误在哪一层发生。

### 3.2 Agent / RAG / Tool 系统具有多步、多组件、多状态特征

AI 系统不像普通接口那样“一次请求、一条 SQL、一段返回”，而是典型的多阶段链路。Trace 是把这些阶段串起来的基础设施。

### 3.3 近两年的 runtime 和 observability 工具都在强化 Trace

OpenAI Agents SDK、LangSmith、Phoenix 等工具都把 tracing 作为一等能力，说明行业已经形成共识：

AI 应用的排障、优化和质量治理，必须建立在链路级可观测之上。

## 4. Trace 要解决什么问题

### 4.1 链路可见性

看见一次请求完整经过了什么步骤。

### 4.2 性能定位

知道耗时主要花在哪里。

### 4.3 失败定位

知道失败是在哪个 span 或哪个阶段发生的。

### 4.4 质量分析

把质量问题回溯到具体步骤。

### 4.5 样本沉淀

把高价值失败样本、异常样本沉淀为后续 eval 数据源。

## 5. Trace、日志、审计、回放的边界

### 5.1 与普通日志的区别

普通日志更零散，Trace 更强调同一请求的端到端关联。

### 5.2 与审计日志的区别

Trace 偏执行过程与调试；审计偏责任、授权、审批和合规。

### 5.3 与回放的关系

回放通常依赖 Trace 和状态快照，但 Trace 本身不一定足够支持完整 replay。

## 6. 近两年的关键变化

### 6.1 Trace 正从 LLM span 扩展为全链路 span

早期很多系统只追踪模型调用；现在更成熟的做法会追踪：

- retrieval spans
- tool spans
- routing spans
- approval spans
- guardrail spans
- cost spans

### 6.2 typed span 和 structured attributes 越来越重要

仅有文本日志不够，更需要结构化属性用于聚合和检索。

### 6.3 Trace 正在与 eval、annotation、feedback 打通

很多 observability 平台开始支持：

- 给 trace 打标签
- 在 trace 上挂评分
- 在 trace 上挂用户反馈

这让 tracing 变成质量闭环的中心资产。

## 7. 推荐的 Trace 结构

建议每个请求至少包含：

### 7.1 顶层 Trace 字段

- `traceId`
- `requestId`
- `sessionId`
- `tenantId`
- `userId`
- `sceneType`
- `workflowName`
- `startTime`
- `endTime`
- `status`

### 7.2 Span 字段

- `spanId`
- `parentSpanId`
- `spanType`
- `name`
- `status`
- `latencyMs`
- `inputSummary`
- `outputSummary`
- `errorCode`
- `costMetrics`
- `riskFlags`

## 8. 推荐的 Span 分类

建议至少支持以下 span 类型：

### 8.1 LLM Span

记录：

- model
- provider
- prompt / messages summary
- token usage
- latency

### 8.2 Retrieval Span

记录：

- query summary
- filters
- index source
- topK
- hit summaries

### 8.3 Tool Span

记录：

- tool name
- params summary
- result summary
- retry count

### 8.4 Planner / Routing Span

记录：

- route decision
- nextActionType
- confidence
- risk flags

### 8.5 Guardrail / Approval Span

记录：

- policy checks
- HITL trigger
- approval decision

### 8.6 Validation Span

记录：

- schema validation
- business rule validation
- output safety check

## 9. 在 RAG 中如何落地 Trace

建议完整记录：

- rewrite span
- recall span
- rerank span
- context builder span
- answer generation span

这样才能定位问题是：

- 没召回
- 召回错了
- 排序错了
- 拼文错了
- 生成错了

## 10. 在 Tool / Agent 中如何落地 Trace

对于 Tool / Agent，建议至少记录：

- planner decision
- tool selection
- parameter generation
- tool execution
- observation
- state transition
- escalation / approval
- final outcome

在复杂 Agent 中，Trace 常常比最终回答更有诊断价值。

## 11. 面向低代码平台的典型 Trace 设计建议

### 11.1 Schema 生成链路

建议看到：

- 查了哪些模板
- 查了哪些组件物料
- 生成用了哪个模型
- schema 校验是否通过

### 11.2 配置诊断链路

建议看到：

- 先查了规则还是先查了日志
- 哪一步给出了 root cause
- 是否触发自动修复建议

### 11.3 页面发布链路

建议看到：

- 风险检查
- 审批触发
- 执行动作
- 发布后验证

## 12. Trace 设计的常见误区

### 12.1 只记录 LLM 调用，不记录 retrieval 和 tools

这样仍然无法解释大多数复杂问题。

### 12.2 span 只有自然语言日志，没有结构化属性

后续无法聚合分析和做质量指标。

### 12.3 trace 与身份、租户、场景脱节

会导致链路可见但无法做责任和分布分析。

### 12.4 trace 不做脱敏和权限控制

Trace 本身会变成新的敏感数据泄露面。

### 12.5 trace 只用于开发，不接入线上治理

那它就很难真正支撑生产优化。

## 13. 一句话总结

Trace 设计的核心，是把 AI 系统中跨模型、检索、工具、审批和动作执行的多阶段链路组织成统一的可观测骨架，并通过 span 结构把质量、性能、风险和成本信息都绑定到同一条请求轨迹上；近两年的最佳实践也越来越明确，AI 系统能否被稳定调试和持续优化，很大程度上取决于 Trace 是否设计得足够完整。

## 14. 参考资料

- OpenAI Agents SDK tracing: [https://openai.github.io/openai-agents-python/tracing/](https://openai.github.io/openai-agents-python/tracing/)
- OpenAI Agents JS tracing: [https://openai.github.io/openai-agents-js/guides/tracing/](https://openai.github.io/openai-agents-js/guides/tracing/)
- LangSmith observability: [https://docs.smith.langchain.com/observability](https://docs.smith.langchain.com/observability)
- Phoenix tracing tutorial: [https://arize.com/docs/phoenix/tracing](https://arize.com/docs/phoenix/tracing)
- OpenTelemetry: [https://opentelemetry.io/](https://opentelemetry.io/)
