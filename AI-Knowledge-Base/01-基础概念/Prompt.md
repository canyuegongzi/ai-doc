# Prompt

> 状态：第一版草案

## 1. 定义

Prompt 指的是输入给模型的指令、上下文、约束、示例和任务描述的组合。

它既可以是一段简单文本，也可以是更复杂的消息结构、系统指令、变量模板、检索上下文、工具定义、输出 Schema、多轮历史甚至多模态输入的组合。

用一句话概括：

Prompt 的本质，是模型行为的直接控制接口。

## 2. 为什么 Prompt 仍然重要

随着模型能力持续提升，很多人会误以为 Prompt 已经不重要了。实际上，Prompt 的重要性并没有下降，只是重点发生了变化。

过去 Prompt 更像“技巧集合”，强调：

- 怎么写更聪明
- 怎么诱导模型按某种方式回答
- 怎么做 few-shot
- 怎么让模型一步步思考

而现在 Prompt 更像“系统接口设计”，强调：

- 如何清楚表达任务边界
- 如何把外部知识和工具能力组织给模型
- 如何约束输出格式
- 如何让 Prompt 可复用、可版本化、可评测
- 如何与 RAG、Tool、Workflow、Agent 协同

因此，Prompt 在现代 AI 应用里不是可有可无的小技巧，而是连接模型能力与业务目标的第一层控制面。

## 3. Prompt 解决什么问题

### 3.1 任务定义问题

模型能力很强，但它并不知道当前到底要做什么。

Prompt 负责明确：

- 当前任务是什么
- 输出要达到什么目标
- 输出风格和粒度是什么
- 成功标准是什么

### 3.2 上下文组织问题

企业场景下，模型往往需要结合：

- 用户问题
- 业务规则
- 检索内容
- 工具定义
- 历史上下文
- 输出约束

Prompt 负责把这些信息组织成模型可消费的结构。

### 3.3 输出约束问题

如果没有 Prompt 层的明确约束，模型很容易输出：

- 粒度不对
- 结构不稳
- 语气不对
- 缺少必要字段
- 未引用上下文依据

### 3.4 行为引导问题

Prompt 可以引导模型：

- 先判断任务类型
- 先查知识再回答
- 需要时调用工具
- 遵守安全边界
- 以结构化格式返回结果

## 4. Prompt 不解决什么问题

### 4.1 不替代知识工程

Prompt 再好，也不能弥补知识库脏、Chunk 切错、Metadata 缺失、索引更新失效的问题。

### 4.2 不替代 Tool 和 Workflow

Prompt 可以描述“应该如何做”，但不能替代真正的工具执行、状态机管理和流程编排。

### 4.3 不替代治理机制

Prompt 可以表达限制，但不能替代：

- 权限控制
- 审批机制
- 结构化校验
- 审计与追踪

### 4.4 不保证稳定性

Prompt 是重要控制手段，但它本质上仍然是软约束。真实生产环境必须配合显式 Schema、规则校验、Guardrails 和评测机制。

## 5. Prompt 的基本组成

在工程化场景中，一个完整 Prompt 往往由多个部分组成。

### 5.1 角色与身份设定

例如：

- 你是一个企业内部知识助手
- 你是一个低代码页面生成助手
- 你是一个配置诊断助手

角色设定的重点不是“人格化”，而是明确职责边界。

### 5.2 任务说明

明确当前任务目标。

例如：

- 根据提供的知识回答问题
- 根据上下文生成 JSON Schema
- 判断是否需要调用工具

### 5.3 输入上下文

包括：

- 用户输入
- 历史对话
- 检索结果
- 工具说明
- 业务规则
- 当前状态

### 5.4 输出要求

例如：

- 必须返回 JSON
- 必须引用来源
- 禁止编造未提供的信息
- 先给结论，再给依据

### 5.5 约束条件

例如：

- 只能基于提供内容回答
- 不要输出权限外信息
- 无法确认时明确说明不确定

### 5.6 示例

在必要时，通过示例帮助模型更准确地理解输入输出模式。

## 6. Prompt 的常见类型

### 6.1 指令型 Prompt

直接描述任务和输出要求。

适用于：

- 摘要
- 改写
- 分类
- 抽取

