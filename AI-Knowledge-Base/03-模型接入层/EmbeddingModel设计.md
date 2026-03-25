# EmbeddingModel设计

> 状态：第一版草案

## 1. 文档定位

本文档用于定义企业 AI 平台中的 `EmbeddingModel` 设计，重点回答：

- embedding 模型在平台中应该如何统一抽象
- query/document、维度裁剪、多语言、多模态、多向量等差异如何表达
- embedding 模型接口应如何服务 RAG、推荐和检索系统

## 2. 为什么 EmbeddingModel 需要独立设计

Embedding 模型常被误以为只是一个“把文本转向量”的简单接口，但在企业平台中，它直接影响：

- RAG 召回质量
- 检索成本
- 多语言效果
- 向量库存储成本
- 长文档处理方式
- 多模态检索能力

因此，EmbeddingModel 不应作为 ChatModel 的附属方法，而应作为独立能力抽象。

## 3. 当前行业最佳实战与趋势

### 3.1 embedding 已从通用文本向量走向任务化能力

当前主流模型越来越区分：

- query embedding
- document embedding
- code embedding
- multilingual embedding
- multimodal embedding
- multi-vector embedding

这意味着平台接口必须能表达 task-aware 和 modality-aware 的差异。

### 3.2 维度、长度和模式越来越成为一等参数

OpenAI、Voyage、Cohere、Jina、BGE 等近年的路线都在强调：

- 维度裁剪
- query/document distinction
- 长上下文支持
- 多模态输入
- dense / multi-vector 混合输出

统一抽象时必须考虑这些变化。

## 4. EmbeddingModel 的职责边界

### 4.1 应负责什么

- 接收文本、代码或多模态输入
- 根据任务类型生成向量表示
- 返回向量及其 metadata
- 支持维度与模式控制
- 支持 batch 调用

### 4.2 不应负责什么

- 不直接负责相似度搜索
- 不直接负责召回排序
- 不直接负责文档切分
- 不直接负责向量库存储

这些职责属于 Retrieval Engine、Knowledge Pipeline 或 Vector Store。

## 5. 推荐的输入结构

建议 EmbeddingRequest 至少包含：

- `inputs`: 输入内容列表
- `inputType`: query / document / code / multimodal
- `dimensions`: 可选目标维度
- `embeddingMode`: single-vector / multi-vector
- `languageHint`: 可选语言提示
- `metadata`: 调用侧附加信息

如果支持多模态，还应允许：

- text
- image
- visual document

等输入类型进入统一请求结构。

## 6. 推荐的输出结构

EmbeddingResponse 建议至少包含：

- `vectors`
- `dimensions`
- `mode`
- `usage`
- `modelInfo`
- `providerResponseMetadata`

如果是 multi-vector 模式，则 `vectors` 应允许表达多个局部表示。

## 7. 关键设计点

### 7.1 query/document 区分

很多现代 embedding 模型在 query 和 document 方向上有不同的编码最优路径。

平台抽象必须支持这一差异，而不是简单地所有内容都走同一模式。

### 7.2 维度裁剪

随着 Matryoshka 和 dimensions override 等能力增强，平台层建议直接支持 `dimensions` 参数。

这对：

- 成本控制
- 索引大小
- 召回速度
- 多层级索引

都有重要意义。

### 7.3 长输入支持

长文档 embedding 能力会直接影响 chunk 策略。

平台抽象应至少暴露：

- max input length
- truncation behavior
- recommended chunk policy hint

### 7.4 multimodal 与 multi-vector 支持

如果未来平台要支持图文混合检索、视觉文档检索或 late interaction 检索，抽象层必须预留能力位。

## 8. 能力声明建议

建议 EmbeddingModel 至少声明：

- supportsQueryDocumentMode
- supportsDimensionOverride
- supportsMultilingual
- supportsLongInput
- supportsMultimodalInput
- supportsMultiVector
- maxInputLength
- defaultDimensions

## 9. 统一接口示意

```ts
interface EmbeddingModel {
  capability: 'embedding';
  modelId: string;
  provider: string;
  features: EmbeddingModelFeatures;
  embed(request: EmbeddingRequest): Promise<EmbeddingResponse>;
}
```

## 10. 面向低代码平台的设计重点

在低代码平台中，EmbeddingModel 应特别适配以下对象：

- 组件说明
- 页面 Schema 摘要
- 模板描述
- 错误案例描述
- 数据源说明
- 代码与 DSL 片段

因此要重点关注：

- 中英文混合表现
- query/document 模式区分
- 长输入支持
- 维度与成本平衡

## 11. 常见误区

### 11.1 把 EmbeddingModel 设计成只接收单字符串

这会限制 batch、query/document distinction 和 multimodal 进化。

### 11.2 不暴露 dimensions 和 mode

会让平台失去后续成本优化和能力扩展空间。

### 11.3 把 embedding 接口和 vector search 接口混在一起

会导致检索层和模型层职责混乱。

## 12. 一句话总结

EmbeddingModel 设计的核心，是以“任务感知、模式可扩展、元数据完备”的方式统一表达向量生成能力，让平台既能支撑当前文本检索，也能平滑演进到多语言、多模态和 multi-vector 检索体系。

## 13. 参考资料

- OpenAI text-embedding-3-large docs: [https://platform.openai.com/docs/models/text-embedding-3-large](https://platform.openai.com/docs/models/text-embedding-3-large)
- OpenAI new embedding models and API updates: [https://openai.com/index/new-embedding-models-and-api-updates/](https://openai.com/index/new-embedding-models-and-api-updates/)
- Voyage Embeddings docs: [https://docs.voyageai.com/docs/embeddings](https://docs.voyageai.com/docs/embeddings)
- Cohere Embed docs: [https://docs.cohere.com/docs/cohere-embed](https://docs.cohere.com/docs/cohere-embed)
- BGE-M3 model card: [https://huggingface.co/BAAI/bge-m3](https://huggingface.co/BAAI/bge-m3)
- Jina Embeddings v4 model card: [https://huggingface.co/jinaai/jina-embeddings-v4](https://huggingface.co/jinaai/jina-embeddings-v4)
