# Multi-Agent架构

> 状态：第一版草案

## 1. 文档定位

本文档用于定义 Multi-Agent 架构，重点回答：

- 什么情况下真正需要多智能体架构
- Multi-Agent 与 Workflow-Agent、Tool-Based Agent 的边界是什么
- 近两年多智能体工程实践给出了哪些新的约束和经验

## 2. 为什么 Multi-Agent 既热门又容易被误用

随着单 Agent 能力不断提升，很多团队会自然想到：

- 是否可以让多个 Agent 分工合作
- 是否可以让一个主 Agent 带多个子 Agent
- 是否可以并行处理复杂任务

这类思路在某些场景非常有效，但也很容易被滥用。

因为 Multi-Agent 会显著带来：

- 架构复杂度上升
- 协调成本上升
- token 成本上升
- 评测难度上升
- 状态一致性问题上升

所以 Multi-Agent 不应被视为默认更高级形态，而应被视为在特定问题类型下才值得引入的架构模式。

## 3. 近两年的关键变化

### 3.1 行业开始更谨慎地看待 Multi-Agent

LangChain 的多智能体文档直接指出：并不是每个复杂任务都需要 multi-agent，很多任务一个单 agent 配合合适 tools 和 prompt 就足够了。

这和近两年的工程实践非常一致：

Multi-Agent 的价值是有条件成立的，而不是普适优势。

### 3.2 Anthropic 的多智能体研究系统给出了更现实的判断

Anthropic 在 2025 年 6 月公开的多智能体研究系统文章中指出：

- Multi-agent 在 breadth-first、可并行、信息量远超单 context window 的任务上表现非常强
- 但 token 成本会显著增加
- 编排和协调复杂度也明显增加
- 并不适合依赖强共享上下文和高耦合依赖的任务

这是目前最有价值的公开实战结论之一。

### 3.3 Multi-Agent 正逐渐从概念演示走向 orchestrator-worker 实践模式

目前最常见、最实用的 Multi-Agent 模式不是任意 agent 彼此随意通信，而是：

- 一个 lead / orchestrator agent
- 多个 worker / specialist agents
- 清晰任务边界
- 明确输出格式

这也是 Anthropic Research 系统公开采用的主模式。

## 4. 什么是 Multi-Agent 架构

Multi-Agent 架构可以定义为：

由多个具备独立上下文、独立任务边界或独立能力侧重的 agent 共同协作完成任务的系统架构。

用一句话概括：

Multi-Agent 的本质，是把复杂任务拆给多个具有上下文或角色分工的 agent 并进行协调，而不是让一个 agent 独自承担全部任务。

## 5. Multi-Agent 适合解决什么问题

### 5.1 并行探索问题

当问题天然可以拆成多个独立方向并行搜索时，Multi-Agent 非常有价值。

### 5.2 上下文容量问题

当一个任务需要处理的信息量太大，单 Agent 难以在同一上下文窗口中稳定完成时，Multi-Agent 可以把不同上下文分摊到多个 Agent 上。

### 5.3 角色分工问题

当任务存在明确的专业角色分工时，可以用不同 agent 对应不同职责。

### 5.4 团队与能力模块解耦问题

在大型系统中，不同 team 维护不同 agent 能力边界，也是一种合理动机。

## 6. Multi-Agent 不适合什么问题

### 6.1 强依赖共享上下文的问题

如果每一步都强依赖同一份细粒度上下文，拆给多个 agent 反而会增加协调成本。

### 6.2 不可并行的问题

如果任务本质是严格串行，Multi-Agent 往往收益很低。

### 6.3 低价值轻任务

简单任务使用多个 agent 往往会显著增加成本而几乎没有收益。

### 6.4 高耦合编码 / 配置修改类任务（当前阶段常见）

Anthropic 的经验也指出，很多编码任务并没有足够多真正可并行的部分，因此不一定适合强 multi-agent。

## 7. 常见 Multi-Agent 模式

### 7.1 Orchestrator-Worker

最常见、也最推荐的模式。

特点：

- 一个主 agent 负责规划与协调
- 多个子 agent 各自处理独立子任务
- 主 agent 汇总结果

### 7.2 Router-Specialist

特点：

- 先判断问题类型
- 再路由到不同 specialist agent

适合：

- 专业域明显分层
- 每个子 agent 只处理固定类型任务

### 7.3 Evaluator-Generator / Reviewer-Executor