### 6.2 结构化输出 Prompt

强调输出格式、字段和约束。

适用于：

- JSON 输出
- DSL 生成
- Schema 生成
- Tool 参数生成

### 6.3 RAG Prompt

将检索结果与任务说明结合，引导模型基于外部知识回答。

适用于：

- 企业文档问答
- FAQ
- 推荐解释
- 生成辅助

### 6.4 Tool Use Prompt

引导模型识别何时应该调用工具、如何调用工具、如何使用工具结果。

### 6.5 Workflow / Agent Prompt

把当前节点状态、目标、历史动作和可用工具传给模型，用于局部决策。

## 7. 当前行业前沿变化

近两年 Prompt 工程最大的变化，不是技巧更多了，而是 Prompt 的工程定位发生了变化。

### 7.1 变化一：从“写提示词技巧”走向“系统接口设计”

OpenAI 当前官方文档已经不再把 Prompt 只讲成文字技巧，而是强调：

- 使用消息角色
- 用结构化输入组织上下文
- 使用 Prompt 对象、版本和变量
- 结合 evals 持续验证 Prompt 质量

这意味着 Prompt 已经从个人经验型能力，转向团队协作和系统设计能力。

### 7.2 变化二：新模型对 Prompt 的“脆弱技巧依赖”在下降

随着更强模型和 reasoning model 出现，很多早期复杂 prompt 技巧的重要性正在下降。

例如：

- 一些模型更擅长直接遵循清晰指令
- 更强调上下文质量，而不是奇技淫巧
- 更强调任务定义与评测，而不是堆砌咒语式模板

这不意味着 Prompt 不重要，而是意味着：

Prompt 设计的重点从“巧”转向“清晰、结构化、可测量”。

### 7.3 变化三：Prompt 与 Schema、Tool、Evals 正在深度结合

现代 Prompt 工程越来越少孤立存在，而是和以下能力组合：

- JSON Schema
- function / tool definitions
- structured output
- evals
- prompt versioning
- prompt caching

这意味着 Prompt 已经进入“资产化”阶段，而不是单次聊天技巧阶段。

### 7.4 变化四：Prompt 越来越依赖上下文编排而不是单段文本

企业场景中，一个真正有效的 Prompt 往往不是单独几句话，而是以下内容的组合：

- 系统规则
- 场景模板
- RAG 上下文
- 工具协议
- 当前任务状态
- 输出约束

因此，Prompt 的质量越来越取决于“上下文编排能力”。

### 7.5 变化五：Prompt 开始进入版本管理和平台化管理

OpenAI 当前文档已经直接把 Prompt Objects、变量、版本化和 Playground 生成能力纳入官方流程。这反映出行业正在形成共识：

Prompt 不应只散落在代码里，而应逐步成为：

- 可复用资产
- 可版本化对象
- 可测试对象
- 可共享配置

## 8. Prompt 的设计原则

### 8.1 明确任务，不要让模型猜任务

Prompt 应明确：

- 做什么
- 基于什么做
- 产出什么
- 不允许做什么

### 8.2 尽量结构化，而不是纯散文式描述

相比长篇自由说明，更推荐：

- 明确分段
- 明确字段
- 明确规则
- 明确输入输出边界

### 8.3 先给约束，再给材料，再给任务

在很多企业场景中，比较稳的结构通常是：

- 角色和规则
- 输入上下文
- 外部知识
- 当前任务
- 输出格式

### 8.4 优先使用显式结构约束

只要输出需要进入系统，就应尽量配合：

- JSON Schema
- 结构化响应
- 字段定义
- 校验器

而不是把所有要求都写成自然语言。

### 8.5 Prompt 必须进入评测闭环

Prompt 的优劣不应靠个人感觉，而应通过样本和评测集持续验证。

## 9. Prompt 与 RAG 的关系

RAG 并不是把检索结果简单拼在用户问题后面。

真正有效的 RAG Prompt 通常需要明确：

- 检索结果只是依据，不是任意发挥材料
- 当上下文不足时应明确说明
- 回答应优先引用检索内容
- 不得编造上下文中不存在的事实

也就是说，Prompt 决定模型如何使用检索结果。

## 10. Prompt 与 Tool Calling 的关系

