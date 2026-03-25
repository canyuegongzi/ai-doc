# RAG评测体系

> 状态：第一版草案

## 1. 文档定位

本文档用于定义 RAG 系统的评测体系，重点回答：

- 为什么 RAG 必须做分层评测，而不是只看最终回答好不好
- retrieval、generation、citation、groundedness 应如何分别评估
- 近两年 agentic retrieval 和复杂 RAG 架构兴起后，评测体系应如何升级

## 2. 为什么 RAG 评测比普通 LLM 评测更复杂

普通生成任务常常只需要判断输出质量，但 RAG 系统本质上是多阶段链路：

- query understanding
- recall
- rerank
- context building
- generation
- citation

因此，同样一个错误回答，可能根因完全不同：

- query rewrite 错了
- recall 没召回对的内容
- rerank 排错了
- context builder 把关键块丢了
- generation 幻觉了
- citation 对不上

如果没有分层评测，优化就会非常盲目。

## 3. 近两年的关键变化

### 3.1 RAG 评测正在从“看最终答案”走向“看整个 retrieval pipeline”

Azure AI Search、Anthropic 和越来越多 enterprise RAG 实践都在强调：

生产级系统必须能分析 query plan、retrieval response、source grounding、answer quality，而不是只看最终回答表面质量。

### 3.2 agentic retrieval 让评测对象变得更多

当 retrieval 本身变成：

- multi-query
- query plan
- parallel search
- structured retrieval response

评测就不仅要看 recall，还要看：

- 子查询是否合理
- 并行召回是否覆盖充分
- query activity 是否有效

### 3.3 groundedness / citation correctness 成为正式指标

近两年企业 RAG 越来越强调：

- answer 是否由证据支撑
- 引用是否正确
- 是否存在 evidence-free hallucination

这使 citation 与 groundedness 评测变成正式环节。

## 4. RAG 评测的总体原则

### 4.1 分层评测优先于整体打分

先分解链路，再汇总整体质量。

### 4.2 样本评测与线上反馈结合

离线样本可以做稳定回归，线上反馈可以发现真实世界问题。

### 4.3 质量与成本一起看

高质量不代表最优，如果成本不可接受，系统仍不可持续。

### 4.4 retrieval 评测和 generation 评测必须联动

二者分开看不够，联合看才能定位根因。

## 5. 推荐的评测分层

建议至少从六层做评测。

### 5.1 Query Understanding 评测

关注：

- rewrite 是否保留原意
- 是否补全关键信息
- 子查询拆分是否合理

### 5.2 Recall 评测

关注：

- 正确知识是否进入候选池
- recall topK 是否覆盖关键块
- multi-source coverage 是否充分

### 5.3 Rerank 评测

关注：

- 正确候选是否排到更前
- 噪声是否被压下去
- 不同 source 的排序是否合理

### 5.4 Context Builder 评测

关注：

- 关键 evidence 是否被保留
- 上下文是否过于冗余
- 是否去重成功
- citation lineage 是否保留

### 5.5 Answer Generation 评测

关注：

- correctness
- completeness
- groundedness
- structured validity
- uncertainty handling

### 5.6 Citation 评测

关注：

- citation 是否正确映射来源
- 引用是否真实支撑答案
- page / section / object 指向是否准确

## 6. 常见评测指标

### 6.1 recall / coverage 指标

例如：

- Recall@K
- Hit@K
- source coverage
- evidence coverage

### 6.2 ranking 指标

例如：

- MRR
- NDCG
- top-ranked relevance

### 6.3 answer 指标

例如：

- answer correctness
- answer groundedness
- answer completeness
- hallucination rate

### 6.4 citation 指标

例如：

- citation precision
- citation correctness
- unsupported claim rate

### 6.5 系统指标

例如：

- latency
- token cost
- retrieval cost
- subquery count
- failure rate

## 7. 离线评测集如何设计

建议样本至少覆盖：

- FAQ 问答
- 复杂模糊问题
- 多轮上下文问题
- 多源知识问题
- 精确术语问题
- 错误码 / 字段名问题
- 生成增强问题
- 诊断型问题

每个样本最好包括：

- 原始 query
- 场景类型
- 期望 evidence
- 期望答案要点
- 可接受引用来源

## 8. 线上评测与反馈

离线评测不能覆盖所有真实问题，因此还应有线上反馈机制，例如：

- 用户是否点击来源
- 用户是否认为答案有帮助
- 用户是否投诉“答非所问”
- 哪类 sourceType 常被反馈不可信

## 9. 面向 agentic retrieval 的新增评测点

如果系统进入 agentic retrieval 模式，建议额外评测：

- subquery quality
- query plan reasonableness
- parallel retrieval effectiveness
- merged retrieval response quality
- query activity explainability

## 10. 面向低代码平台的评测重点

在低代码平台场景中，建议重点评测：

### 10.1 组件问答

- 是否命中正确组件
- 是否引用正确组件协议条目

### 10.2 Schema 生成增强

- 检索到的模板和规则是否正确
- 生成结果是否结构合法
- 是否引用了合适模板来源

### 10.3 配置诊断

- 是否命中真实问题类型
- 是否召回相关错误案例
- 是否给出可执行修复建议

### 10.4 运维辅助

- 是否召回正确规范与日志证据
- 是否减少错误结论和过度推断

## 11. 评测体系中的常见误区

### 11.1 只看最终回答是否“像对的”

### 11.2 不看 evidence quality

### 11.3 不区分 retrieval 退化和 generation 退化

### 11.4 没有线上反馈闭环

### 11.5 成本完全不进入评测视野

## 12. 一句话总结

RAG 评测体系的核心，是把 query understanding、recall、rerank、context building、answer generation 和 citation 分层评估，并与成本、延迟和线上反馈联合分析；近两年的发展也越来越明确，只有具备 pipeline-level evals 的 RAG，才能真正持续优化并稳定进入企业生产环境。

## 13. 参考资料

- Azure AI Search RAG overview: [https://learn.microsoft.com/en-us/azure/search/retrieval-augmented-generation-overview](https://learn.microsoft.com/en-us/azure/search/retrieval-augmented-generation-overview)
- Azure AI Search agentic retrieval concept: [https://learn.microsoft.com/en-us/azure/search/search-agentic-retrieval-concept](https://learn.microsoft.com/en-us/azure/search/search-agentic-retrieval-concept)
- Cohere Agentic RAG tutorial: [https://docs.cohere.com/v2/docs/agentic-rag](https://docs.cohere.com/v2/docs/agentic-rag)
- OpenAI Retrieval guide: [https://platform.openai.com/docs/guides/retrieval](https://platform.openai.com/docs/guides/retrieval)
- DeepEval RAG evaluation guide: [https://www.deepeval.com/guides/guides-rag-evaluation](https://www.deepeval.com/guides/guides-rag-evaluation)
- TruLens RAG triad docs: [https://www.trulens.org/getting_started/core_concepts/rag_triad/](https://www.trulens.org/getting_started/core_concepts/rag_triad/)
