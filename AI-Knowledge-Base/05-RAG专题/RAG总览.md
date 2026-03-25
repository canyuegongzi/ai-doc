# RAG总览

> 状态：第一版草案

## 1. 文档定位

本文档用于从系统视角总结 Retrieval-Augmented Generation（RAG）的完整架构，回答：

- RAG 在企业 AI 应用中到底是什么角色
- 经典 RAG 和近两年兴起的 agentic retrieval / advanced RAG 有什么差异
- 一个生产级 RAG 系统应由哪些关键模块构成
- 为什么 RAG 仍然是当前 AI 业务应用中最广泛、最具性价比的落地形态

## 2. 为什么 RAG 是当前最广泛的 AI 业务应用形态

在企业场景中，最稳定、最容易证明价值的 AI 需求，往往集中在以下几类：

- 私有知识问答
- 文档搜索与解释
- 结构化生成前知识补充
- 推荐与导航
- 执行前规范查询

这些问题有一个共同点：

模型本身不具备所需知识，或者所需知识高度依赖企业内部、实时变化或领域语义。

RAG 正是解决这类问题的最直接手段，因此在当前 AI 业务应用中，它仍然是覆盖面最广的架构模式。

## 3. 近两年的关键变化

### 3.1 RAG 已从“向量检索 + 拼 Prompt”升级为完整检索架构

早期很多系统把 RAG 做成：

- embedding
- topK 向量召回
- 把结果拼进 Prompt
- 模型回答

但当前行业最佳实践已经越来越明确：生产级 RAG 至少要关注：

- query understanding
- hybrid recall
- rerank
- context builder
- citation
- security trimming
- evals

也就是说，RAG 现在更像完整的信息检索和上下文编排系统，而不只是“向量库接模型”。

### 3.2 agentic retrieval 正在成为高复杂场景的新方向

微软在 2025 年 Azure AI Search 文档中明确提出“agentic retrieval”，强调：

- LLM-assisted query planning
- multi-query decomposition
- parallel retrieval
- structured retrieval response

这代表一个明显趋势：

传统 single-query RAG 正在向 multi-query、planner-assisted retrieval 演进，尤其适合复杂问题、多源知识和 agent 场景。

### 3.3 RAG 与 Workflow / Agent 的边界越来越模糊

当前很多先进实践已经不再把 RAG 限定为“问答模块”，而是把它视为：

- Agent 的认知增强模块
- 生成型任务的约束输入层
- 多步骤任务中的动态检索子流程

这意味着 RAG 既可以独立服务问答，也越来越常作为更大 AI 系统的一部分存在。

## 4. RAG 的总体定义

可以将 RAG 定义为：

一套在生成前引入外部知识、通过检索和上下文构建对模型进行认知增强的系统架构，用于让模型在企业私有知识、最新知识和领域知识约束下生成更准确、更可解释、更可落地的结果。

用一句话概括：

RAG 的本质，是把“信息检索系统”和“生成模型”组合成一个可治理的知识增强系统。

## 5. RAG 解决什么问题

### 5.1 私有知识缺失

模型不知道企业内部知识，而 RAG 可以接入企业知识源。

### 5.2 知识过期

模型训练知识有时间边界，而 RAG 可以引入更新后的知识对象。

### 5.3 领域语义不足

模型不天然理解企业专有术语、配置对象、错误案例和平台规则，RAG 可以提供领域上下文。

### 5.4 可解释性不足

通过 citation 和来源保留，RAG 可以比纯模型生成更容易解释“答案依据来自哪里”。

## 6. RAG 不解决什么问题

### 6.1 不直接解决任务执行问题

RAG 负责“知道”，不直接负责“做事”。

### 6.2 不直接解决权限体系本身

RAG 需要依赖知识工程和检索系统中的权限继承与过滤，而不是模型天然理解权限。

### 6.3 不天然解决结构化输出与系统接入

RAG 可以提供知识约束，但最终输出仍需 Schema、校验、规则与治理体系。

## 7. 经典 RAG 的标准链路

经典 RAG 通常包含以下主链路：

1. 用户提问
2. Query 理解
3. Recall
4. Rerank
5. Context Builder
6. LLM 回答
7. Citation / 结果返回

如果从知识工程往前追溯，则完整链路通常是：

数据源分类 -> 采集 -> 解析 -> 清洗与标准化 -> chunking -> metadata -> 向量化 -> 索引构建 -> query pipeline -> 生成

## 8. 新一代 RAG 架构趋势

结合 2024-2026 年公开实践，可以把当前 RAG 架构演进概括为以下几条主线。

### 8.1 从 single-query RAG 走向 multi-query retrieval

系统不再只拿用户原句检索，而会：

- query rewrite
- query expansion
- sub-question decomposition
- parallel retrieval

### 8.2 从 dense-only 走向 hybrid retrieval

生产级检索越来越倾向：

- dense + sparse
- vector + keyword
- metadata filter + rerank

### 8.3 从答案生成走向结构化 retrieval response

在 agentic retrieval 场景中，检索结果本身会被组织成更结构化的中间结果，供 agent 或 workflow 继续消费。

