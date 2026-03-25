# Rerank策略

> 状态：第一版草案

## 1. 文档定位

本文档用于定义 RAG 在线链路中的 Rerank 策略，重点回答：

- 为什么 recall 之后必须做 rerank
- semantic ranker、cross-encoder reranker、业务加权排序应如何协同
- 近两年生产级 RAG 为什么越来越把 rerank 当成标准组件

## 2. 为什么 Rerank 是现代 RAG 的标准环节

Recall 的目标是：

- 召回尽可能覆盖正确候选

但 recall 的代价是：

- 候选池通常会带来噪声
- 排名往往不够精细
- dense / sparse / hybrid 的初始顺序不一定适合直接给模型

因此需要 rerank 对候选池做更精细的相关性判断。

可以简单理解为：

- recall 负责别漏掉
- rerank 负责别排错

## 3. 近两年的关键变化

### 3.1 rerank 已从“可选优化”变成“生产默认件”

Cohere、Jina、BGE、Qwen、Azure semantic ranker 等路线都说明，当前行业对 rerank 的态度已经很明确：

没有 rerank 的企业 RAG 系统，很难长期保持稳定检索质量。

### 3.2 排序越来越重视 query-candidate relevance，而不是只看 embedding 分数

embedding recall 更像高覆盖初筛；而 rerank 越来越承担“真正判断这个候选是否最适合当前 query”的职责。

### 3.3 rerank 开始与 metadata、时效性和业务优先级结合

成熟系统不再只依赖模型分数，还会引入：

- freshness boost
- authority boost
- source trust
- biz priority
- tenant preference

这使 rerank 越来越成为“语义判断 + 业务排序”的结合层。

## 4. Rerank 解决什么问题

### 4.1 dense / hybrid recall 后的排序噪声

候选内容可能相关，但不一定最值得进入最终上下文。

### 4.2 多路召回结果合并后的排序混乱

hybrid / multi-query 召回后，初始顺序通常难以直接使用。

### 4.3 上下文预算有限

模型上下文有限，必须优先保留最有价值候选。

## 5. 常见 rerank 方式

### 5.1 Cross-encoder / dedicated reranker

优点：

- 精排序效果通常更好
- 直接判断 query 与 candidate 的相关性

缺点：

- 成本更高
- 延迟更高

### 5.2 Semantic ranker

例如搜索引擎内置的 semantic ranker。

优点：

- 与检索引擎结合紧密
- 工程接入简单

### 5.3 LLM-as-reranker

优点：

- 灵活
- 适合复杂多因素判断

缺点：

- 成本高
- 稳定性和吞吐不一定适合大规模在线场景

### 5.4 规则与业务加权 rerank

例如：

- 官方文档加权
- 最新版本加权
- 高可信 source 加权

## 6. 推荐的 Rerank 策略结构

生产级系统通常可采用以下组合：

1. hybrid recall / multi-query recall
2. semantic or model rerank
3. metadata / biz boost
4. de-dup / diversity control
5. context builder

这说明 rerank 不是一个单点模型，而是一层排序策略。

## 7. rerank 的核心输入

建议至少包括：

- query
- rewrittenQuery
- candidates
- candidate metadata
- sceneType
- rerank objective

这样做的原因是：

不同场景对“最优候选”的定义可能并不相同。

## 8. rerank 的核心输出

建议至少输出：

- reranked candidates
- score
- rank
- optional explanation
- metadata passthrough

如果支持 tracing，最好保留 rerank 版本和策略信息。

## 9. rerank 中为什么要显式保留 metadata

很多企业检索任务并不是单纯看语义最相关，而是还要考虑：

- 更新时间
- sourceType
- 权限范围
- 版本状态
- 官方 / 非官方来源

因此，rerank 阶段通常应支持“模型分数 + metadata 调整”联合工作。

## 10. rerank 与 context builder 的关系

rerank 输出的是“更好的候选顺序”，但 context builder 仍要进一步决定：

- 最终保留哪些
- 是否合并相近块
- 是否去重
- 是否平衡多个 source

因此 rerank 虽重要，但不等于最终上下文选择。

## 11. rerank 在复杂 RAG 中的进一步演化

在 multi-query / agentic retrieval 场景中，rerank 还可能承担：

- 跨查询结果融合排序
- 跨 source 排序
- 多阶段 rerank
- query-specific 或 task-specific rerank

这意味着 rerank 正在从“单 query 单轮精排”升级为更复杂的 relevance orchestration。

## 12. 面向低代码平台的 rerank 策略建议

### 12.1 组件问答

优先项：

- 组件协议正文
- 组件 props / setter 定义
- 官方组件文档

### 12.2 Schema 生成增强

优先项：

- 同类页面模板
- 结构更完整的 schema 样本
- 与当前场景最接近的组件组合

### 12.3 配置诊断

优先项：

- 同错误码案例
- 同组件类型案例
- 最新规则文档
- 相关日志解释条目

## 13. 常见误区

### 13.1 认为 recall 排前面的结果已经够用了

很多时候初始排序并不适合直接进入模型上下文。

### 13.2 rerank 只看模型分数，不看业务信号

这样会让排序结果在企业场景中缺少稳定性。

### 13.3 rerank 后不做去重和多样性控制

最终上下文可能出现大量同质块。

### 13.4 把 rerank 当成最后一步

实际上 rerank 只是 context builder 的上游。

## 14. 一句话总结

Rerank 策略的核心，是在 recall 候选池基础上，通过语义精排、业务加权和多源排序控制，把“可能相关”的内容筛到“最值得进入最终上下文”的水平；近两年的最佳实践也越来越清楚，生产级 RAG 几乎都应把 rerank 视为标准组件，而不是附加优化项。

## 15. 参考资料

- Cohere Rerank overview: [https://docs.cohere.com/docs/rerank-overview](https://docs.cohere.com/docs/rerank-overview)
- Azure semantic ranking: [https://learn.microsoft.com/en-us/azure/search/semantic-search-overview](https://learn.microsoft.com/en-us/azure/search/semantic-search-overview)
- Azure AI Search hybrid ranking (RRF): [https://learn.microsoft.com/en-us/azure/search/hybrid-search-ranking](https://learn.microsoft.com/en-us/azure/search/hybrid-search-ranking)
- Jina reranker collection: [https://huggingface.co/collections/jinaai/rerankers-66f17c9502cc988bffa6fce4](https://huggingface.co/collections/jinaai/rerankers-66f17c9502cc988bffa6fce4)
- BGE reranker docs: [https://bge-model.com/tutorial/5_Reranking/5.1.html](https://bge-model.com/tutorial/5_Reranking/5.1.html)
