# Context-Builder

> 状态：第一版草案

## 1. 文档定位

本文档用于定义 RAG 在线链路中的 Context Builder 设计，重点回答：

- 为什么检索结果不能直接原样塞给模型
- context builder 应负责哪些压缩、去重、组合和引用工作
- 近两年长上下文模型出现后，context builder 为什么反而更重要了

## 2. 为什么 Context Builder 是现代 RAG 的关键中枢

在很多早期 RAG 系统中，流程通常是：

- 检索 topK
- 拼接文本
- 发给模型

这种方式在小规模 demo 中有时能工作，但在生产级系统中问题非常明显：

- 候选里有大量重复内容
- 多个 chunk 高度同质
- 长文和短文混排导致上下文利用率低
- 无关内容挤占 token
- citation 信息丢失
- 模型接收到的上下文结构混乱

因此，Context Builder 的职责就是把“检索结果”变成“模型可高质量消费的上下文”。

## 3. 近两年的关键变化

### 3.1 长上下文模型没有弱化 Context Builder，反而抬高了它的重要性

一个常见误解是：

既然模型上下文窗口更长了，就可以少做上下文治理。

但现实恰恰相反。

长上下文带来的不是“可以乱塞”，而是：

- 可注入内容更多
- 噪声污染风险更大
- 成本更高
- 选择与组织更关键

因此，Context Builder 现在更像“上下文编排引擎”，而不是简单拼接器。

### 3.2 context compression 正在成为标准能力

LlamaIndex 等近两年的检索与上下文优化实践越来越强调：

- content selection
- compression
- synthesis-friendly formatting
- source-aware grouping

这说明 Context Builder 已经从检索后置步骤升级为正式架构层。

### 3.3 agentic retrieval 使上下文构造更动态

在 agentic retrieval 场景中，上下文不一定一次构完，而可能：

- 分阶段构造
- 按子任务构造
- 按工具步骤构造
- 随运行状态动态更新

这对 Context Builder 提出了更高要求。

## 4. Context Builder 解决什么问题

### 4.1 去噪问题

从候选池中去掉明显无关或低价值内容。

### 4.2 去重问题

避免多个 chunk 传达同样信息，浪费上下文预算。

### 4.3 结构组织问题

把候选内容按可理解方式组织给模型，例如：

- 按主题
- 按来源
- 按优先级
- 按推理步骤

### 4.4 引用保留问题

确保模型使用内容时仍能回溯来源。

## 5. Context Builder 的核心职责

### 5.1 Candidate Selection

决定哪些候选真正进入上下文。

### 5.2 Deduplication

对文本相近、来源相同、语义重复的内容做去重。

### 5.3 Compression

对过长内容做压缩、摘要或裁剪。

### 5.4 Grouping

按来源、主题、对象类型或任务阶段对内容分组。

### 5.5 Formatting

把内容组织成适合模型理解和引用的结构。

### 5.6 Citation Packaging

保留：

- source id
- title
- section
- page
- object reference

## 6. 为什么 topK 结果不能直接拼接

### 6.1 topK 排序不等于上下文最优组合

最相关的几个候选可能高度重复，拼在一起并不一定最好。

### 6.2 模型并不擅长自动去噪

如果上下文混乱，模型常常会：

- 忽略关键信息
- 过度依赖错误片段
- 混淆多个来源

### 6.3 上下文预算是稀缺资源

特别是在长输出、tool use 或多轮推理场景中，context token 预算非常珍贵。

## 7. 常见 Context Builder 策略

### 7.1 topN 直接截断

最简单，但通常不够稳。

### 7.2 source-aware merge

把同一来源或同一标题路径的 chunk 合并。

### 7.3 diversity-aware selection

避免多个相似 chunk 重复进入上下文。

### 7.4 summary-based compression

对于过长候选，先压缩成摘要片段，再放入上下文。

### 7.5 task-aware packaging

根据任务类型组织上下文，例如：

- FAQ -> 问题相关说明片段
- 生成 -> 模板 + 规则 + 示例
- 诊断 -> 规则 + 案例 + 当前状态

## 8. 推荐的上下文组织方式

建议尽量不要把所有内容简单串成大段文本，而是组织为更可读结构，例如：

- 问题相关知识
- 关键规则
- 示例
- 注意事项
- 来源列表

或者：

- Source A:
- Source B:
- Source C:

这样有利于模型识别结构和引用来源。

## 9. Context Builder 与 citation 的关系

Citation 不应在最后临时补，而应在 Context Builder 阶段就保留。

推荐至少保留：

- source id
- source title
- source path / uri
- page / section / object path

这样后续生成回答时，模型或应用层才有足够信息生成可解释引用。

## 10. Context Builder 与不同场景的关系

### 10.1 FAQ 场景

重点：

- 保留最直接解释问题的片段
- 避免长背景材料压制核心答案

### 10.2 生成增强场景

重点：

- 保留模板
- 保留规则
- 保留少量高质量示例
- 避免混入互相冲突的样例

### 10.3 配置诊断场景

重点：

- 保留相关规则
- 保留相似错误案例
- 保留当前状态相关信息

### 10.4 agentic retrieval 场景

重点：

- 为当前步骤构造局部上下文
- 不一定一次把所有材料都塞进去
- 上下文可随任务推进动态变化

## 11. 面向低代码平台的 Context Builder 建议

在低代码平台场景中，建议按任务类型做不同上下文包。

### 11.1 组件问答

可组织为：

- 组件说明
- props / setter / slot 定义
- 常见问题
- 来源信息

### 11.2 Schema 生成

可组织为：

- 页面模板摘要
- 相关组件定义
- 数据源约束
- 生成规则

### 11.3 配置纠错

可组织为：

- 相关错误规则
- 相似案例
- 当前组件 / schema 上下文
- 可疑字段信息

## 12. 常见误区

### 12.1 检索到什么就全塞给模型

### 12.2 不做去重和来源合并

### 12.3 citation 信息在上下文构建阶段丢失

### 12.4 不按任务类型调整上下文组织方式

## 13. 一句话总结

Context Builder 的核心，是把 recall 和 rerank 产生的候选池压缩、去重、分组、格式化并保留引用信息，最终构造成真正适合模型消费的高质量上下文；近两年的最佳实践也越来越清楚，长上下文并没有让这一步变得不重要，反而让“上下文治理”成为 RAG 成败的关键中枢。 

## 14. 参考资料

- LlamaIndex retrieval + node postprocessors docs: [https://docs.llamaindex.ai/en/stable/module_guides/querying/node_postprocessors/](https://docs.llamaindex.ai/en/stable/module_guides/querying/node_postprocessors/)
- LlamaIndex response synthesis docs: [https://docs.llamaindex.ai/en/stable/module_guides/querying/response_synthesizers/](https://docs.llamaindex.ai/en/stable/module_guides/querying/response_synthesizers/)
- OpenAI Retrieval guide: [https://platform.openai.com/docs/guides/retrieval](https://platform.openai.com/docs/guides/retrieval)
- Cohere Agentic RAG tutorial: [https://docs.cohere.com/v2/docs/agentic-rag](https://docs.cohere.com/v2/docs/agentic-rag)
- Azure AI Search RAG overview: [https://learn.microsoft.com/en-us/azure/search/retrieval-augmented-generation-overview](https://learn.microsoft.com/en-us/azure/search/retrieval-augmented-generation-overview)
