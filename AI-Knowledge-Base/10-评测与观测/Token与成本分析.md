# Token与成本分析

> 状态：第一版草案

## 1. 文档定位

本文档用于定义 AI 系统中的 Token & Cost Analytics 设计，重点回答：

- 为什么 token 分析是评测与观测体系的一部分，而不仅是财务报表问题
- token 指标应如何与场景质量、链路复杂度、模型路由和成本治理联动
- 在 RAG、Tool、Agent 场景下应关注哪些 token / cost 细分指标

## 2. 什么是 Token 与成本分析

Token 与成本分析可以理解为：

对 AI 请求在输入、输出、检索、结构化输出、工具调用和长任务运行中的 token 消耗、模型费用及相关资源成本进行结构化观测和解释的一套分析机制。

它不仅关心“花了多少钱”，还关心：

- 哪一步最耗 token
- 为什么某类任务突然变贵
- token 花出去后换来了什么质量收益
- 哪些上下文是低价值消耗

用一句话概括：

Token 与成本分析的本质，是从链路视角理解 AI 请求的资源消耗结构，而不是只看最终账单。

## 3. 为什么这部分应放在评测与观测里

### 3.1 token 消耗往往能直接暴露系统设计问题

例如：

- context 拼得太长
- retrieval 召回过多无效 chunk
- tool loop 无意义反复执行
- 结构化输出反复失败重试

### 3.2 成本分析必须与质量分析一起看

同样多花 2 倍 token：

- 如果质量显著提升，可能是合理投入
- 如果质量没有改善，就是系统效率问题

### 3.3 近两年的 Agent 和长上下文模型让 token 结构更复杂

现在不仅要看：

- input token
- output token

还要看：

- retrieval context token
- reasoning-related token overhead
- multi-turn accumulation
- multimodal token cost

## 4. Token 与成本分析要解决什么问题

### 4.1 找出高消耗链路

### 4.2 找出低价值 token 浪费

### 4.3 支撑路由和降级优化

### 4.4 为预算策略和发布策略提供依据

## 5. 推荐的分析维度

建议至少按以下维度分析：

### 5.1 场景维度

- FAQ 问答
- Schema 生成
- 配置诊断
- 运维分析
- 长运行 Agent

### 5.2 模型维度

- provider
- model name
- route tier

### 5.3 链路维度

- retrieval
- generation
- tool use
- approval path
- retries

### 5.4 租户 / 应用维度

看是否存在异常消耗租户、团队或业务线。

## 6. 推荐的核心指标

### 6.1 Token 指标

- input tokens
- output tokens
- avg tokens per request
- avg tokens per successful task
- avg context tokens
- avg tokens per retry

### 6.2 成本指标

- avg cost per request
- avg cost per successful task
- cost per scene
- cost per tenant
- cost per tool chain

### 6.3 效率指标

- token-to-success ratio
- cost-to-success ratio
- wasted token ratio
- retry cost ratio

## 7. 在 RAG 中如何分析 token 成本

RAG 场景重点关注：

### 7.1 retrieval context 占比

上下文占总输入 token 的比例是否过高。

### 7.2 chunk 冗余度

是否出现大量重复或低价值 chunk 被拼入。

### 7.3 citation / evidence 结构成本

grounded answer 是否因格式或引用设计过度膨胀。

### 7.4 rewrite / rerank 带来的额外成本

其成本是否换来了足够的质量收益。

## 8. 在 Agent 场景中如何分析 token 成本

Agent 场景重点关注：

### 8.1 多轮累计 token

任务运行越久，历史上下文越可能形成成本膨胀。

### 8.2 tool loop 成本

是否出现低价值循环：

- 重复查相同信息
- 重复调用失败工具
- 重复生成无效参数

### 8.3 planner / summary token 占比

规划和总结消耗是否合理。

### 8.4 approval / HITL 前后 token 变化

高风险链路是否引入了可接受的附加成本。

## 9. 推荐的分析方法

### 9.1 聚合报表

看趋势、分布、top cost scenes。

### 9.2 Trace 级分析

按请求链路拆到 span 级别看 token 和成本。

### 9.3 样本回看

人工看高成本失败样本，判断浪费来源。

### 9.4 版本对比

比较模型、prompt、索引、tool 版本切换前后的 token / cost 变化。

## 10. 面向低代码平台的典型场景

### 10.1 组件问答

重点看：

- retrieval context 是否过长
- citation 结构是否冗余
- FAQ 场景是否用了过重模型

### 10.2 Schema 生成

重点看：

- 模板召回上下文占比
- 结构化输出重试 token 成本
- 自动修复链路追加成本

### 10.3 配置诊断

重点看：

- 多工具并发后是否带来高额上下文合并成本
- 日志摘要是否过长

### 10.4 运维 Agent

重点看：

- 长任务的多轮累计 token
- 审批前分析链路的成本上限

## 11. 常见误区

### 11.1 只看总 token，不拆链路

这样无法知道成本浪费发生在哪。

### 11.2 只看平均值，不看异常长尾

高成本异常请求往往才最值得优化。

### 11.3 不把成本和成功率一起看

会得出片面的优化结论。

### 11.4 不分析 retry 和失败样本成本

很多浪费就隐藏在失败链路里。

### 11.5 只在月底做成本复盘

缺少实时优化和发布判断价值。

## 12. 一句话总结

Token 与成本分析的核心，是把 AI 系统每条链路的 token 消耗和费用拆解到可解释、可比较、可优化的粒度上，并和质量、成功率、风险、路由策略一起看；近两年的最佳实践也越来越明确，真正成熟的 AI 平台不是“会计费”，而是“会解释为什么花了这些 token，并知道哪些值得继续花”。

## 13. 参考资料

- OpenAI Usage Dashboard help: [https://help.openai.com/en/articles/8554956-understanding-the-usage-dashboard](https://help.openai.com/en/articles/8554956-understanding-the-usage-dashboard)
- OpenAI Pricing: [https://openai.com/api/pricing/](https://openai.com/api/pricing/)
- Anthropic, Demystifying evals for AI agents: [https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents](https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents)
- LangSmith observability: [https://docs.smith.langchain.com/observability](https://docs.smith.langchain.com/observability)
