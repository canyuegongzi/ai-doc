# Hybrid-Search

> 状态：第一版草案

## 1. 文档定位

本文档用于定义 RAG 检索链路中的 Hybrid Search 设计，重点回答：

- 为什么生产级 RAG 越来越不应只依赖单一路径检索
- dense、sparse、metadata、semantic ranker 等信号应如何组合
- 近两年 hybrid search 为什么逐渐从“优化项”变成“默认项”

## 2. 为什么 Hybrid Search 已经接近生产级 RAG 默认实践

在企业知识系统中，用户查询往往同时包含两种信号：

- 语义表达信号
- 精确术语信号

例如：

- “审批流表单里这个 slot 为什么不生效”
- “error code 1024 对应什么问题”
- “查询 dataSource 配置里的分页参数”

这类问题如果只用 dense 检索，容易丢失：

- 错误码
- 组件名
- 字段名
- API path
- 特定术语

如果只用 sparse / keyword 检索，又容易丢失：

- 同义表达
- 模糊意图
- 场景化描述
- 领域隐式关联

因此，hybrid search 在真实企业场景中往往更稳。

## 3. 近两年的关键变化

### 3.1 hybrid search 已从“可选优化”走向“主流推荐”

Azure AI Search 2026 年 2 月更新的 hybrid search 文档已经明确把 hybrid query 描述为并行运行 full-text search 和 vector search，并通过 RRF 合并结果。它还直接指出：在很多基准和真实数据集上，hybrid retrieval with semantic ranker 通常优于单一路径检索。

这说明行业已经形成较清晰共识：

hybrid 不是复杂玩法，而是高质量 RAG 的常见默认起点。

### 3.2 hybrid search 正在和 semantic ranking / rerank 深度结合

现代 hybrid search 已不只是“向量结果 + BM25 结果拼一下”，而是越来越强调：

- 并行召回
- RRF 融合
- semantic ranking 或 cross-encoder rerank
- metadata filtering
- multi-vector query support

### 3.3 agentic retrieval 让 hybrid 的价值进一步扩大

在 multi-query、parallel retrieval 场景中，每个子查询都可能同时走：

- lexical path
- vector path
- metadata constrained path

这让 hybrid search 从单请求优化，进一步升级为多路召回架构基础。

## 4. Hybrid Search 的本质

Hybrid Search 的本质，是在同一次检索中把不同类型的相关性信号并行组合，从而构造一个覆盖更高、精度更稳的候选池。

用一句话概括：

Hybrid Search 不是简单叠加两种检索，而是让语义信号、关键词信号和结构化过滤信号共同参与召回。

## 5. Hybrid Search 解决什么问题

### 5.1 dense 漏掉精确术语

例如：

- 组件名
- 错误码
- SQL 字段名
- API 路径
- 版本号

### 5.2 sparse 难理解语义表达

例如：

- 同义问题
- 业务化说法
- 模糊描述
- 场景化目标

### 5.3 复杂企业知识同时存在两类信号

企业知识天然混合：

- 文档语言
- 配置语法
- 编号与术语
- 结构化元数据

Hybrid Search 可以同时利用这些信号。

## 6. Hybrid Search 的基本组成

### 6.1 Dense Vector Retrieval

负责语义召回。

### 6.2 Sparse / Keyword Retrieval

负责 lexical recall，例如 BM25、full-text search。

### 6.3 Metadata Filtering

负责：

- tenant
- sourceType
- objectType
- updatedAt
- permissionScope

### 6.4 Fusion 策略

负责把不同召回路径的结果合并成统一候选池。

### 6.5 Optional Rerank / Semantic Ranking

在融合后进一步精排。

## 7. 常见的融合方式

### 7.1 Score fusion

直接按分数做线性或加权组合。

问题：

- 不同检索路径的分数空间通常不一致
- 难以稳定调参

### 7.2 Rank fusion

按排序位置融合，而不是直接比原始分数。

其中最典型的就是 RRF（Reciprocal Rank Fusion）。

### 7.3 RRF 为什么重要

Azure AI Search 的 hybrid search 和多查询融合文档明确采用 RRF。RRF 的优势在于：

- 不要求不同检索器分数可直接比较
- 更稳健
- 对多路召回结果融合更友好

## 8. 为什么企业场景尤其需要 Hybrid Search

