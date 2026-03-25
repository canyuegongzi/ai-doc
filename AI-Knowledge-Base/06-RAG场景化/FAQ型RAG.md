# FAQ型RAG

> 状态：第一版草案

## 1. 文档定位

本文档用于定义 FAQ 型 RAG 的业务形态与架构特点，重点回答：

- 什么样的场景适合归为 FAQ 型 RAG
- FAQ 型 RAG 在企业里为什么仍然是最常见、最容易落地的形态
- 近两年 FAQ 型 RAG 在架构上有哪些升级，而不再只是“文档问答”

## 2. FAQ 型 RAG 的定义

FAQ 型 RAG 指的是：

围绕相对稳定的知识内容，为用户提供基于企业私有资料、规范文档、产品文档、帮助中心、常见问题和操作说明的问答型知识服务。

它的核心目标通常不是执行任务，而是：

- 准确回答
- 引用依据
- 缩短查文档成本
- 提高平台使用效率

用一句话概括：

FAQ 型 RAG 是把企业知识库升级为可问、可答、可引用的知识服务层。

## 3. 为什么 FAQ 型 RAG 仍然是最广泛的企业场景

在企业里，大量高频问题天然具有 FAQ 属性，例如：

- 这个功能怎么用
- 这个组件做什么
- 某个接口参数是什么意思
- 平台某个报错通常是什么原因
- 某个流程规范是什么

这些问题的共同特点是：

- 需求高频
- 问题边界相对清晰
- 知识来源相对稳定
- 用户本来就会去查文档

因此 FAQ 型 RAG 通常是企业 AI 能力最先落地、ROI 最清晰的一类应用。

## 4. 近两年的关键变化

### 4.1 FAQ 型 RAG 已从“文档聊天机器人”升级为受治理知识服务

Azure AI Search 近年的 RAG 文档已经把 query understanding、security trimming、citations、multi-source access 明确作为正式问题域。这说明 FAQ 型 RAG 不再只是“上传文档，接个聊天框”，而是需要：

- query understanding
- hybrid retrieval
- semantic ranking / rerank
- citation
- 权限过滤
- 可观测性

### 4.2 FAQ 型 RAG 正从单文档源走向多源知识统一查询

当前企业 FAQ 已不只来自帮助文档，还来自：

- Wiki
- 产品文档
- 组件协议
- API 文档
- FAQ 表
- 工单沉淀
- 平台规范

这意味着 FAQ 型 RAG 也在逐步从“单知识库问答”演进到“多源知识统一问答”。

### 4.3 FAQ 型 RAG 也开始受 agentic retrieval 影响

虽然 FAQ 场景通常不需要完整 agentic retrieval，但复杂 FAQ 问题也越来越受益于：

- query rewrite
- query expansion
- multi-query retrieval
- semantic answer packaging

因此，FAQ 型 RAG 虽然是经典形态，但底层链路已经明显升级。

## 5. 典型业务场景

### 5.1 产品 / 平台使用问答

例如：

- 某个功能入口在哪
- 某个配置项如何使用
- 某个模块的作用是什么

### 5.2 组件文档问答

例如：

- 某组件适合什么场景
- props / slot / setter 如何配置
- 某组件常见问题是什么

### 5.3 API / 数据模型说明问答

例如：

- 某字段含义是什么
- API 参数如何传
- 哪个接口适用于什么场景

### 5.4 规则与规范问答

例如：

- 发布规范是什么
- 命名规范是什么
- 某审批规则怎么定义

## 6. FAQ 型 RAG 的典型架构

FAQ 型 RAG 通常采用以下经典链路：

1. query understanding
2. hybrid recall
3. rerank
4. context builder
5. grounded answer generation
6. citation return

与复杂 agentic RAG 相比，它通常具有以下特点：

- 检索路径相对稳定
- 很少涉及工具调用
- 更强调回答准确与来源可信
- 更适合单轮或轻量多轮场景

## 7. FAQ 型 RAG 的核心设计重点

### 7.1 query understanding 要足够稳

FAQ 问题经常很短、很口语化、上下文省略多，因此 rewrite 和 expansion 很关键。

### 7.2 retrieval 需要兼顾术语与语义

FAQ 场景常同时包含：

