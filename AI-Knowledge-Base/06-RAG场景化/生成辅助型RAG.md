# 生成辅助型RAG

> 状态：第一版草案

## 1. 文档定位

本文档用于定义生成辅助型 RAG 的业务形态与架构特点，重点回答：

- 什么样的场景属于“生成辅助型 RAG”
- 为什么这类场景与 FAQ 型 RAG 有本质区别
- 在结构化生成、代码生成、配置生成、页面生成等场景中，RAG 应如何发挥作用

## 2. 生成辅助型 RAG 的定义

生成辅助型 RAG 指的是：

检索结果不是最终答案本身，而是作为生成约束、模板参考、示例支持、规则依据或对象素材，辅助模型生成结构化结果、草稿或方案。

用一句话概括：

生成辅助型 RAG 不是“问到了就答”，而是“先检索，再借助知识生成”。

## 3. 为什么这类场景越来越重要

近两年很多企业 AI 场景的目标，已经从“回答问题”延伸到：

- 生成页面 / 表单 / 配置
- 生成 SQL / DSL / Schema
- 生成报告草稿
- 生成运维分析建议
- 生成代码片段和样板

这类任务最大的挑战通常不是语言生成本身，而是：

- 生成必须贴近企业对象
- 生成必须符合平台规则
- 生成不能脱离已有模板和样板
- 生成结果必须可校验、可接入系统

RAG 在这里的核心价值，是把生成约束提前补给模型。

## 4. 近两年的关键变化

### 4.1 RAG 正在从 answer grounding 扩展到 generation grounding

早期 RAG 主要强调“回答有依据”；现在很多高价值业务场景更强调“生成有依据”。

这意味着 retrieval 结果的作用发生了变化：

- 不只是证据
- 更是模板
- 也是规则
- 还是可复用示例

### 4.2 结构化输出能力强化，使生成辅助型 RAG 更实用

随着 structured outputs、JSON schema、response format 等能力成熟，生成辅助型 RAG 更容易进入生产环境。

### 4.3 领域对象检索变得比长文问答更重要

在生成场景中，最有价值的往往不是长段说明文，而是：

- 模板对象
- 配置样本
- schema 片段
- 规则条目
- 组件组合示例

这与 FAQ 型 RAG 的知识对象结构明显不同。

## 5. 典型业务场景

### 5.1 页面 / 表单 / Schema 生成

根据业务需求生成页面结构或配置草稿。

### 5.2 组件组合生成

根据需求推荐并生成一组组件配置。

### 5.3 代码和配置生成

例如：

- SQL 草稿
- DSL 配置
- 工作流节点定义
- API 请求样例

### 5.4 诊断建议生成

基于规则和案例生成结构化诊断结果或修复建议。

## 6. 与 FAQ 型 RAG 的本质区别

FAQ 型 RAG 的 retrieval 更偏“回答依据”。

生成辅助型 RAG 的 retrieval 更偏：

- 模板依据
- 规则依据
- 结构示例
- 对象素材

因此，其上下文构建和 answer generation 通常都更复杂。

## 7. 典型架构特点

生成辅助型 RAG 通常更强调：

- 结构化对象检索
- 规则与模板并存
- context packaging for generation
- structured outputs
- post-generation validation

典型链路通常是：

需求理解 -> query understanding -> 检索模板/规则/样本 -> 组织 generation context -> 结构化生成 -> 校验 -> 必要时修复

## 8. 为什么 retrieval 在生成场景中特别关键

如果 retrieval 不准，模型就可能：

- 用错模板
- 引用错规则
- 混入不兼容样例
- 生成不符合平台规范的结果

因此，生成辅助型 RAG 中 retrieval 的重要性甚至不低于 FAQ 场景。

## 9. 推荐的知识对象类型

这类场景最适合检索：

- 模板对象
- schema 样本
- 组件协议
- 约束规则
- 字段定义
- 历史优秀案例
- 失败案例和反例

而不只是检索长文说明。

## 10. 推荐的 context builder 组织方式

生成辅助型 RAG 通常不适合把结果组织成单纯文档拼接。

更适合按以下方式组织：

- 目标说明
- 约束规则
- 可参考模板
- 相似样本
- 注意事项
- 输出格式要求

这让模型更像“在规则和示例之间生成”，而不是“看文档写作文”。

## 11. 生成后为什么还要校验

生成辅助型 RAG 的最终结果通常要进入系统，因此必须考虑：

- JSON / Schema 合法性
- 组件协议合法性
- 字段完整性
- 风险约束
- 平台规则校验

因此，它通常天然需要和 Tool / Validator 结合。

## 12. 面向低代码平台的典型形态

### 12.1 页面 Schema 生成

检索：

- 页面模板
- 组件协议
- 数据源定义
- 布局样例

生成：

- Schema 草稿

### 12.2 逻辑编排生成

检索：

- 节点定义
- 连接规则
- 历史案例

生成：

- DAG / workflow 草稿

### 12.3 组件配置生成

检索：

- props 定义
- setter 规则
- 常见示例

生成：

- 配置片段

## 13. 各大公司常见落地思路

近两年大量企业把 RAG 用于：

- 文档到表单 / 页面生成
- 知识增强代码生成
- 模板增强写作
- 规则增强配置生成
- 诊断建议和修复草稿生成

其共同特点是：

检索结果不是终点，而是模型输出可控化的输入。

## 14. 主要评测重点

建议重点评估：

- 模板召回是否正确
- 规则召回是否完整
- 结构化输出合法率
- 生成与引用的一致性
- 后置校验通过率
- 用户采纳率 / 修改量

## 15. 常见误区

### 15.1 把生成辅助型 RAG 当成普通 FAQ 问答来做

### 15.2 不检索模板和结构对象，只检索长文说明

### 15.3 生成后不做结构校验

### 15.4 检索结果过多，导致模型混合冲突样例

## 16. 一句话总结

生成辅助型 RAG 的核心，是用检索得到的模板、规则、样例和结构对象为模型生成提供 grounded constraints，让输出更贴近企业对象、更可控、更可验证；这也是近两年 RAG 从“回答系统”走向“生成系统”的最重要扩展方向之一。

## 17. 参考资料

- OpenAI Structured Outputs guide: [https://platform.openai.com/docs/guides/structured-outputs](https://platform.openai.com/docs/guides/structured-outputs)
- OpenAI Retrieval guide: [https://platform.openai.com/docs/guides/retrieval](https://platform.openai.com/docs/guides/retrieval)
- Azure AI Search RAG overview: [https://learn.microsoft.com/en-us/azure/search/retrieval-augmented-generation-overview](https://learn.microsoft.com/en-us/azure/search/retrieval-augmented-generation-overview)
- Cohere Agentic RAG tutorial: [https://docs.cohere.com/v2/docs/agentic-rag](https://docs.cohere.com/v2/docs/agentic-rag)