### 8.1 术语密集

企业内部常有大量：

- 专有名词
- 模块代号
- 错误码
- 字段名
- 页面名
- 组件名

### 8.2 数据源异构

同一查询可能需要在：

- FAQ 文档
- 结构化对象摘要
- 页面 Schema
- 错误案例
- 运维日志

中寻找候选。

### 8.3 场景目标复杂

很多问题不是“解释概念”，而是要支持：

- 诊断
- 推荐
- 生成约束
- 执行前查规范

这类场景通常比单一问答更依赖 hybrid recall。

## 9. Hybrid Search 的推荐架构

### 9.1 Parallel Retrieval

对同一 query 并行执行：

- dense recall
- sparse recall
- metadata constrained recall

### 9.2 Candidate Merge

使用 RRF 或其他融合策略合并候选。

### 9.3 Rerank / Semantic Ranking

对融合结果做进一步排序。

### 9.4 Context Builder

对最终候选做压缩与组装。

这说明：

Hybrid Search 更适合作为 recall 阶段的主框架，而不是最终答案阶段。

## 10. 与 Query Rewrite 的关系

Hybrid Search 对 query 质量要求更高，而不是更低。

原因是：

- dense 路径需要语义清晰 query
- sparse 路径需要 lexical signal
- metadata 路径需要显式实体和过滤条件

因此 rewrite 很多时候还要刻意保留关键词和实体，而不是过度自然语言化。

## 11. 与 Rerank 的关系

Hybrid Search 主要解决候选覆盖问题，Rerank 主要解决候选排序问题。

生产级链路通常更像：

query rewrite -> hybrid recall -> rerank -> context builder

而不是在 hybrid 之后直接把 topK 塞给模型。

## 12. 面向低代码平台的 hybrid 典型形态

### 12.1 组件知识检索

- dense：组件功能说明语义匹配
- sparse：组件名、props 名、setter 名、slot 名
- metadata：componentCategory、bizDomain、tenant

### 12.2 Schema 模板检索

- dense：页面目标与模板描述语义匹配
- sparse：页面类型、模块名、节点名
- metadata：pageType、appCode、templateTag

### 12.3 配置诊断

- dense：错误描述与历史案例匹配
- sparse：错误码、字段名、组件名
- metadata：moduleType、runtimeStage、errorCategory

## 13. 常见误区

### 13.1 Hybrid 就是把 dense 和 sparse 简单拼一起

真正有效的 hybrid 需要考虑 fusion、filter、权重、去重和后续 rerank。

### 13.2 只做 hybrid，不做 metadata filter

这样候选池仍然可能过脏。

### 13.3 认为 hybrid 一定更贵，因此不值得

实际企业场景中，hybrid 带来的召回质量提升往往远大于多一点召回成本。

### 13.4 只在检索层考虑 hybrid，不与场景结合

不同场景应有不同 hybrid 配方，而不是一套固定策略走天下。

## 14. 一句话总结

Hybrid Search 的核心，不是把多种检索方式机械叠加，而是让 dense、sparse、metadata 和后续 semantic ranking / rerank 共同形成一个更稳、更广、更适合企业知识场景的召回框架；近两年的最佳实践也越来越明确，生产级 RAG 的默认形态通常就是 hybrid，而不是单一路径检索。

## 15. 参考资料

- Azure AI Search hybrid search overview: [https://learn.microsoft.com/en-us/azure/search/hybrid-search-overview](https://learn.microsoft.com/en-us/azure/search/hybrid-search-overview)
- Azure AI Search hybrid query guide: [https://learn.microsoft.com/en-us/azure/search/hybrid-search-how-to-query](https://learn.microsoft.com/en-us/azure/search/hybrid-search-how-to-query)
- Azure AI Search RRF ranking: [https://learn.microsoft.com/en-us/azure/search/hybrid-search-ranking](https://learn.microsoft.com/en-us/azure/search/hybrid-search-ranking)
- Weaviate hybrid search docs: [https://docs.weaviate.io/weaviate/concepts/search/hybrid-search](https://docs.weaviate.io/weaviate/concepts/search/hybrid-search)
- Pinecone hybrid search docs: [https://docs.pinecone.io/guides/data/encode-sparse-vectors](https://docs.pinecone.io/guides/data/encode-sparse-vectors)
