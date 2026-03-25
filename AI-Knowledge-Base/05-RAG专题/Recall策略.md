# Recall策略

> 状态：第一版草案

## 1. 文档定位

本文档用于定义 RAG 在线链路中的 Recall 策略，重点回答：

- recall 在 RAG 系统中到底承担什么角色
- 为什么生产级 RAG 几乎不应只靠单一路径召回
- dense、sparse、hybrid、metadata filter、多路召回应如何组合
- 近两年的 agentic retrieval 和 hybrid search 发展，对 recall 架构提出了哪些新要求

## 2. 为什么 Recall 是 RAG 的第一决定因素

RAG 系统中有一个很朴素但非常重要的事实：

如果正确知识没有被召回，后续再强的 rerank、context builder 和 LLM 也很难补救。

因此，recall 是整个 RAG 链路中的第一上限。

很多效果问题表面上看是“模型回答错了”，本质上往往是：

- 没召回到正确内容
- 召回过窄
- 召回被噪声淹没
- 多个知识源没有被一起覆盖

## 3. 近两年的关键变化

### 3.1 hybrid recall 已经越来越接近默认实践

Azure AI Search、Weaviate、Pinecone 等近两年的文档都在明显强化 hybrid search，这说明生产级 RAG 已越来越不推荐只用 dense 或只用 keyword。

### 3.2 多路召回正在取代单路召回

当前更成熟的企业系统常常会组合：

- dense recall
- sparse / keyword recall
- metadata filter recall
- source-specific recall
- graph / relation recall
- query decomposition 后的 parallel recall

### 3.3 agentic retrieval 推动 recall 走向 query-planned parallel retrieval

在复杂问题场景中，recall 已经不再是“单 query 打一次索引”，而可能是：

- 多子查询并行召回
- 多源并行召回
- 分意图并行召回

这会显著改变 recall 策略设计方式。

## 4. Recall 的职责边界

### 4.1 应负责什么

- 基于 query 找出相关候选
- 最大化相关内容覆盖率
- 为 rerank 提供高质量候选池

### 4.2 不应负责什么

- 不负责最终排序最优
- 不负责上下文压缩
- 不负责最终回答生成

这些属于 rerank、context builder 和 answer generation。

## 5. 常见 recall 方式

### 5.1 Dense Recall

基于 embedding 语义相似度召回。

优点：

- 擅长语义表达
- 能覆盖同义与变体表达

缺点：

- 对精确术语、错误码、编号不稳定
- 容易召回语义相近但不够精确内容

### 5.2 Sparse / Keyword Recall

基于关键词、BM25 或类似 lexical 信号召回。

优点：

- 精确术语命中强
- 对编号、字段名、路径、错误码友好

缺点：

- 对模糊语义表达覆盖弱

### 5.3 Metadata-filtered Recall

基于：

- tenant
- app
- sourceType
- objectType
- updatedAt
- tags

先缩小召回空间。

### 5.4 Source-aware Recall

对不同 sourceType 采用不同召回路径，例如：

- FAQ 走文档索引
- 组件问题走组件索引
- 错误诊断走案例和日志索引

### 5.5 Multi-query Recall

对一个复杂问题生成多个子查询并行召回。

### 5.6 Relation / Graph-aware Recall

对存在关系链的知识对象，基于对象关系做补充召回。

## 6. 为什么 hybrid recall 越来越重要

企业知识中常常同时存在：

- 自然语言描述
- 专有术语
- 字段名
- 错误码
- API path
- 组件名
- schema 节点名

这决定了单一 dense 或单一 sparse 都很难稳定覆盖全部需求。

因此，hybrid recall 的目标不是“多堆几种方法”，而是让不同信号在候选池阶段互补。

## 7. 推荐的 recall 架构模式

### 7.1 classic hybrid recall

适用于：

- 大多数 FAQ 和企业文档场景

模式：

- dense + sparse + metadata filter

### 7.2 object-aware recall

适用于：

- 组件库
- 页面模板
- schema 样本

模式：

- objectType routing + dense / sparse recall

