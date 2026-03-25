# 搜索推荐型RAG

> 状态：第一版草案

## 1. 文档定位

本文档用于定义搜索推荐型 RAG 的业务形态与架构特点，重点回答：

- 什么时候 RAG 更像“搜索与推荐”而不是“问答”
- 搜索推荐型 RAG 在企业里常见于哪些场景
- 为什么这类场景对 recall、ranking、explanation 的要求通常高于长答案生成

## 2. 搜索推荐型 RAG 的定义

搜索推荐型 RAG 指的是：

系统主要目标不是直接生成长篇答案，而是根据用户意图，检索并排序一组最相关的对象、文档、组件、模板、案例或导航结果，再辅以简洁解释、推荐理由或使用建议。

它的核心目标通常是：

- 找到最相关对象
- 推荐最适合选项
- 给出排序和理由
- 引导用户去正确位置

用一句话概括：

搜索推荐型 RAG 是“检索增强 + 排序增强 + 轻量生成解释”的场景形态。

## 3. 为什么这是很多公司的高频场景

在企业里，很多高价值问题本质上不是“请你替我完整回答”，而是：

- 该用哪个组件
- 有没有现成模板
- 哪个 API 更适合
- 有没有相似案例
- 应该去哪篇文档或哪个对象看

这类问题更接近搜索和推荐，而不是完整问答。

因此，很多公司会把 RAG 用在：

- 企业搜索
- 组件推荐
- 模板推荐
- 代码 / API 搜索
- 案例导航
- 运维知识导航

## 4. 近两年的关键变化

### 4.1 企业搜索正在从 keyword search 升级为 semantic + hybrid + recommendation

近两年无论是 Azure AI Search、Weaviate、Cohere 还是各种 enterprise search 产品，都在明显强化：

- semantic relevance
- hybrid retrieval
- rerank
- explanation
- multi-source search

这说明现代企业搜索已经不只是搜索框，而是更接近“搜索 + 推荐 + grounding explanation”的统一体验。

### 4.2 推荐场景越来越受 RAG 影响

过去推荐系统更多依赖规则、协同过滤、排序模型；而现在在知识密集型、对象理解型场景里，RAG 提供了非常强的补充能力，尤其适合：

- 长尾对象推荐
- 解释型推荐
- 语义相似样板推荐
- 领域知识驱动推荐

### 4.3 解释性推荐越来越重要

企业用户通常不仅想知道“推荐结果是什么”，还想知道“为什么推荐这个”。

因此，搜索推荐型 RAG 相比传统搜索，更适合生成简短 explanation 和 source grounding。

## 5. 典型业务场景

### 5.1 组件推荐

例如：

- 做审批场景适合哪些表单组件
- 这个场景更适合用表格还是列表组件

### 5.2 页面模板推荐

例如：

- 采购审批页面有什么现成模板
- 有没有类似报表页面模板

### 5.3 API / SDK 推荐

例如：

- 查数据应该优先用哪个接口
- 某个业务场景用哪个 SDK 方法更合适

### 5.4 案例和知识导航

例如：

- 有没有类似错误案例
- 有没有类似实现样板
- 应该先看哪篇规范文档

## 6. 搜索推荐型 RAG 的典型架构特点

与 FAQ 型 RAG 相比，这类场景通常更强调：

- recall 覆盖
- ranking 质量
- 推荐理由
- 多候选展示

而不是长文本生成。

典型链路通常是：

query understanding -> hybrid recall -> rerank -> diversity control -> topN recommendation + explanation

## 7. 核心设计重点

### 7.1 候选覆盖比长回答更重要

如果候选池不够好，后续 explanation 再好也没价值。

### 7.2 排序质量是第一关键指标

这类场景通常比 FAQ 更依赖：

- rerank
- metadata boost
- sourceType weighting
- diversity control

### 7.3 explanation 应轻量但 grounded

用户通常需要的是：

- 推荐对象
- 推荐原因
- 关键差异
- 来源信息

而不是大段长文。

### 7.4 多样性比单一精度更重要

特别在推荐场景里，如果 topN 都是同类对象，用户体验往往很差。

## 8. 为什么搜索推荐型 RAG 很适合企业场景

这类场景天然符合企业知识系统特点：

- 对象很多
- 知识分散
- 命名不统一
- 用户往往不知道具体搜什么

RAG 的价值在这里主要体现为：

- 更懂用户意图
- 更懂领域对象
- 能解释推荐原因
- 能跨 source 统一搜索

## 9. 与传统搜索的区别

传统搜索更强调：

- keyword match
- 文档列表
- 用户自己点进去看

搜索推荐型 RAG 更强调：

- semantic + hybrid retrieval
- 面向对象而不是只面向文档
- 排序解释
- 检索结果二次整理

## 10. 与 FAQ 型 RAG 的区别

FAQ 型 RAG 更像：

- 直接回答一个问题

搜索推荐型 RAG 更像：

- 给出一组最相关候选，并解释为什么值得看

因此两者在 generation 层上的差异非常明显。

## 11. 与传统推荐系统的区别

传统推荐更依赖：

- 用户行为
- 统计模式
- 规则或排序模型

搜索推荐型 RAG 更依赖：

- 知识对象语义理解
- query understanding
- retrieval + rerank
- grounding explanation

两者可以结合，但不应混为一谈。

## 12. 面向低代码平台的典型形态

### 12.1 组件推荐

根据业务目标推荐组件。

### 12.2 页面模板推荐

根据场景目标推荐相似页面模板。

### 12.3 schema 片段推荐

根据当前页面上下文推荐局部 schema 结构或组件组合。

### 12.4 错误案例推荐

根据当前异常推荐相似历史案例和修复路径。

## 13. 常见优化方向

### 13.1 object-aware recall

优先检索对象而不是纯文本段落。

### 13.2 ranking + diversity 联合优化

既要相关，也要避免过度同质。

### 13.3 推荐解释模板化

例如：

- 推荐原因
- 适用场景
- 风险提醒
- 来源对象

### 13.4 结果卡片化展示

搜索推荐型 RAG 更适合对象卡片，而不是纯聊天气泡。

## 14. 主要评测重点

建议重点评测：

- topN 命中率
- 排序合理性
- 多样性
- explanation quality
- 用户点击与采纳率

## 15. 常见误区

### 15.1 把推荐场景硬做成长回答

### 15.2 只看语义相似，不做 rerank 和 object-level sorting

### 15.3 不控制多样性

### 15.4 不输出推荐理由

## 16. 一句话总结

搜索推荐型 RAG 的核心，是用 retrieval、ranking 和轻量 explanation 帮用户找到“最值得看、最适合选”的对象，而不是直接替用户写长答案；近两年的企业实践也越来越清楚，这类场景是 RAG 在产品、平台、开发者工具和企业搜索中最广泛的高价值形态之一。

## 17. 参考资料

- Azure AI Search overview: [https://learn.microsoft.com/en-us/azure/search/search-what-is-azure-search](https://learn.microsoft.com/en-us/azure/search/search-what-is-azure-search)
- Azure AI Search hybrid search overview: [https://learn.microsoft.com/en-us/azure/search/hybrid-search-overview](https://learn.microsoft.com/en-us/azure/search/hybrid-search-overview)
- Weaviate hybrid search docs: [https://docs.weaviate.io/weaviate/concepts/search/hybrid-search](https://docs.weaviate.io/weaviate/concepts/search/hybrid-search)
- Cohere Rerank overview: [https://docs.cohere.com/docs/rerank-overview](https://docs.cohere.com/docs/rerank-overview)