### 8.4 从被动检索走向 task-aware retrieval

检索不再只服务问答，还服务：

- 结构化生成
- 工具调用前规范查询
- 工作流节点决策
- 错误分析与修复建议

### 8.5 从“检索更多”走向“上下文治理更强”

随着长上下文模型出现，真正难点反而更清楚：

- 检索什么
- 保留什么
- 丢掉什么
- 怎么组合
- 怎么引用

因此 context builder 的重要性越来越高。

## 9. 生产级 RAG 的核心模块

### 9.1 Knowledge Pipeline

负责准备知识资产。

### 9.2 Retrieval Engine

负责在线 query 理解、召回、重排和上下文构建。

### 9.3 Prompt / Context Assembly

负责把用户请求、检索结果、规则和输出要求组织成模型输入。

### 9.4 Answer Generation

负责基于上下文生成答案或结构化结果。

### 9.5 Governance Layer

负责：

- 权限过滤
- citation
- output validation
- trace
- evals

## 10. 经典 RAG 与 Agentic RAG 的区别

### 10.1 经典 RAG

特点：

- 一次查询
- 一次召回
- 一次上下文构建
- 一次生成

适用于：

- FAQ
- 文档问答
- 简单推荐
- 基础生成增强

### 10.2 Agentic RAG

特点：

- 复杂 query planning
- 多次检索
- 多源搜索
- 可能带有检索路径选择
- 更适合服务 agent 和复杂任务

适用于：

- 多问题复合问法
- 多源知识整合
- 工具调用前规范查询
- 复杂诊断或分析场景

### 10.3 两者关系

Agentic RAG 不是对经典 RAG 的否定，而是在复杂场景下对 retrieval pipeline 的升级。

## 11. RAG 的四类主要业务形态

### 11.1 FAQ / 问答型 RAG

回答“这是什么、怎么用、为什么这样”。

### 11.2 搜索推荐型 RAG

回答“给我找什么、推荐什么、导航到哪里”。

### 11.3 生成增强型 RAG

回答“生成之前需要参考什么约束和样本”。

### 11.4 执行增强型 RAG

回答“执行任务前，需要查什么规则、案例和协议”。

## 12. 当前企业 RAG 架构设计的最佳实践

结合近两年的实践，生产级 RAG 通常应满足以下特征：

### 12.1 采用 hybrid retrieval，而不是只押 dense

### 12.2 显式做 query understanding，而不是直接原句检索

### 12.3 把 rerank 作为标准组件，而不是可选项

### 12.4 重视 context builder，而不是 topK 全塞给模型

### 12.5 把权限过滤、citation 和 evals 作为正式能力

### 12.6 让 RAG 能服务问答、生成和 agent，而不是局限于 FAQ

## 13. 面向低代码平台如何理解 RAG

在低代码平台场景中，RAG 最常见也最有价值的形态包括：

- 组件知识问答
- Schema 模板与页面模板推荐
- 生成前约束检索
- 错误案例检索
- 平台规范检索

此时，RAG 的知识源往往不只是产品文档，还包括：

- 组件协议
- setter / slot 定义
- 页面 Schema 样本
- 配置错误案例
- 运维规范

因此，低代码平台的 RAG 更接近“平台知识增强引擎”。

## 14. 常见误区

### 14.1 把 RAG 等同于向量库

### 14.2 只做 single-query dense retrieval

### 14.3 不做 metadata 和权限过滤

### 14.4 不做 citation 和 context compression

### 14.5 认为长上下文会替代 RAG

长上下文会改变部分设计，但不会消灭企业场景中的检索、过滤、更新与治理需求。

## 15. 一句话总结

RAG 总览的核心结论是：当前最广泛落地的 AI 业务架构，已经从“向量召回 + Prompt 拼接”的经典形态，演进为“知识工程 + hybrid retrieval + query planning + rerank + context builder + governance”的完整知识增强系统；而在复杂场景中，它还正在进一步演化为 agentic retrieval 的基础能力。

## 16. 参考资料

- Azure AI Search RAG overview: [https://learn.microsoft.com/en-us/azure/search/retrieval-augmented-generation-overview](https://learn.microsoft.com/en-us/azure/search/retrieval-augmented-generation-overview)
- Azure AI Search agentic retrieval: [https://learn.microsoft.com/en-us/azure/search/search-agentic-retrieval-concept](https://learn.microsoft.com/en-us/azure/search/search-agentic-retrieval-concept)
- OpenAI Retrieval guide: [https://platform.openai.com/docs/guides/retrieval](https://platform.openai.com/docs/guides/retrieval)
- OpenAI File Search guide: [https://platform.openai.com/docs/guides/tools-file-search/](https://platform.openai.com/docs/guides/tools-file-search/)
- Cohere Agentic RAG tutorial: [https://docs.cohere.com/v2/docs/agentic-rag](https://docs.cohere.com/v2/docs/agentic-rag)
- Weaviate blog: Agentic RAG: [https://weaviate.io/blog/what-is-agentic-rag](https://weaviate.io/blog/what-is-agentic-rag)
