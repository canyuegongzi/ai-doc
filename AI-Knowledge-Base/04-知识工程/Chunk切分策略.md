# Chunk切分策略

> 状态：第一版草案

## 1. 文档定位

本文档用于定义知识工程中的 chunk 切分策略，重点回答：

- 为什么 chunking 不是简单按固定字数切片
- 不同知识对象应该如何采用不同切分策略
- 近两年 RAG 和 agentic retrieval 的发展，对 chunking 提出了哪些新要求

## 2. 为什么 chunking 是 RAG 的关键分水岭

在很多系统里，chunking 被简化成：

- 每 500 字切一段
- overlap 50 字
- 完成

这种做法在小规模 demo 中可能勉强可用，但在企业系统中通常会迅速暴露问题：

- 语义边界被切断
- 标题和正文分离
- 代码块被截断
- 表格被打碎
- 不同类型知识对象被粗暴统一处理

结果就是：

召回的不是“知识单元”，而是“随机文本片段”。

因此，chunking 不是预处理小细节，而是知识对象建模的核心步骤。

## 3. 近两年的关键变化

### 3.1 chunking 正从固定窗口走向结构感知

当前更成熟的知识工程实践越来越强调：

- 按标题层级切
- 按段落语义切
- 按代码块切
- 按表格切
- 按字段定义切
- 按案例单元切

固定窗口仍然可用，但已不应是默认唯一策略。

### 3.2 agentic RAG 需要更“可行动”的 chunk

随着 agentic retrieval 和 tool-augmented RAG 增多，chunk 不只是为问答准备，还可能用于：

- 工具调用前查规范
- 生成前查模板
- 诊断前查错误案例
- workflow 节点做局部推理

这意味着 chunk 应更接近“可消费知识单元”，而不是“大段文本”。

### 3.3 长上下文模型并没有消灭 chunking

虽然长上下文能力增强，但在企业场景中仍然存在：

- 权限过滤
- 成本控制
- 更新粒度
- 引用定位
- 检索精度

因此，chunking 依然是基础工程能力，只是策略更精细了。

## 4. chunk 的核心目标

一个高质量 chunk 至少应满足：

- 语义相对完整
- 尺度适中
- 可被准确检索
- 可挂载 metadata
- 可保留来源定位
- 可被模型直接消费

## 5. chunking 的基本原则

### 5.1 语义完整优先于字符整齐

不要为了固定长度破坏语义边界。

### 5.2 结构边界优先于滑窗边界

能按标题、段落、代码块、表格切时，不要优先按字符数切。

### 5.3 领域对象优先于通用文本规则

如果内容本质是组件协议、字段定义、Schema 节点，就应按领域对象切，而不是按自然语言段落切。

### 5.4 引用能力必须保留

chunk 必须保留来源和定位信息，否则后续 citation、调试和审计都很弱。

## 6. 常见 chunking 策略

### 6.1 固定长度切分

特点：

- 简单
- 易实现
- 对纯文本基本可用

问题：

- 容易切断语义边界
- 不适合复杂结构内容

### 6.2 滑窗切分

特点：

- 通过 overlap 缓解边界断裂

问题：

- 会引入重复召回
- 不能真正解决结构感知问题

### 6.3 标题层级切分

适用于：

- 产品文档
- 规范文档
- 教程文档

优点：

- 更符合文档自然组织方式
- 更有利于 citation

### 6.4 段落语义切分

适用于：

- FAQ
- 说明文档
- 经验总结

### 6.5 代码块切分

适用于：

- 代码仓库
- 配置文件
- API 样例

优点：

- 保留完整函数、类、配置块

### 6.6 表格切分

适用于：

- 参数表
- 字段定义表
- 配置说明表

通常需要按表格整体、表头+行组或字段定义单元切分。

### 6.7 结构化对象切分

适用于：

- 页面 Schema
- 组件协议
- 数据源定义
- 逻辑节点定义

这种场景最适合按对象或子对象切，而不是按文本切。

### 6.8 案例单元切分

适用于：

- 工单
- 错误案例
- 复盘记录

通常以“一个问题 + 现象 + 原因 + 处理结果”为一个 chunk 单元。

## 7. overlap 应如何理解

overlap 只是缓解工具，不是 chunking 策略本身。

适合 overlap 的场景：

- 连续长文档
- 跨段落语义较强

不宜盲目 overlap 的场景：

- 结构化对象
- 代码块
- 表格
- 案例单元