- 专业术语
- 用户口语化表达
- 模块简称
- 组件别名

因此 hybrid search 通常比 single dense 更稳。

### 7.3 answer generation 要尽量 grounded

FAQ 场景最怕“听起来像对，但其实不对”。

### 7.4 citation 非常重要

因为 FAQ 用户经常希望继续点击原文深入阅读。

## 8. FAQ 型 RAG 的适用条件

通常适用于：

- 知识相对稳定
- 来源可信度较高
- 问题目标清晰
- 不要求系统直接执行动作
- 用户更需要解释和引导

## 9. FAQ 型 RAG 不适合什么

不太适合：

- 多步骤复杂诊断
- 高副作用执行任务
- 高度依赖实时状态的任务
- 强结构化生成任务

这类场景更适合生成辅助型或执行增强型 RAG，甚至 Workflow / Agent 形态。

## 10. FAQ 型 RAG 在各类公司中的典型落地模式

近两年在各类公司中，FAQ 型 RAG 非常常见，常见落地方向包括：

- 企业内部知识助手
- 员工帮助台问答
- 开发者文档问答
- SaaS 产品使用助手
- 平台组件和 API 文档问答
- 客服知识库增强

本质上，只要“原来依赖搜索文档和翻页面”的场景，都很容易演进为 FAQ 型 RAG。

## 11. FAQ 型 RAG 的常见优化方向

### 11.1 FAQ 结构化条目优先

对于高频问题，FAQ 对问答对对象化往往比长文档切片更有效。

### 11.2 同义词和术语表建设

对 FAQ 场景提升通常非常明显。

### 11.3 官方来源和最新文档加权

可减少旧文档或社区内容造成的误导。

### 11.4 引导式回答结构

例如：

- 结论
- 操作步骤
- 注意事项
- 来源文档

这通常比自由回答更适合企业场景。

## 12. FAQ 型 RAG 的主要评测重点

建议重点评估：

- 问题命中率
- 正确来源是否被召回
- 回答是否 grounded
- citation 是否正确
- 高发问题上的稳定性
- 用户是否继续点击来源文档

## 13. 面向低代码平台的 FAQ 型 RAG 映射

在低代码平台中，FAQ 型 RAG 可以重点服务：

- 组件使用说明问答
- Schema 配置规则问答
- 平台功能使用问答
- 发布和运维规范问答
- 常见错误说明问答

这是最适合作为第一阶段落地的 RAG 形态之一。

## 14. 常见误区

### 14.1 认为 FAQ 型 RAG 很简单，不需要完整检索架构

实际上，它往往是用户量最大、最容易暴露质量问题的场景。

### 14.2 只做 dense retrieval

FAQ 场景的术语、简称和配置名非常多，hybrid 通常更稳。

### 14.3 不做 citation

企业 FAQ 如果没有来源，很难形成用户信任。

### 14.4 把所有文档都一股脑导入，不做结构化 FAQ 对象化

高频问题场景通常更适合 FAQ 条目化沉淀。

## 15. 一句话总结

FAQ 型 RAG 的核心，是把企业中高频、相对稳定、以解释和引导为主的知识问题，转化为可检索、可回答、可引用、可治理的知识服务能力；近两年的最佳实践也越来越清楚，真正有价值的 FAQ 型 RAG，已经不是“聊天读文档”，而是具备 query understanding、hybrid retrieval、citation 和安全治理的正式知识服务系统。

## 16. 参考资料

- Azure AI Search RAG overview: [https://learn.microsoft.com/en-us/azure/search/retrieval-augmented-generation-overview](https://learn.microsoft.com/en-us/azure/search/retrieval-augmented-generation-overview)
- Azure AI Search classic vs agentic retrieval: [https://learn.microsoft.com/en-us/azure/search/search-agentic-retrieval-overview](https://learn.microsoft.com/en-us/azure/search/search-agentic-retrieval-overview)
- OpenAI Retrieval guide: [https://platform.openai.com/docs/guides/retrieval](https://platform.openai.com/docs/guides/retrieval)
- Cohere Agentic RAG tutorial: [https://docs.cohere.com/v2/docs/agentic-rag](https://docs.cohere.com/v2/docs/agentic-rag)
