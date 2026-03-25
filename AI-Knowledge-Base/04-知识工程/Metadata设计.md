# Metadata设计

> 状态：第一版草案

## 1. 文档定位

本文档用于定义知识工程中的 metadata 设计，重点回答：

- 为什么 metadata 在企业知识系统中和正文同样重要
- metadata 应该设计哪些层次和字段
- metadata 如何服务检索、权限、治理、评测与后续 Agent 场景

## 2. 为什么 metadata 比很多人想象中更重要

在很多初学型 RAG 系统中，团队会把注意力放在正文内容和 embedding 上，metadata 往往只保留一个标题和来源链接。

但在真实企业系统中，很多关键能力都依赖 metadata 才能成立，包括：

- 权限过滤
- 租户隔离
- 场景限定检索
- 时间新鲜度排序
- sourceType 路由
- 结果引用与解释
- 评测切片分析
- Agent 在行动前做条件判断

可以直接说：

没有 metadata 的企业知识系统，最多只是语义搜索 demo，很难成为可治理的知识基础设施。

## 3. 近两年的关键变化

### 3.1 metadata 正从“附属字段”变成检索控制平面

当前主流知识工程实践越来越强调 metadata-aware retrieval，而不是单纯 dense similarity。

原因是企业场景天然需要：

- 多租户隔离
- sourceType 区分
- 时间过滤
- 对象类型过滤
- 风险与权限过滤

这意味着 metadata 已经不只是补充信息，而是在线检索和治理的核心输入之一。

### 3.2 agentic retrieval 对 metadata 的要求更高

当检索结果不仅服务问答，还服务：

- 结构化生成
- 工具调用前规范查询
- Workflow 节点决策
- 多阶段推理

metadata 就不只是为了过滤，而是为了让系统理解“这是什么类型的知识单元、适合在哪一步使用”。

### 3.3 评测和可观测性越来越依赖 metadata 切片

近两年更成熟的 eval 实践往往会按 metadata 维度切片分析，例如：

- 哪类 sourceType 命中差
- 哪类文档更新时间较老时表现差
- 哪个租户或模块的数据更容易产生幻觉

这说明 metadata 还是质量分析的重要维度。

## 4. metadata 解决什么问题

### 4.1 解决过滤问题

例如：

- 只查某租户
- 只查某应用
- 只查某文档类型
- 只查某时间范围内更新的内容

### 4.2 解决排序问题

例如：

- 新文档优先
- 官方文档优先
- 业务关键知识优先
- 高可信 source 优先

### 4.3 解决治理问题

例如：

- 权限控制
- 审计定位
- 引用追踪
- 生命周期管理

### 4.4 解决场景适配问题

例如：

- 生成场景更偏好 schema/template 类型 chunk
- FAQ 场景更偏好说明文档类型 chunk
- 诊断场景更偏好错误案例和日志规则类型 chunk

## 5. metadata 的推荐分层

建议至少把 metadata 拆成五层。

### 5.1 来源层 metadata

描述它来自哪里。

例如：

- sourceSystem
- sourceType
- sourceUri
- repository
- connectorId

### 5.2 业务层 metadata
n
描述它属于什么业务对象或业务域。

例如：

- bizDomain
- moduleType
- tenantCode
- appCode
- componentName
- pageType
- resourceId

### 5.3 结构层 metadata

描述它在文档或对象中的结构位置。

例如：

- documentId
- chunkId
- titlePath
- sectionId
- pageNumber
- objectType
- fieldPath

### 5.4 治理层 metadata

描述它与权限、生命周期和治理有关的信息。

例如：

- permissionScope
- sensitivityLevel
- updatedAt
- version
- status
- deletedFlag

### 5.5 检索层 metadata

描述它如何参与检索和排序。

例如：

- retrievalTags
- keywords
- embeddingVersion
- chunkStrategy
- rankBoost
- freshnessScore

## 6. 推荐的最小字段集

如果要定义一个企业知识系统的最小 metadata 集，建议至少包含：