否则只会造成重复召回和上下文污染。

## 8. 推荐的多策略 chunking 架构

成熟知识系统通常不应只有一种切分策略，而应采用“按 sourceType / objectType 选择切分器”的方式。

例如：

- 文档类 -> 标题层级 + 段落切分
- 代码类 -> 代码块切分
- 表格类 -> 表格单元切分
- Schema 类 -> 对象级切分
- 案例类 -> 案例单元切分

这也是近两年知识工程越来越清晰的一个方向：

chunking 应被视为一组策略，而不是一个参数。

## 9. chunk 大小如何决定

chunk size 没有全局最优，主要取决于：

- 模型上下文长度
- 检索粒度要求
- 文档结构
- 是否需要 citation
- 是否需要结构化生成约束

建议优先原则：

- 先保证语义完整
- 再控制 token 长度
- 最后再看统一性

## 10. chunk 应保留哪些附加信息

每个 chunk 建议至少保留：

- chunkId
- documentId
- sourceType
- title path
- section / page info
- objectType
- content summary（可选）
- metadata
- permissionScope

这样 chunk 才能真正作为可治理知识对象存在。

## 11. chunking 与 retrieval 的关系

chunk 设计会直接影响：

- recall precision
- recall coverage
- rerank quality
- context builder 的可控性
- citation 质量

如果 chunk 太粗：

- 容易混入噪声
- 上下文浪费 token

如果 chunk 太细：

- 语义不足
- 容易丢失完整信息

这也是为什么 chunking 必须和 retrieval 一起看，而不是单独拍脑袋决定。

## 12. chunking 与生成型场景的关系

对于生成增强场景，chunk 的作用不只是“让模型回答”，而是“让模型获得生成约束”。

例如低代码平台中的：

- 组件协议 chunk
- Schema 模板 chunk
- 字段定义 chunk
- 错误案例 chunk

这些 chunk 更像“结构约束片段”，而不是自然语言知识片段。

## 13. 面向低代码平台的 chunking 建议

### 13.1 组件文档

建议按：

- 组件简介
- props 定义
- setter 定义
- slots 定义
- 示例
- 常见问题

分别切分。

### 13.2 页面 Schema 样本

建议按：

- 页面级摘要
- 区块级 schema
- 组件节点级 schema
- 数据源绑定定义

切分。

### 13.3 错误案例

建议按“一条错误案例”作为完整单元切分。

### 13.4 逻辑编排节点说明

建议按“单节点定义”或“单模式模板”切分。

这类切分通常会明显优于固定 token 切分。

## 14. 常见误区

### 14.1 所有内容统一固定 500/1000 token 切分

这几乎必然损害知识质量。

### 14.2 chunking 不结合 metadata 和 objectType

这样会让 chunking 失去语义上下文。

### 14.3 只为问答设计 chunk，不为生成与诊断设计

现代知识工程的 chunk 必须服务更多任务。

### 14.4 overlap 过大

会导致重复召回、上下文膨胀和成本上升。

## 15. 一句话总结

chunk 切分策略的核心，不是按固定长度把文本切开，而是按语义单元、结构边界和领域对象把知识组织成可检索、可引用、可作为生成约束的知识块；近两年的最佳实践也越来越明确，真正有效的 chunking 一定是多策略、结构感知、面向任务的。

## 16. 参考资料

- OpenAI Retrieval guide: [https://platform.openai.com/docs/guides/retrieval](https://platform.openai.com/docs/guides/retrieval)
- OpenAI File Search guide: [https://platform.openai.com/docs/guides/tools-file-search/](https://platform.openai.com/docs/guides/tools-file-search/)
- LlamaIndex node parsers: [https://docs.llamaindex.ai/en/stable/module_guides/loading/node_parsers/modules/](https://docs.llamaindex.ai/en/stable/module_guides/loading/node_parsers/modules/)
- LlamaIndex ingestion pipeline: [https://docs.llamaindex.ai/en/stable/module_guides/loading/ingestion_pipeline/](https://docs.llamaindex.ai/en/stable/module_guides/loading/ingestion_pipeline/)
- Unstructured chunking: [https://docs.unstructured.io/open-source/core-functionality/chunking](https://docs.unstructured.io/open-source/core-functionality/chunking)
- Cohere Agentic RAG tutorial: [https://docs.cohere.com/v2/docs/agentic-rag](https://docs.cohere.com/v2/docs/agentic-rag)
