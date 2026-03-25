# 企业级AI治理边界

> 状态：第一版草案

## 1. 文档定位

本文档用于明确企业级 AI 系统的治理边界，回答：

- 哪些问题必须由治理体系负责
- AI 能力在企业里应该被限制到什么边界
- 为什么治理不是附属能力，而是系统设计的正式组成部分
- 在大模型、RAG、Tool、Agent 场景下，治理重点分别是什么

## 2. 为什么企业 AI 必须先谈治理边界

在个人产品或实验环境中，AI 更多关注“能不能做出来”。

但在企业环境中，更重要的问题通常是：

- 做错了怎么办
- 越权了怎么办
- 泄露数据怎么办
- 成本失控怎么办
- 无法审计怎么办
- 高风险动作误执行怎么办

这意味着：

企业级 AI 的核心，不只是“能力上限”，更是“能力边界”。

## 3. 行业最佳实战中的共识

结合近一阶段行业公开资料，可以看到几个越来越稳定的共识。

### 3.1 Tools 和 agents 越强，治理要求越高

OpenAI、Anthropic 的最新 agent 资料都在强调：当系统开始具备 tool use、computer use、长时任务和多步骤执行能力时，安全控制、审批、可观测性和 evals 必须同步增强。

### 3.2 企业 AI 不应是黑盒决策系统

Anthropic、OpenAI 和主流企业内部实践都越来越强调 trace、human-in-the-loop、evaluation、logging、replay，这说明“可解释可回放”已经成为 agentic system 的默认要求之一。

### 3.3 Guardrails 不能只靠 Prompt

当前主流最佳实战已经很明确：

- Prompt 是软约束
- Schema、rules、permission、审批、sandbox、tracing 才是硬边界

### 3.4 权限和租户隔离必须进入 AI 主链路

AI 不应绕过企业已有的权限体系，而应尽量继承和复用它。

## 4. 企业 AI 治理边界的总体定义

可以用一句话概括：

企业级 AI 治理边界，指的是系统在知识访问、输出生成、工具执行、数据处理、成本消耗和责任归属等方面可被允许、可被限制、可被审计的范围。

这个边界不是单一功能，而是一组控制面能力的集合。

## 5. 治理边界主要覆盖哪些维度

建议至少从七个维度定义企业 AI 治理边界。

### 5.1 身份边界

AI 请求必须知道“是谁发起的”。

至少应明确：

- 用户身份
- 组织身份
- 租户身份
- 应用身份
- 调用来源

如果连身份都不明确，后续权限、审计和责任归属都无从谈起。

### 5.2 数据边界

AI 系统不应默认能访问所有数据。

至少应明确：

- 哪类数据可检索
- 哪类数据不能进入模型上下文
- 哪些字段需要脱敏
- 哪些数据按租户隔离
- 哪些日志或配置只允许特定角色查看

### 5.3 输出边界

AI 输出不应无限制自由生成。

至少应明确：

- 哪些场景必须结构化输出
- 哪些场景必须引用来源
- 哪些场景禁止编造结论
- 哪些场景必须标记不确定性
- 哪些内容类型禁止输出

### 5.4 动作边界

当 AI 具备 Tool 或 Agent 能力时，必须定义它“可以做什么、不可以做什么”。

至少应明确：

- 哪些工具允许自动调用
- 哪些工具只能人工确认后调用
- 哪些动作必须审批
- 哪些动作绝不允许由 AI 自动执行

### 5.5 成本边界

AI 系统天然存在成本爆炸风险。

至少应明确：

- 哪些场景可用高成本模型
- 哪些场景必须降级
- 单请求成本上限是多少
- 单用户、单租户、单场景配额是多少

### 5.6 责任边界

企业 AI 系统必须明确“出了问题谁负责”。

至少应支持：

- 请求留痕
- 决策留痕
- 工具留痕
- 审批留痕
- 输出留痕

### 5.7 反馈边界

治理不是静态限制，还包括持续反馈与修正机制。

至少应具备：

- 质量反馈通道
- 风险事件上报
- 失败案例归档
- Prompt / Tool / retrieval 迭代依据

## 6. 四类核心治理对象

在企业 AI 系统中，治理对象可以进一步归纳为四类。

### 6.1 模型治理

关注：

- 模型选择
- 模型路由
- 模型 fallback
- 模型成本
- 模型安全配置
- 模型输出限制

### 6.2 知识治理

关注：

- 哪些知识可进入索引
- 哪些知识可被哪些用户检索
- 索引更新是否及时
- 元数据是否正确
- 是否有权限穿透风险

### 6.3 工具治理

关注：

- Tool 权限
- Tool 风险等级
- Tool 幂等性
- Tool 可审计性
- Tool 失败恢复策略

### 6.4 运行时治理

关注：

- Workflow 边界
- Agent 步数限制
- 重试限制
- 人工确认节点
- Trace 与回放
- 高风险终止条件

## 7. 企业里必须明确的高风险边界

一些动作天然属于高风险边界，原则上不应默认自动化。

典型包括：

- 发布
- 删除
- 写入生产配置
- 发消息 / 发通知
- 跨租户读取
- 修改权限
- 对外部系统进行写操作

这些场景至少应考虑：

- 人工确认
- 多级审批
- 沙箱验证
- 幂等控制
- 回滚方案

## 8. RAG 场景下的治理边界

