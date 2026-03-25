# Query-Expansion

> 状态：第一版草案

## 1. 文档定位

本文档用于定义 RAG 在线链路中的 Query Expansion 设计，重点回答：

- 为什么 query expansion 在现代 RAG 中越来越重要
- Query Expansion 与 Query Rewrite、Query Planning 的边界是什么
- 近两年 multi-query retrieval 和 agentic retrieval 发展后，query expansion 的角色发生了哪些变化

## 2. 什么是 Query Expansion

Query Expansion 指的是：

在不改变用户核心意图的前提下，为原始查询补充更多相关表达、同义说法、上下位词、领域术语、结构化关键词或子查询，从而提升 recall 覆盖率和相关结果命中率。

用一句话概括：

Query Rewrite 更像“把问题改写得更适合检索”，Query Expansion 更像“把问题相关的检索表达扩展开”。

## 3. 为什么 Query Expansion 越来越重要

在企业知识系统中，用户问题与知识表达之间往往存在显著距离。

常见情况包括：

- 用户说的是业务语言，知识库存的是技术语言
- 用户说的是泛化目标，知识对象用的是具体对象名
- 用户只说一个表象，但真实知识散落在多个术语下
- 同一个概念在组织内部存在多种别名

如果检索系统只拿原 query 或简单 rewrite 结果去搜索，常常会漏掉：

- 相关术语变体
- 同义表达
- 别名和旧名
- 近义问题模板

因此，query expansion 是提高 recall coverage 的关键手段之一。

## 4. 近两年的关键变化

### 4.1 Query Expansion 正从“加几个同义词”升级为 multi-query retrieval 基础能力

Azure AI Search 2026 年 3 月的 agentic retrieval 文档明确提到：系统会利用同义词映射和 LLM 生成的 paraphrasing，把原始 query 重写成多个子查询，并并行执行。

这说明在最新架构里，query expansion 不再只是检索前的小技巧，而是 multi-query / agentic retrieval 的正式组成部分。

### 4.2 expansion 与 query planning 正在逐渐融合

当系统开始：

- 生成多个查询变体
- 拆出多个 focused subqueries
- 对不同 source 走不同检索路径

就很难再把 expansion 只看成“加词”。它已经开始和 query planning 接近。

### 4.3 expansion 越来越依赖领域词表、metadata 和知识对象结构

在企业场景中，最有效的 expansion 往往不是通用同义词，而是：

- 组件别名
- 模块缩写
- 历史版本名称
- 字段名和技术术语
- 错误码类别
- 领域对象之间的近义映射

这意味着 expansion 开始更强依赖企业知识图谱、metadata 和术语表。

## 5. Query Expansion 解决什么问题

### 5.1 解决召回覆盖不足问题

通过补充相关表达，降低漏召回风险。

### 5.2 解决术语不一致问题

把用户语言与知识系统语言桥接起来。

### 5.3 解决复杂问题单 query 不足的问题

对复合问题，可通过多个扩展查询覆盖不同方面。

### 5.4 解决跨 source 检索表达差异问题
n
同一问题在 FAQ、API 文档、错误案例、页面模板中的表达方式通常不同，扩展查询有助于跨源覆盖。

## 6. Query Expansion 与相近概念的区别

### 6.1 与 Query Rewrite 的区别

- rewrite 主要是生成“更好的主查询”
- expansion 主要是补充“一组相关查询表达”

### 6.2 与 Query Decomposition 的区别

- expansion 不一定拆任务
- decomposition 更强调把复合问题拆成多个子问题

### 6.3 与 Query Planning 的区别

- expansion 偏查询表达扩展
- planning 偏检索路径和步骤设计

在 agentic retrieval 场景里，这几者开始逐步融合，但在工程上仍建议保留概念区分。

## 7. 常见 Query Expansion 方式

### 7.1 同义词扩展

例如：

- slot -> 插槽
- props -> 属性配置
- pagination -> 分页

### 7.2 别名扩展

例如：

- 组件旧名 / 新名
- 内部简称 / 正式名
- API 旧路径 / 新路径

### 7.3 上下位词扩展

例如：

- “审批页面”扩展到“表单页面”“流程页面”“审批模板”

### 7.4 结构化关键词扩展