### 7.3 multi-query parallel recall

适用于：

- 复杂复合问题
- agentic retrieval
- 多源知识查询

模式：

- query decomposition + parallel recall + merge

### 7.4 case-augmented recall

适用于：

- 配置诊断
- 运维分析
- 错误修复建议

模式：

- 文档规范 recall + 历史案例 recall + 错误码 lexical recall

## 8. recall 设计中的关键参数

### 8.1 topK

召回过少会漏掉正确结果，召回过多会给 rerank 和 context builder 增加噪声与成本。

### 8.2 recall pool size

对于 multi-stage retrieval，第一阶段候选池往往比最终返回数量大得多。

### 8.3 query weighting

hybrid recall 中，不同信号可赋予不同权重。

### 8.4 source weighting

某些 sourceType 可赋予更高优先级，例如官方文档、最新规范、组件协议。

## 9. recall 与 rerank 的关系

Recall 和 rerank 必须一起设计。

可以简单理解为：

- recall 追求“不漏掉该有的内容”
- rerank 追求“把最对的内容排到最前面”

如果 recall 太弱，rerank 没有材料；如果 recall 太脏，rerank 压力会非常大。

## 10. recall 与 query rewrite 的关系

现代 RAG 中，query rewrite 和 recall 经常应被视为同一子系统的上下游：

- rewrite 决定怎么查
- recall 决定查哪些候选

两者分开优化很容易失真，最好联动评测。

## 11. 面向低代码平台的 recall 策略建议

### 11.1 组件知识问答

建议：

- 组件名 sparse recall
- 组件说明 dense recall
- sourceType / objectType filter

### 11.2 Schema 生成增强

建议：

- 页面模板 recall
- 组件协议 recall
- 数据源定义 recall
- generation-specific chunk recall

### 11.3 配置诊断

建议：

- 错误码 lexical recall
- 错误案例 dense recall
- 规则文档 recall
- 当前组件 / schema object routing

### 11.4 运维辅助

建议：

- 日志 keyword recall
- 运维规范 dense recall
- 发布记录 metadata filter

## 12. 常见误区

### 12.1 只做 dense recall

这在真实企业场景里通常不够稳。

### 12.2 recall 不做 sourceType / objectType 分流

会造成候选池噪声过高。

### 12.3 recall topK 越大越好

过大的候选池会增加 rerank 成本和上下文污染风险。

### 12.4 不把权限和 metadata 纳入 recall

这会让系统既不安全也不高效。

## 13. 一句话总结

Recall 策略的核心，是在 query 理解之后，用 dense、sparse、metadata、source-aware 和 multi-query 等多种手段构建一个高覆盖、低噪声、可治理的候选池；近两年的最佳实践也越来越明确，生产级 RAG 的 recall 几乎总是 hybrid、multi-stage、task-aware 的，而不应再停留在单路 topK 向量召回。 

## 14. 参考资料

- Azure AI Search RAG overview: [https://learn.microsoft.com/en-us/azure/search/retrieval-augmented-generation-overview](https://learn.microsoft.com/en-us/azure/search/retrieval-augmented-generation-overview)
- Azure AI Search agentic retrieval: [https://learn.microsoft.com/en-us/azure/search/search-agentic-retrieval-concept](https://learn.microsoft.com/en-us/azure/search/search-agentic-retrieval-concept)
- Weaviate hybrid search docs: [https://docs.weaviate.io/weaviate/concepts/search/hybrid-search](https://docs.weaviate.io/weaviate/concepts/search/hybrid-search)
- Pinecone hybrid search docs: [https://docs.pinecone.io/guides/data/encode-sparse-vectors](https://docs.pinecone.io/guides/data/encode-sparse-vectors)
- Cohere Agentic RAG tutorial: [https://docs.cohere.com/v2/docs/agentic-rag](https://docs.cohere.com/v2/docs/agentic-rag)
- OpenAI Retrieval guide: [https://platform.openai.com/docs/guides/retrieval](https://platform.openai.com/docs/guides/retrieval)
