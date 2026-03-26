# 日志与Trace实现

> 状态：第一版草案

## 1. 文档定位

本文档用于定义 AI 平台中的日志与 Trace 工程实现方案，重点回答：

- 普通日志、结构化事件、Trace Span 在实现上如何协同
- 如何在不引入过度开销的情况下实现链路可观测
- 如何让日志、Trace、审计和评测样本之间形成可关联的数据基础

## 2. 为什么这部分要单独设计

前面的观测文档解释了 Trace 的作用，但工程实现还需要继续回答：

- 什么信息记日志，什么信息进 span
- span 如何关联 requestId / traceId / taskId
- 日志和 trace 如何兼顾脱敏、性能和排障价值

如果这一层设计不好，常见后果是：

- 日志很多，但定位问题仍然困难
- Trace 很重，线上成本过高
- 审计、反馈、评测无法回链到真实运行轨迹

用一句话概括：

日志与 Trace 实现的本质，是为 AI 平台提供一套低噪声、可关联、可分层的线上事实记录机制。

## 3. 推荐的观测实现分层

建议至少分成三层：

### 3.1 普通结构化日志

用于记录局部运行信息和错误上下文。

### 3.2 Trace Span

用于记录一次请求的端到端链路。

### 3.3 关键治理事件

用于记录：

- approval
- policy block
- risk escalation
- retry / degrade decision

## 4. 普通日志建议记录什么

普通日志适合记录：

- 模块启动与配置加载
- 外部依赖连接状态
- 非关键但有诊断价值的错误信息
- 本地模块级调试信息

建议采用结构化 JSON 日志，而不是纯文本拼接。

建议字段至少包含：

- `timestamp`
- `level`
- `service`
- `module`
- `requestId?`
- `traceId?`
- `taskId?`
- `message`
- `details?`

## 5. Trace Span 建议记录什么

Trace Span 更强调链路级信息，建议记录：

- span 类型
- 父子关系
- 输入摘要
- 输出摘要
- latency
- cost metrics
- retry count
- risk flags
- error type

要点是：

- 重点在“可解释链路”
- 不要求记录所有原始内容
- 重点记录足以定位问题的结构化摘要

## 6. 推荐的链路关联 ID

建议统一传播以下关键 ID：

- `requestId`
- `traceId`
- `taskId`
- `tenantId`
- `userId`
- `sceneType`

其中：

- `requestId` 偏入口请求
- `traceId` 偏整条链路
- `taskId` 偏长运行任务

日志和 Trace 至少应共享前 3 个核心 ID。

## 7. 在 RAG 中的实现建议

建议至少为以下节点建 span：

- query rewrite
- recall
- rerank
- context build
- answer generation

日志则只补充：

- 异常详情
- timeout 原因
- fallback 路径说明

## 8. 在 Tool / Agent 中的实现建议

建议至少为以下节点建 span：

- planner decision
- tool selection
- tool execution
- validation
- HITL
- approval
- final action / result

对于普通日志，建议侧重：

- handler 异常
- 下游依赖错误
- 本地策略判断失败

## 9. 采样与保留策略建议

### 9.1 Trace 采样

不是所有请求都需要全量高细度 span。

可考虑：

- 核心业务全量
- 普通场景采样
- 失败与高风险请求强制保留

### 9.2 日志保留

普通日志可较短保留；
高风险运行日志、审批日志、关键失败链路可延长保留周期。

### 9.3 原始 payload 保留策略

大对象、敏感数据、原始上下文建议：

- 不直接入日志
- 只保留 ref 或 sanitized snapshot

## 10. 面向低代码平台的具体建议

### 10.1 组件问答

Trace 应能看到：

- 用了哪个知识源
- 是否有 citation
- 是否命中 FAQ 缓存

### 10.2 Schema 生成

Trace 应能看到：

- 召回哪些模板
- 生成用了哪个模型
- 校验失败点在哪里

### 10.3 配置诊断

Trace 应能看到：

- 哪个规则命中
- 哪个工具执行失败
- 是否进入人工确认

### 10.4 发布与运维动作

Trace 应能看到：

- risk check
- approval
- action receipt
- verify result

## 11. 实现时的脱敏与安全要求

### 11.1 不在日志中记录明文密钥、token、prompt 原文

### 11.2 Trace 输出做字段级摘要化

### 11.3 审计与 Trace 查询都必须带租户和角色控制

### 11.4 失败样本导出前再次做 redaction

## 12. 常见误区

### 12.1 什么都记日志，什么都不做结构化

噪声会很高，后期难以分析。

### 12.2 只做日志，不做链路 Trace

复杂问题几乎无法定位。

### 12.3 把所有原始上下文都塞进 Span

成本和泄露风险都会快速上升。

### 12.4 Trace 与反馈、评测、审计不共享 ID

无法形成真正闭环。

### 12.5 没有采样与保留策略

上线后存储成本和查询成本会迅速升高。

## 13. 一句话总结

日志与 Trace 实现的核心，是把模块级结构化日志、链路级 Span 和关键治理事件分层记录，并通过统一 ID、摘要化 payload 和采样保留策略构成可观测底座；对 AI 平台来说，真正有价值的不是“记录越多越好”，而是“记录得足够可关联、可诊断、可治理”。
