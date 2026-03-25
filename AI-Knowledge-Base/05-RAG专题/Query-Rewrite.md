# Query-Rewrite

> 状态：第一版草案

## 1. 文档定位

本文档用于定义 RAG 在线链路中的 Query Rewrite 设计，重点回答：

- 为什么用户原始问题通常不适合直接检索
- Query Rewrite 与 Query Expansion、Query Planning 的边界是什么
- 近两年多查询、会话感知、agentic retrieval 兴起后，rewrite 能力发生了哪些变化

## 2. 为什么 Query Rewrite 现在越来越重要

很多 RAG 系统效果差，不是因为索引不好，而是因为输入给检索系统的 query 不够好。

企业真实请求通常存在这些问题：

- 表达模糊
- 默认省略上下文
- 使用内部简称或别名
- 一句话里混多个问题
- 用业务场景话语表达，但知识库按技术术语组织

如果直接拿原句去检索，结果常常是：

- 召回过宽
- 召回过窄
- 召回方向错误
- 多问题混在一起导致相关性下降

因此，query rewrite 已经从“锦上添花”变成现代 RAG 的标准环节。

## 3. 近两年的关键变化

### 3.1 Query Rewrite 正从简单改写走向 query understanding

早期 rewrite 更多是同义改写；现在更成熟的系统会同时做：

- 指代消解
- 会话上下文补全
- 实体显式化
- 术语扩展
- 问题拆分
- 检索意图识别

### 3.2 多查询和 agentic retrieval 放大了 rewrite 的价值

Azure AI Search 的 agentic retrieval 已经明确把复杂问题拆解成多个 focused subqueries。Cohere 的 agentic RAG 实践也强调 sequential reasoning over multiple sources。

这说明 rewrite 已不只是“修一句话”，而开始成为更大 query planning 流程的一部分。

### 3.3 会话历史成为 rewrite 的正式输入

现代企业助手中，用户很多问题依赖前文，如果不做会话感知 rewrite，检索往往会明显失真。

## 4. Query Rewrite 解决什么问题

### 4.1 补全上下文

例如把“这个为什么渲染不出来”补成：

- 当前组件是什么
- 当前页面对象是什么
- 当前问题具体指 slot 还是 props

### 4.2 统一术语表达

例如把业务说法改成知识系统更容易命中的技术说法。

### 4.3 提升召回覆盖率

通过更明确、更完整的 query 让 recall 更稳。

### 4.4 为多查询和 query planning 提供基础

rewrite 是后续 expansion、decomposition 和 subquery generation 的前置基础。

## 5. Query Rewrite 与相近概念的区别

### 5.1 与 Query Expansion 的区别

- rewrite 更强调“重写为更好的主查询”
- expansion 更强调“补充更多相关查询表达”

### 5.2 与 Query Planning 的区别

- rewrite 更偏单 query 层面优化
- planning 更偏多查询、多步骤检索路径设计

### 5.3 与意图识别的区别

- 意图识别回答“这是什么类型问题”
- rewrite 回答“这个问题更适合怎么去查”

## 6. 推荐的 rewrite 目标

一个好的 rewrite 结果通常应满足：

- 更明确
- 更少歧义
- 更贴近知识库语义
- 不改变原问题真实意图
- 便于后续 recall 和 filter 使用

## 7. 常见 rewrite 类型

### 7.1 指代消解

例如：

- “这个” -> 组件名 / 页面名 / schema 节点名
- “它” -> 上一轮明确对象

### 7.2 上下文补全

结合会话历史补上省略信息。

### 7.3 术语标准化

把用户用语映射为标准术语。

### 7.4 问题显式化

把隐含问题改写为完整问题。

### 7.5 多问题拆分前重写

为后续 subquery 生成做准备。

## 8. 推荐的 rewrite 输入

rewrite 阶段建议至少能拿到：

- 用户原始 query
- 会话历史
- 当前场景类型
- 已知实体上下文
- 领域术语表
- 当前 app / tenant / module context

## 9. 推荐的 rewrite 输出

建议至少输出：

- rewrittenQuery
- rewriteReason（可选）
- extractedEntities
- confidence
- optionalSubqueries（若进入复杂模式）

这有助于后续 tracing 和调试。

## 10. rewrite 的实现路径

### 10.1 规则驱动 rewrite

适用于：

- 术语替换
- 指代模板
- 固定场景补全

优点：

- 稳定
- 可控

### 10.2 模型驱动 rewrite

适用于：

- 模糊表达
- 多轮会话
- 复杂意图补全

优点：

- 覆盖复杂情况

### 10.3 混合式 rewrite

企业场景更推荐：

- 先做规则化补全与术语映射
- 再用模型做高层语义重写

## 11. rewrite 与 retrieval 的关系

rewrite 不应独立设计，而应围绕目标 retrieval 方式设计。

例如：

- dense retrieval 更关注语义清晰度
- sparse retrieval 更关注关键词和术语
- hybrid retrieval 更希望 query 同时兼顾语义和 lexical signal

因此 rewrite 结果不一定总是“更自然语言化”，很多时候应更“检索友好”。

## 12. agentic retrieval 场景中的 rewrite

在复杂 retrieval 场景中，rewrite 可能升级为：

- subquery generation
- question decomposition
- source-aware reformulation
- parallel retrieval prompts

这时 rewrite 已经开始与 query planning 融合。

## 13. 面向低代码平台的 rewrite 场景

### 13.1 组件问题问法标准化

例如：

- “这个控件显示不出来” -> 组件渲染异常 / slot 配置问题 / props 配置问题

### 13.2 Schema 生成需求补全

例如：

- “帮我做个审批页面” -> 审批类表单页面 / 流程状态 / 数据字段 / 常见模板

### 13.3 错误诊断问题补全

例如：

- “这个字段不对” -> 字段绑定不存在 / 类型不匹配 / schema path 异常

## 14. 常见误区

### 14.1 直接用原句检索，不做 rewrite

### 14.2 rewrite 过度改写，改变原意

### 14.3 rewrite 只做语言润色，不考虑检索目标

### 14.4 rewrite 结果不可追踪，难以调试

## 15. 一句话总结

Query Rewrite 的核心，不是把用户问题说得更漂亮，而是把原始问题转化为更适合检索系统理解、召回和排序的查询表达；近两年的实践也越来越表明，rewrite 已经从简单改写演进为现代 RAG query understanding 的关键环节。

## 16. 参考资料

- Azure AI Search RAG overview: [https://learn.microsoft.com/en-us/azure/search/retrieval-augmented-generation-overview](https://learn.microsoft.com/en-us/azure/search/retrieval-augmented-generation-overview)
- Azure AI Search agentic retrieval: [https://learn.microsoft.com/en-us/azure/search/search-agentic-retrieval-concept](https://learn.microsoft.com/en-us/azure/search/search-agentic-retrieval-concept)
- Cohere Agentic RAG tutorial: [https://docs.cohere.com/v2/docs/agentic-rag](https://docs.cohere.com/v2/docs/agentic-rag)
- OpenAI Retrieval guide: [https://platform.openai.com/docs/guides/retrieval](https://platform.openai.com/docs/guides/retrieval)
