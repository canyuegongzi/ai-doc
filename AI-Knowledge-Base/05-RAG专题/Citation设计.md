# Citation设计

> 状态：第一版草案

## 1. 文档定位

本文档用于定义 RAG 系统中的 Citation 设计，重点回答：

- 为什么 citation 在企业级 RAG 中不是附加体验，而是核心能力
- citation 应该在知识工程、检索链路和生成链路中的哪个阶段设计
- 近两年 agentic retrieval 与结构化 retrieval response 发展后，citation 设计正在发生什么变化

## 2. 为什么 Citation 是企业级 RAG 的关键能力

纯生成系统最常见的问题之一是：

- 说得像对
- 但不知道依据在哪
- 也无法验证或追溯

在企业场景中，这种不可追溯性会迅速放大为治理问题，因为用户通常不仅要“答案”，还要：

- 来源
- 证据
- 适用范围
- 更新时间
- 是否来自官方资料

因此，citation 的价值不是锦上添花，而是：

- 提升可信度
- 提升可解释性
- 支撑审计与回放
- 支撑用户二次验证
- 减少幻觉风险

## 3. 近两年的关键变化

### 3.1 citation 正从“回答里附个链接”走向正式检索响应结构

Azure AI Search 的 agentic retrieval 文档已经明确把 retrieval response 拆成：

- extracted response / grounding data
- reference data
- query activity

这说明最新的 retrieval 系统已经开始把“引用信息”视为检索响应的正式部分，而不是生成后临时拼接。

### 3.2 citation 开始服务 agent，而不只服务最终用户

在 agentic retrieval 场景中，引用信息不仅用来展示，还可能用于：

- 后续步骤决策
- 工具调用前校验
- trace 和 replay
- 执行结果解释

这意味着 citation 不再只是 UI 层功能，而是 retrieval pipeline 的结构化输出之一。

### 3.3 结构化 retrieval response 提高了 citation 设计的重要性

越来越多系统开始把检索结果与 source reference、activity log 一起返回，这使 citation 成为生产级 RAG 的一等公民。

## 4. Citation 解决什么问题

### 4.1 结果可追溯问题

让系统能够回答：

- 这句话来自哪篇文档
- 来自哪个 section / page / block
- 来自哪个对象或字段

### 4.2 结果可验证问题

用户和系统都可以回到原始来源，判断内容是否被错误理解或引用。

### 4.3 权限与审计问题

引用记录能帮助审计：

- 是否用了权限内知识
- 是否使用了过时资料
- 最终回答建立在哪些来源上

## 5. Citation 的职责边界

### 5.1 Citation 应负责什么

- 保留来源定位信息
- 支持回答到来源的映射
- 支持用户查看原文依据
- 支持系统 trace / replay

### 5.2 Citation 不应负责什么

- 不替代检索质量本身
- 不替代权限系统
- 不替代回答生成逻辑

citation 是证据层，不是答案层。

## 6. Citation 信息应从哪里来

citation 不是在最后生成答案时凭空补的，而应贯穿整个链路。

### 6.1 知识工程阶段

应保留：

- documentId
- chunkId
- sourceUri
- titlePath
- pageNumber
- objectPath

### 6.2 Retrieval 阶段

应确保这些信息在 recall、rerank、context builder 后没有丢失。

### 6.3 Answer Generation 阶段

模型或应用层应基于结构化引用信息把答案与来源关联起来。

## 7. 推荐的 citation 最小字段集

建议每条 citation 至少包含：

- citationId
- sourceType
- sourceTitle
- sourceUri
- documentId
- chunkId
- pageNumber / sectionId / objectPath
- snippet（可选）
- updatedAt（可选）

在低代码平台等结构化场景中，建议补充：

- componentName
- schemaNodePath
- errorCaseId
- templateId

## 8. Citation 的常见形态

### 8.1 文档级 citation

只标注文档来源。

优点：

- 实现简单

缺点：