- documentId
- chunkId
- sourceType
- title
- titlePath
- bizDomain
- objectType
- resourceId
- tenantCode
- permissionScope
- updatedAt
- version
- tags

这组字段通常足以支撑大多数检索、过滤、治理与引用需求。

## 7. metadata 应如何与 chunk 绑定

metadata 不应只挂在整篇文档上，很多字段必须精确到 chunk 或对象级。

例如：

- 整篇文档权限可能一致
- 但 chunk 的 titlePath、sectionId、objectType 不同
- 某些 chunk 属于组件 props 定义
- 某些 chunk 属于错误案例

因此建议采用：

- 文档级 metadata
- chunk 级 metadata
- 对象级 metadata

三层组合。

## 8. metadata 如何服务 retrieval

### 8.1 pre-filter

在召回前先缩小候选空间，例如：

- tenantCode = 当前租户
- sourceType in [componentDoc, schemaTemplate]
- updatedAt within 6 months

### 8.2 rerank 辅助

在 rerank 阶段可结合：

- freshness
- authority
- business priority
- source trust level

### 8.3 context builder 辅助

在上下文构建阶段可以利用 metadata：

- 去重同 source
- 合并同 titlePath
- 优先保留高可信来源

## 9. metadata 如何服务治理与审计

高质量 metadata 会显著降低治理复杂度。

典型用法包括：

- 根据 permissionScope 做权限过滤
- 根据 tenantCode 做隔离
- 根据 updatedAt 做失效判断
- 根据 sourceUri 做引用跳转
- 根据 embeddingVersion 做索引重建定位

## 10. metadata 如何服务评测

评测系统可以按 metadata 做切片分析，例如：

- sourceType 维度
- 文档新旧维度
- 模块维度
- 租户维度
- objectType 维度

这样可以更精准地判断问题出在哪一类知识对象上，而不只是看整体平均分。

## 11. 面向低代码平台的 metadata 设计建议

在低代码平台场景中，建议重点考虑以下字段：

- componentName
- componentCategory
- pageType
- schemaNodeType
- setterType
- slotName
- dataSourceType
- flowNodeType
- publishStage
- errorCode
- appCode
- tenantCode

这些字段会直接影响：

- 组件推荐
- schema 检索
- 配置纠错
- 运维诊断

## 12. metadata 设计中的常见误区

### 12.1 metadata 只保留 title 和 url

这对企业级检索远远不够。

### 12.2 metadata 粒度太粗

只挂在文档级，后续 chunk 级检索与引用能力会很弱。

### 12.3 metadata 字段不统一

不同数据源各自定义字段，会导致检索过滤和平台治理失控。

### 12.4 metadata 设计与业务对象脱节

如果没有组件、页面、schema、错误案例等领域字段，后续场景效果会明显受限。

## 13. 一句话总结

metadata 设计的核心，不是给知识块补几个附加字段，而是为检索、权限、排序、引用、评测和 agentic use 建立控制面信息；近两年的最佳实践也越来越清楚，企业知识系统的真正可治理性，很大程度上取决于 metadata 体系是否设计到位。

## 14. 参考资料

- OpenAI Retrieval guide: [https://platform.openai.com/docs/guides/retrieval](https://platform.openai.com/docs/guides/retrieval)
- OpenAI File Search guide: [https://platform.openai.com/docs/guides/tools-file-search/](https://platform.openai.com/docs/guides/tools-file-search/)
- LlamaIndex metadata filtering: [https://docs.llamaindex.ai/en/stable/module_guides/storing/vector_stores/](https://docs.llamaindex.ai/en/stable/module_guides/storing/vector_stores/)
- Weaviate metadata filtering: [https://docs.weaviate.io/weaviate/search/filters](https://docs.weaviate.io/weaviate/search/filters)
- Pinecone metadata filtering: [https://docs.pinecone.io/guides/search/filter-by-metadata](https://docs.pinecone.io/guides/search/filter-by-metadata)
