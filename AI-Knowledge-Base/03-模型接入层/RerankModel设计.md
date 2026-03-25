# RerankModel设计

> 状态：第一版草案

## 1. 文档定位

本文档用于定义企业 AI 平台中的 `RerankModel` 设计，重点回答：

- 为什么 rerank 应作为独立模型能力抽象
- rerank 接口应该如何统一设计
- rerank 在 RAG、搜索推荐和 Agent 场景中起什么作用

## 2. 为什么 RerankModel 需要单独抽象

很多系统在做检索时只关注 embedding + vector search，但在生产级系统中，召回后的精排序通常决定最终效果上限。

原因在于：

- 向量召回强调高召回，不强调最终排序绝对最优
- query 和 candidate 的细粒度匹配往往需要更强的交叉建模
- metadata、时效性、业务优先级常常需要进入排序阶段

因此，rerank 不应被当成 retrieval 的小细节，而应视为独立能力模块。

## 3. 行业最佳实战与趋势

### 3.1 生产级 RAG 越来越默认配 rerank

当前主流检索架构已经逐渐形成共识：

- 第一阶段高召回
- 第二阶段高精排

在很多企业知识问答、推荐和代码检索场景中，没有 rerank 的系统通常很难达到稳定质量。

### 3.2 rerank 模型正在从附加件变成标准组件

OpenAI、Cohere、Jina、Qwen、BGE 等路线都在强化 embedding + rerank 的组合能力。这说明行业已经把 rerank 看成标准检索栈的一部分。

### 3.3 排序越来越强调“任务相关性”，而不只是语义相似度

rerank 模型通常更接近“query-candidate relevance modeling”，因此它补的是“检索相关性判断”而不是简单距离计算。

## 4. RerankModel 的职责边界

### 4.1 应负责什么

- 接收 query
- 接收候选列表
- 计算相关性分数
- 返回排序后的候选结果
- 支持 topK 裁剪
- 保留 metadata passthrough

### 4.2 不应负责什么

- 不负责原始召回
- 不负责知识切分
- 不负责向量索引
- 不负责最终回答生成

## 5. 推荐的输入结构

RerankRequest 建议至少包含：

- `query`
- `candidates`
- `topK`
- `taskType`
- `metadataPolicy`（可选）

其中 `candidates` 应允许携带：

- id
- text
- metadata
- source info
- previous recall score

这样 rerank 结果后续可以方便回灌到上下文构建阶段。

## 6. 推荐的输出结构

RerankResponse 建议至少包含：

- `items`
- `score`
- `rank`
- `usage`
- `modelInfo`
- `providerResponseMetadata`

其中每个 item 最好保留原始 candidate metadata。

## 7. 关键设计点

### 7.1 保留 candidate metadata

rerank 不只是排文本，有时还要结合：

- source type
- 更新时间
- 权限范围
- 业务优先级

因此接口层应允许 metadata 贯穿。

### 7.2 topK 裁剪要在模型层支持

这样可以减少上层重复处理，并控制成本。

### 7.3 任务类型应可配置

未来 rerank 模型可能会针对：

- QA
- recommendation
- code retrieval
- multimodal retrieval

表现不同，因此预留 `taskType` 有助于扩展。

### 7.4 batch 行为要明确

需要明确 rerank 是否支持批量 query、候选数上限、性能边界与 timeout 行为。

## 8. 统一接口示意

```ts
interface RerankModel {
  capability: 'rerank';
  modelId: string;
  provider: string;
  features: RerankModelFeatures;
  rerank(request: RerankRequest): Promise<RerankResponse>;
}
```

## 9. Rerank 在整体架构中的位置

### 9.1 在 RAG 中

作用是把 recall 的结果精排为更高质量上下文候选。

### 9.2 在推荐系统中

作用是把初步召回的模板、组件、页面或案例排序为更适合当前场景的结果。

### 9.3 在 Agent 中

作用是对：

- 检索结果
- 候选工具
- 候选案例

做进一步判断，帮助运行时选择更优路径。

## 10. 面向低代码平台如何使用 RerankModel

在低代码平台中，rerank 特别适合用于：

- 组件推荐排序
- 页面模板排序
- 错误案例优先级排序
- Schema 片段候选排序
- 配置建议候选排序

这类场景通常仅靠 embedding 相似度不够，需要更细粒度的 query-candidate 匹配。

## 11. 常见误区

### 11.1 认为 rerank 可有可无

在很多业务场景里，没有 rerank，最终上下文质量会明显下降。

### 11.2 把 rerank 和 embedding 混成一个接口

虽然两者都和检索有关，但职责完全不同。

### 11.3 不保留 metadata

这样会让排序结果很难进入后续上下文治理和业务优先级控制。

## 12. 一句话总结

RerankModel 设计的核心，是把“高召回后的高精度相关性判断”抽象成独立模型能力，让平台能够在 embedding 召回之后，通过更细粒度的 query-candidate 匹配显著提升最终上下文质量与推荐质量。

## 13. 参考资料

- Cohere Rerank docs: [https://docs.cohere.com/docs/rerank-overview](https://docs.cohere.com/docs/rerank-overview)
- Jina reranker collection: [https://huggingface.co/collections/jinaai/rerankers-66f17c9502cc988bffa6fce4](https://huggingface.co/collections/jinaai/rerankers-66f17c9502cc988bffa6fce4)
- BGE reranker docs: [https://bge-model.com/tutorial/5_Reranking/5.1.html](https://bge-model.com/tutorial/5_Reranking/5.1.html)
- OpenAI retrieval guidance: [https://platform.openai.com/docs/guides/retrieval](https://platform.openai.com/docs/guides/retrieval)