- 粒度粗
- 难以精确定位

### 8.2 section / page 级 citation

指向章节、页码或标题路径。

这是企业文档问答中常见且比较实用的形态。

### 8.3 chunk 级 citation

直接指向当前 chunk。

适合：

- 高精度检索
- RAG trace
- answer-to-evidence mapping

### 8.4 对象级 citation

适合结构化对象，例如：

- 组件定义
- schema 节点
- setter 规则
- 错误案例

这种 citation 对低代码平台场景尤其重要。

## 9. citation 与 context builder 的关系

Context Builder 阶段是 citation 最容易丢失的地方。

原因是这一步会做：

- 去重
- 合并
- 压缩
- 分组

如果没有明确保留 source lineage，最终上下文虽然更干净，但来源映射会断掉。

因此建议：

- 每个上下文片段都附带 source lineage
- 合并后的片段保留多个 citation reference
- 压缩后保留原始来源列表

## 10. citation 与 answer generation 的关系

回答生成阶段通常有两种实现方式。

### 10.1 模型内联引用

让模型在生成时直接引用来源标记，例如 `[1]`、`[2]`。

优点：

- 用户体验直观

缺点：

- 需要更严格的 prompt / post-processing 控制

### 10.2 应用层后处理引用

模型生成主答案，应用层根据 answer spans 和 context provenance 做引用映射。

优点：

- 更稳定
- 更适合高要求场景

### 10.3 推荐实践

企业场景中更稳的方式通常是：

- retrieval pipeline 保留结构化 citation
- 模型生成时尽量利用 citation markers
- 最终由应用层做引用一致性校验和展示

## 11. citation 在 agentic retrieval 中的新角色

在 agentic retrieval 场景中，citation 不只给用户看，还能作为：

- reasoning trace 的一部分
- query planning 验证依据
- tool use 前的知识证据
- 后续 agent step 的 reference object

这意味着 citation 可以被设计成“retrieval evidence object”，而不只是 UI 上的小脚注。

## 12. 面向低代码平台的 citation 设计建议

在低代码平台场景中，citation 应尽量指向结构化对象，而不是只指向长文档。

例如：

- 组件协议条目
- props 定义
- setter 说明
- schema 模板节点
- 错误案例条目
- 发布规范条目

这样用户和系统都更容易理解“这条结论来自哪个平台对象”。

## 13. 常见误区

### 13.1 citation 只在回答层补一条文档链接

这通常不够支撑调试、验证和审计。

### 13.2 context builder 里丢失 source lineage

这是很多系统 citation 做不好的根因。

### 13.3 citation 信息不进入 trace 和 replay

这样后续难以解释一次回答为什么会这样生成。

### 13.4 只做人类可读 citation，不做结构化 citation

现代 RAG 和 agent 场景通常同时需要两种形态。

## 14. 一句话总结

Citation 设计的核心，不是在答案后面贴几个链接，而是把来源定位信息作为 retrieval pipeline 的正式输出，与 chunk、context、answer 和 trace 一起被结构化保留；近两年的最佳实践也越来越明确，高质量 citation 已经是生产级 RAG 的基础能力而不是附属体验。

## 15. 参考资料

- Azure AI Search agentic retrieval concept: [https://learn.microsoft.com/en-us/azure/search/search-agentic-retrieval-concept](https://learn.microsoft.com/en-us/azure/search/search-agentic-retrieval-concept)
- Azure AI Search RAG overview: [https://learn.microsoft.com/en-us/azure/search/retrieval-augmented-generation-overview](https://learn.microsoft.com/en-us/azure/search/retrieval-augmented-generation-overview)
- OpenAI Retrieval guide: [https://platform.openai.com/docs/guides/retrieval](https://platform.openai.com/docs/guides/retrieval)
- OpenAI File Search guide: [https://platform.openai.com/docs/guides/tools-file-search/](https://platform.openai.com/docs/guides/tools-file-search/)