RAG 场景看起来风险较低，但实际上依然有明显治理需求。

### 8.1 知识权限边界

检索结果必须与原始权限一致，不能因为做了 embedding 或建了索引就绕开权限控制。

### 8.2 输出事实边界

回答应尽量基于已检索内容，并标识引用来源。上下文不足时应明确说明，而不是补全猜测。

### 8.3 敏感信息边界

即使是问答系统，也必须避免把敏感字段、内部配置或不适合公开的运行信息直接暴露出来。

## 9. Agent 场景下的治理边界

Agent 场景的治理要求更高。

### 9.1 决策边界

Agent 不应自由决定所有行为。必须明确：

- 哪些决策可自动化
- 哪些决策只能给建议
- 哪些决策必须人工确认

### 9.2 工具边界

不同工具必须按风险等级分层。

例如：

- 查询类：低风险
- 生成草稿类：中风险
- 发布 / 删除 / 通知类：高风险

### 9.3 状态边界

Agent 应有：

- 最大步数
- 最大重试次数
- 超时限制
- 升级为人工处理的触发条件

### 9.4 证据边界

对于关键动作，必须保留：

- 请求上下文
- 决策依据
- 工具调用结果
- 人工确认记录

## 10. 治理边界不等于“把能力关死”

治理的目标不是让 AI 什么都做不了，而是：

- 让低风险能力可以稳定自动化
- 让中风险能力可控半自动化
- 让高风险能力留在人工和审批边界内

也就是说：

治理的目标是“有边界地放大能力”，而不是“简单禁止”。

## 11. 企业治理的推荐分层策略

可以按风险和能力成熟度，分三层设计。

### 11.1 低风险层

允许自动处理：

- FAQ 问答
- 知识检索
- 推荐解释
- 草稿生成

### 11.2 中风险层

允许受控处理：

- 配置建议
- 诊断建议
- 生成后校验
- 草稿创建

通常需要：

- 结构化校验
- 风险提示
- 操作留痕

### 11.3 高风险层

不建议默认自动化：

- 发布
- 删除
- 生产写操作
- 通知触发
- 权限变更

必须具备：

- 审批
- 人工确认
- 沙箱验证
- 回滚方案

## 12. 面向低代码平台如何定义治理边界

在低代码平台场景中，建议重点定义以下边界。

### 12.1 页面与 Schema 边界

- AI 可生成草稿，但不能默认直接发布
- AI 可做结构化建议，但必须经过协议校验
- AI 不能绕过 Schema 校验器直接写入正式配置

### 12.2 组件与物料边界

- AI 可推荐组件与配置方式
- AI 不能伪造不存在的组件能力
- AI 输出必须受组件协议和 setter 规则约束

### 12.3 运维边界

- AI 可查日志和状态
- AI 可给诊断建议
- AI 不应默认直接操作生产环境或跨租户环境

### 12.4 租户与权限边界

- AI 必须继承租户隔离
- AI 不应聚合展示用户无权访问的页面、配置和日志

## 13. 治理边界的工程化落点

治理不能只写在原则里，必须落到工程点位。

典型落点包括：

- 身份透传中间件
- 检索时权限过滤
- Tool registry 风险分级
- Runtime 中的审批节点
- 输出 Schema 校验
- 敏感信息脱敏器
- token / cost 统计器
- trace / audit / replay 体系

## 14. 常见误区

### 14.1 把治理理解为上线前补丁

治理应从架构设计阶段就纳入，而不是最后补一层过滤。

### 14.2 认为 Prompt 足够承担所有约束

Prompt 只能承担软约束，不能替代硬边界。

### 14.3 把高风险动作和低风险动作一视同仁

这样要么导致系统过于保守，要么导致风险失控。

### 14.4 没有 trace 和 audit 就做复杂 Agent

这几乎必然导致后续无法定位和追责。

## 15. 一句话总结

企业级 AI 治理边界的核心，是把模型、知识、工具和运行时都纳入身份、权限、风险、成本、审计和反馈闭环之中，让 AI 能力在可控范围内持续放大，而不是在失控风险中盲目扩张。

## 16. 参考资料

- OpenAI, A practical guide to building agents: [https://openai.com/business/guides-and-resources/a-practical-guide-to-building-ai-agents/](https://openai.com/business/guides-and-resources/a-practical-guide-to-building-ai-agents/)
- OpenAI, GPT-4o System Card, August 8, 2024: [https://cdn.openai.com/gpt-4o-system-card.pdf](https://cdn.openai.com/gpt-4o-system-card.pdf)
- Anthropic, Building Effective AI Agents, December 19, 2024: [https://www.anthropic.com/engineering/building-effective-agents](https://www.anthropic.com/engineering/building-effective-agents)
- Anthropic, Writing effective tools for agents, September 11, 2025: [https://www.anthropic.com/engineering/writing-tools-for-agents](https://www.anthropic.com/engineering/writing-tools-for-agents)
- Anthropic, Demystifying evals for AI agents, January 9, 2026: [https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents](https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents)
- Google, Gemini 2.5 Technical Report, 2025: [https://storage.googleapis.com/deepmind-media/gemini/gemini_v2_5_report.pdf](https://storage.googleapis.com/deepmind-media/gemini/gemini_v2_5_report.pdf)
- LangGraph Overview: [https://docs.langchain.com/oss/python/langgraph/overview](https://docs.langchain.com/oss/python/langgraph/overview)