例如补出：

- 错误码
- 字段名
- 组件名
- 模块名

### 7.5 多查询扩展

为一个 query 生成多个表达变体并并行检索。

## 8. 推荐的 Expansion 输入

建议至少使用：

- 原始 query
- rewrite 后 query
- 会话历史
- 术语表 / synonym map
- 领域别名词典
- 场景类型
- 当前 sourceType / objectType hints

## 9. 推荐的 Expansion 输出

建议至少输出：

- primaryQuery
- expandedQueries
- expansionType
- extractedTerms
- confidence
- optionalSourceHints

这样后续可以：

- 做并行 recall
- 做 tracing
- 做 query activity 解释

## 10. Expansion 的实现策略

### 10.1 规则驱动 expansion

适用于：

- 术语词典
- 组件别名
- 错误码映射
- 固定 synonym map

优点：

- 稳定
- 可控

### 10.2 模型驱动 expansion

适用于：

- 模糊问题
- 多语言变体
- 复杂场景扩展

优点：

- 覆盖更灵活

### 10.3 混合策略

企业场景最推荐：

- 先用规则扩展关键术语
- 再用模型做 paraphrase 和多 query 扩展

## 11. expansion 与 recall 的关系

Query Expansion 的价值最终体现在 recall 上。

典型链路通常是：

- rewrite 主查询
- expansion 生成 query variants / subqueries
- recall 并行召回
- merge / fusion
- rerank

因此，expansion 不应脱离 recall 单独优化。

## 12. expansion 的风险与控制

如果 expansion 做得不好，也会带来明显副作用：

- query drift：扩展过度偏离原意
- 噪声扩大：引入太多低价值候选
- 成本上升：多 query 并行增加检索成本
- latency 增加：子查询过多导致响应变慢

因此，expansion 需要受控，例如：

- 限制扩展数量
- 按场景启用
- 高价值问题才走多查询
- 对子查询质量做评测

## 13. 面向低代码平台的 expansion 场景

### 13.1 组件问题扩展

例如：

- “按钮点击没反应”可扩展为事件绑定、onClick、交互逻辑、数据流依赖

### 13.2 Schema 生成需求扩展

例如：

- “审批页面”可扩展为表单、流程状态、审批意见、节点操作、字段权限

### 13.3 诊断问题扩展

例如：

- “渲染不出来”可扩展到 slot、props、schema path、data binding、runtime error

## 14. 常见误区

### 14.1 认为 expansion 就是自动加同义词

现代扩展远不止于此，很多时候是 multi-query retrieval 的前置能力。

### 14.2 expansion 数量越多越好

过多扩展会带来噪声和成本问题。

### 14.3 不区分高价值场景和普通场景，一律走复杂 expansion

这会显著增加延迟和运营成本。

### 14.4 expansion 结果不可追踪

后续很难知道是哪个子查询带来了错误召回或效果提升。

## 15. 一句话总结

Query Expansion 的核心，不是机械补几个近义词，而是围绕用户问题生成一组受控、相关、可追踪的检索表达，以提升 recall coverage；近两年的最新架构也越来越明确，expansion 已经成为 multi-query retrieval 与 agentic retrieval 的关键能力，而不再只是传统搜索的小优化。

## 16. 参考资料

- Azure AI Search semantic query rewrite: [https://learn.microsoft.com/en-us/azure/search/semantic-how-to-query-rewrite](https://learn.microsoft.com/en-us/azure/search/semantic-how-to-query-rewrite)
- Azure AI Search agentic retrieval concept: [https://learn.microsoft.com/en-us/azure/search/search-agentic-retrieval-concept](https://learn.microsoft.com/en-us/azure/search/search-agentic-retrieval-concept)
- Azure AI Search RAG overview: [https://learn.microsoft.com/en-us/azure/search/retrieval-augmented-generation-overview](https://learn.microsoft.com/en-us/azure/search/retrieval-augmented-generation-overview)
- Cohere Agentic RAG tutorial: [https://docs.cohere.com/v2/docs/agentic-rag](https://docs.cohere.com/v2/docs/agentic-rag)
- OpenAI Retrieval guide: [https://platform.openai.com/docs/guides/retrieval](https://platform.openai.com/docs/guides/retrieval)