特点：

- 一个 agent 负责产出
- 另一个 agent 负责评估或修正

适合：

- 高质量生成
- 复杂诊断
- 长输出审查

### 7.4 Hierarchical Multi-Agent

特点：

- 多层 orchestration
- 更复杂的任务树

通常不建议在企业早期阶段使用。

## 8. 近两年最值得借鉴的工程经验

Anthropic 在多智能体研究系统里总结了几个很实用的点。

### 8.1 Orchestrator 必须学会“如何委派”

主 Agent 不应只说一句模糊任务，而应给子 Agent：

- 明确目标
- 输出格式
- 工具范围
- 任务边界

否则子 Agent 很容易重复劳动或走偏。

### 8.2 effort scaling 需要显式设计

简单问题不应生成太多子 Agent，复杂问题才值得更多并行资源。

### 8.3 工具和并行化设计决定速度收益

多智能体如果仍然串行用工具，价值会大打折扣。

### 8.4 可观测性极其重要

Multi-Agent 的调试难度远高于单 Agent，没有 tracing、state inspection 和 task replay 会非常难排查。

## 9. Multi-Agent 的核心组成

### 9.1 Orchestrator

负责：

- 任务拆解
- 子任务分配
- 并行度控制
- 结果汇总
- 再次迭代决策

### 9.2 Worker Agents

负责：

- 独立处理子任务
- 使用自己的上下文和工具
- 输出结构化结果

### 9.3 Coordination Protocol

定义：

- 子任务输入格式
- 子任务输出格式
- 中间状态同步方式
- 错误传播方式

### 9.4 Runtime & State Layer

负责：

- checkpoint
- durable execution
- timeout
- retry
- resume
- replay

## 10. Multi-Agent 与单 Agent 的决策边界

建议优先问以下几个问题：

1. 任务是否可拆成多个相对独立方向。
2. 是否存在足够高的信息量或并行价值。
3. 单 Agent 是否已经在 context / tool routing 上明显吃力。
4. 任务价值是否值得承担更多 token 和工程复杂度。

如果这几条大多不成立，通常不值得做 Multi-Agent。

## 11. 面向低代码平台的适用场景判断

低代码平台里，Multi-Agent 更适合：

- 多页面 / 多模板并行检索比较
- 多 source 运维信息并行收集
- 复杂方案比选与资料汇总

不一定适合：

- 单页面 schema 直接生成
- 简单配置纠错
- 单步组件问答

这些任务通常用单 Agent 或 Workflow-Agent 更稳。

## 12. 面向低代码平台的一个可行模式

一个更可行的模式通常是：

- 主 Agent 负责理解目标并拆任务
- 一个子 Agent 查模板
- 一个子 Agent 查组件协议
- 一个子 Agent 查历史案例
- 主 Agent 汇总并生成最终建议

这类模式在复杂场景下会比一个 Agent 独自检索更清晰。

## 13. 常见误区

### 13.1 认为 Multi-Agent 天然更强

它只在特定问题形态下更强。

### 13.2 一开始就做 agent 之间自由通信网络

这通常会显著放大复杂度和调试难度。

### 13.3 不定义子 Agent 输出契约

最后会很难汇总和验证结果。

### 13.4 忽略成本

Anthropic 的公开经验已经很明确，多智能体系统往往显著更耗 token。

## 14. 一句话总结

Multi-Agent 架构的核心，不是让更多 agent 一起工作，而是只在任务可并行、上下文过大或角色分工明显时，采用 orchestrator-worker 等模式把复杂任务拆给多个具备独立边界的 agent 协同完成；近两年的最佳实践也越来越明确，多智能体的价值成立有条件，工程控制比概念热度更重要。

## 15. 参考资料

- Anthropic, How we built our multi-agent research system: [https://www.anthropic.com/engineering/built-multi-agent-research-system](https://www.anthropic.com/engineering/built-multi-agent-research-system)
- Anthropic, Building Effective AI Agents: [https://www.anthropic.com/engineering/building-effective-agents](https://www.anthropic.com/engineering/building-effective-agents)
- LangChain multi-agent docs: [https://docs.langchain.com/oss/python/langchain/multi-agent](https://docs.langchain.com/oss/python/langchain/multi-agent)
- LangGraph overview: [https://docs.langchain.com/oss/python/langgraph/overview](https://docs.langchain.com/oss/python/langgraph/overview)
