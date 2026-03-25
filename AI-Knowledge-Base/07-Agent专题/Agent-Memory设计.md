# Agent-Memory设计

> 状态：第一版草案

## 1. 文档定位

本文档用于定义 Agent 系统中的 Memory 设计，重点回答：

- Agent 为什么需要独立的 memory 设计，而不能只依赖对话历史
- 不同层次的 memory 应如何分工
- 近两年 agent 实践中，memory 为什么越来越从“聊天上下文”演进成运行时基础设施

## 2. 为什么 Agent 不能只靠聊天上下文

在简单聊天场景中，历史消息可能已经足够。

但在 Agent 场景中，任务通常更复杂：

- 会跨多个步骤
- 会调用多个工具
- 会经历失败与重试
- 可能跨较长时间运行
- 可能插入人工确认

如果只靠原始对话历史，常见问题包括：

- 上下文膨胀
- 关键信息难以精确恢复
- 中间状态难以结构化复用
- 跨任务信息难以管理

这说明 Agent 需要显式 memory 设计，而不是把一切塞进 prompt。

## 3. 近两年的关键变化

### 3.1 memory 正在从“消息历史”转向“状态化运行资产”

LangGraph、Anthropic 和越来越多 agent 系统都在强调：

- memory 不只是上下文
- memory 还包括任务状态
- plan checkpoints
- 工具结果
- long-running process context

### 3.2 长时任务让 memory 变成 durability 的组成部分

Anthropic 在 2025 年多智能体研究系统中明确提到：

当 lead agent 处理长任务时，需要把 plan 写入 memory，以防上下文窗口截断。

这说明 memory 已直接成为系统延续性的关键设施。

### 3.3 memory 与 retrieval 越来越耦合

当前 agent 系统中的 memory 既可能是：

- 短期状态
- 结构化任务记录
- 可检索的长期记忆

这使 memory 与 RAG / retrieval 的边界开始交叉。

## 4. Agent Memory 解决什么问题

### 4.1 状态延续问题

让任务可以跨步骤、跨轮次、跨时间持续推进。

### 4.2 上下文选择问题

让系统只把当前真正需要的信息带入模型，而不是全部历史。

### 4.3 长期经验积累问题

让系统能保存：

- 常用偏好
- 历史方案
- 案例
- 任务结果

### 4.4 失败恢复问题

出错后可以从已有状态恢复，而不是从零开始。

## 5. 推荐的 memory 分层

建议至少分为四层。

### 5.1 会话级 memory

保存当前对话中必要上下文。

### 5.2 任务级 memory

保存当前任务的：

- goal
- plan
- current state
- tool results
- intermediate outputs

### 5.3 长期用户 / 系统 memory

保存：

- 偏好
- 常用配置
- 历史上下文摘要
- 持续有效的个人 / 系统信息

### 5.4 检索型 memory

将历史任务、案例、计划摘要等对象化后做检索，用于支持未来任务。

## 6. 推荐的 memory 对象类型

Agent 系统中，建议显式区分以下对象：

- conversation summary
- task state snapshot
- plan checkpoint
- tool result record
- human feedback record
- final outcome summary
- reusable case memory

这比把所有内容都当成一串文本更适合工程化。

## 7. 写入策略建议

不是所有内容都应写入 memory。

建议重点写入：

- 高价值中间结果
- 工具调用结果摘要
- 计划节点
- 人工确认结果
- 最终结论
- 可复用失败经验

不建议无差别写入所有原始 token。

## 8. 读取策略建议

同样，不应每次把所有 memory 都读进 prompt。

更推荐：

- 当前任务默认读任务级 memory
- 只有在需要时检索长期 memory
- 通过 metadata / taskType / relevance 做 memory retrieval

## 9. memory 与 RAG 的关系

可以简单区分：

- RAG 更偏外部知识检索
- Agent memory 更偏任务和运行状态延续

但在高级系统中，两者会逐渐融合，例如：

- 历史任务案例被对象化后成为可检索 memory
- memory retrieval 成为 retrieval pipeline 的一部分

## 10. memory 与状态机的关系

状态机定义当前处于哪一步，memory 负责保存这一步以及之前发生了什么。

可以理解为：

- state machine = 生命周期骨架
- memory = 生命周期内容

## 11. 面向低代码平台的 memory 设计建议

### 11.1 页面生成 Agent

建议保存：

- 当前页面目标
- 已选模板
- 已选组件
- 校验失败记录
- 修复历史

### 11.2 配置诊断 Agent

建议保存：

- 当前问题描述
- 已查规则
- 已查案例
- 工具检查结果
- 最终修复建议

### 11.3 运维辅助 Agent

建议保存：

- 当前事件摘要
- 已查询日志范围
- 已命中规范
- 是否已进入人工处理

## 12. 常见误区

### 12.1 把全部聊天记录当成完整 memory

### 12.2 没有任务级结构化 memory

### 12.3 过度写入，导致 memory 膨胀和噪声积累

### 12.4 没有生命周期管理，旧 memory 永远不失效

## 13. 一句话总结

Agent-Memory 设计的核心，是把任务状态、关键中间结果、长期偏好和可检索历史经验从原始聊天上下文中抽离出来，形成可写入、可检索、可恢复、可治理的运行时记忆体系；近两年的最佳实践也越来越明确，memory 已经不是聊天附属物，而是 agent runtime 的正式基础设施。

## 14. 参考资料

- Anthropic, How we built our multi-agent research system: [https://www.anthropic.com/engineering/built-multi-agent-research-system](https://www.anthropic.com/engineering/built-multi-agent-research-system)
- LangGraph overview: [https://docs.langchain.com/oss/python/langgraph/overview](https://docs.langchain.com/oss/python/langgraph/overview)
- LangGraph durable execution: [https://docs.langchain.com/oss/python/langgraph/durable-execution](https://docs.langchain.com/oss/python/langgraph/durable-execution)
- OpenAI, A practical guide to building agents: [https://openai.com/business/guides-and-resources/a-practical-guide-to-building-ai-agents/](https://openai.com/business/guides-and-resources/a-practical-guide-to-building-ai-agents/)