在 Tool Calling 场景中，Prompt 往往承担三个作用：

- 告诉模型当前有哪些工具
- 告诉模型什么情况下应调用工具
- 告诉模型工具调用后如何继续推进任务

但需要强调：

Tool 的真实边界仍应由系统协议和权限机制决定，而不是只靠 Prompt 约束。

## 11. Prompt 与 Workflow / Agent 的关系

在 Workflow / Agent 场景中，Prompt 不再只是任务描述，而更像局部决策接口。

它通常需要结合：

- 当前节点状态
- 历史动作
- 当前目标
- 可用工具
- 风险限制
- 终止条件

因此，Agent Prompt 的难点往往不在文案本身，而在状态上下文组织。

## 12. Prompt 与 Guardrails 的关系

Prompt 可以表达边界，但 Guardrails 才是系统级护栏。

两者关系应理解为：

- Prompt 负责“引导模型”
- Guardrails 负责“约束系统”

企业级设计中，不能把关键风险控制全压在 Prompt 上。

## 13. Prompt 资产化为什么重要

在真实项目里，Prompt 不是一次性文本，而是长期资产。

原因包括：

- 不同业务场景会积累不同 Prompt 模板
- Prompt 会随着模型版本变化而演进
- Prompt 需要和 A/B 测试、评测集、效果反馈关联
- Prompt 需要跨团队共享和管理

因此，Prompt Center 或 Prompt Registry 往往是平台化建设的重要组成部分。

## 14. 常见误区

### 14.1 把 Prompt 工程理解成“写咒语”

现代 Prompt 工程更接近明确需求、组织上下文、定义接口与约束，而不是靠神秘 wording 获得奇效。

### 14.2 认为 Prompt 可以替代系统设计

Prompt 很重要，但不能替代：

- RAG
- Tool 系统
- Workflow
- Guardrails
- 权限控制
- 评测与观测

### 14.3 只关注 Prompt 文案，不关注输入材料质量

Prompt 的效果高度依赖上下文质量。如果检索结果不对、状态信息不全、工具定义含糊，再强的 Prompt 也救不了。

### 14.4 Prompt 不做版本管理

如果 Prompt 散落在代码和各类脚本里，很快就会失控，难以评估到底哪个版本带来了效果变化。

## 15. 面向低代码平台如何理解 Prompt

在低代码平台场景中，Prompt 的价值不在于“多会聊天”，而在于它是否能够准确组织以下信息：

- 页面生成目标
- 组件和模板知识
- Schema 协议约束
- Setter 与 slot 规则
- 数据源定义
- 校验规则
- 输出结构要求

因此，低代码平台里的 Prompt 往往不是单条提示词，而是：

- 场景模板
- 领域规则
- 检索上下文
- 结构化输出要求
- 工具定义

共同构成的一套“场景化提示协议”。

## 16. 一句话总结

Prompt 的本质，是模型行为的直接控制接口；而当前行业的变化，正在把 Prompt 从“提示词技巧”升级为“上下文编排、结构化约束、版本管理与评测驱动的系统资产”。

## 17. 参考资料

以下资料用于本篇第一版整理，优先采用官方资料：

- OpenAI Prompting Guide: [https://platform.openai.com/docs/guides/prompting](https://platform.openai.com/docs/guides/prompting)
- OpenAI Prompt Engineering Guide: [https://platform.openai.com/docs/guides/prompt-engineering](https://platform.openai.com/docs/guides/prompt-engineering)
- OpenAI Best practices for prompt engineering with the API: [https://help.openai.com/en/articles/6654000-comprehensive-step-by-step-guide-to-prompt-engineering-with-chatgpt](https://help.openai.com/en/articles/6654000-comprehensive-step-by-step-guide-to-prompt-engineering-with-chatgpt)
- OpenAI Prompt generation guide: [https://platform.openai.com/docs/guides/prompt-generation](https://platform.openai.com/docs/guides/prompt-generation)
- Anthropic Prompt Engineering Overview: [https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/overview](https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/overview)
- Anthropic Building Effective AI Agents, December 19, 2024: [https://www.anthropic.com/engineering/building-effective-agents](https://www.anthropic.com/engineering/building-effective-agents)
