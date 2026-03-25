# ChatModel设计

> 状态：第一版草案

## 1. 文档定位

本文档用于定义企业 AI 平台中 `ChatModel` 或更广义的 `ResponseModel` 设计，回答：

- 面向对话、生成、推理和 tool use 的模型接口应该如何统一抽象
- 当前行业从 Chat Completions 向 Responses-style interface 演进后，模型接口设计应该如何升级
- 如何在统一接口下兼容不同供应商的高级能力

## 2. 为什么 ChatModel 设计是模型层最关键的接口

在大多数 AI 应用里，最常被调用的仍然是“生成类模型”。

无论是：

- 问答
- 摘要
- 内容生成
- 结构化输出
- 推理
- tool use 决策
- 多模态理解后的回答

背后通常都会落到某种 chat / response generation 模型能力上。

因此，ChatModel 设计几乎决定了：

- 场景开发体验
- Prompt 组织方式
- structured outputs 接入方式
- tool use 接入方式
- streaming 体验
- 模型路由和治理能力

## 3. 行业最佳实战与最新趋势

### 3.1 从 Chat Completions 抽象走向 Responses 抽象

OpenAI 当前已明确推动从 Chat Completions 向 Responses API 迁移，这一变化说明现代生成模型接口不应只围绕“聊天消息”设计，而应围绕更通用的“响应生成”设计。

这带来的启发是：

- 接口要支持文本与多模态输入
- 要支持结构化输出
- 要支持 tool use
- 要支持 reasoning-capable models
- 要支持 built-in tools 与外部 tools 共存

### 3.2 多供应商接口正在趋同，但尚未真正统一

Anthropic、Google、OpenAI、Bedrock 等都越来越强调：

- messages-based input
- tool calling
- structured outputs
- multimodal input
- streaming

但它们的字段命名、参数语义、支持能力和错误模型仍有差异，因此平台仍然需要统一设计。

### 3.3 “ChatModel” 这个名字已经有些偏窄

从纯工程角度讲，`ResponseModel` 可能更准确，因为当前一代模型已经不仅用于聊天，而是用于：

- reasoning
- tool orchestration
- schema generation
- multimodal interpretation
- evaluator model

不过在很多工程体系里，`ChatModel` 仍然是一个习惯命名，因此本文保留该命名，但语义上按更广义的响应生成模型理解。

## 4. ChatModel 的职责边界

### 4.1 应负责什么

- 接收统一输入上下文
- 生成文本或结构化响应
- 支持 messages / instructions / tools / schema
- 支持 streaming
- 输出 usage metadata
- 暴露 capability metadata

### 4.2 不应负责什么

- 不直接负责 RAG
- 不直接负责 Workflow / Agent 状态机
- 不直接负责业务工具执行
- 不直接承担权限控制主逻辑

这些能力由上层编排、检索、Tool 和治理系统完成。

## 5. 推荐的核心输入结构

建议把 ChatModel 输入统一抽象为以下几个区域。

### 5.1 instructions

用于放系统规则、角色边界、任务约束。

### 5.2 messages

用于表达当前会话内容、历史交互、tool messages、assistant messages 等。

### 5.3 input content

用于承载文本、多模态输入、外部上下文片段。

### 5.4 tools

用于描述当前允许使用的工具。

### 5.5 response schema

用于声明结构化输出格式。

### 5.6 execution options

例如：

- temperature
- max output tokens
- streaming
- reasoning mode / effort
- timeout
- tool choice policy

## 6. 推荐的输出结构

建议统一输出至少包括：

- `content`
- `structuredData`（如果有）
- `toolCalls`（如果有）
- `stopReason`
- `usage`
- `modelInfo`
- `providerResponseMetadata`

这样可以保证无论模型返回的是文本、JSON 还是 tool call，都能进入统一处理链路。

## 7. 关键能力设计点

### 7.1 结构化输出

现代 ChatModel 必须原生支持结构化输出，而不是让业务方自己从自由文本中解析 JSON。

推荐至少支持：

- response schema
- strict structured outputs
- fallback validator

### 7.2 Tool Use

ChatModel 应支持：

- tool definitions
- tool choice policy
- tool call extraction
- tool results re-injection

但真正的 Tool 执行仍由 Tool Registry / Runtime 负责。

### 7.3 Streaming

对于长输出和交互场景，streaming 是重要体验能力。

统一抽象时应至少考虑：

- 增量文本 token
- structured chunk
- tool call event
- usage finalization event

### 7.4 Multimodal Input

当前行业趋势已经表明，多模态能力正在成为主流而不是附加项。ChatModel 抽象不应只支持纯文本输入。

### 7.5 Reasoning-specific Controls

最新推理模型通常会有额外的推理控制项，例如 reasoning effort、思考模式、长推理时间等。

统一抽象时不应强行抹平，而应允许 optional extension。

## 8. 能力声明建议

建议 ChatModel 至少声明以下能力：

- supportsStreaming
- supportsStructuredOutput
- supportsToolUse
- supportsVisionInput
- supportsReasoningControls
- supportsLongContext
- maxContextWindow
- maxOutputTokens

这有助于上层场景和路由系统做能力匹配。

## 9. 统一接口示意

可参考如下抽象思路：

```ts
interface ChatModel {
  capability: 'chat';
  modelId: string;
  provider: string;
  features: ChatModelFeatures;
  generate(request: ChatRequest): Promise<ChatResponse>;
  stream(request: ChatRequest): AsyncIterable<ChatStreamEvent>;
}
```

其中：

- `ChatRequest` 表达统一输入契约
- `ChatResponse` 表达统一返回结构
- `ChatStreamEvent` 表达流式增量事件

## 10. 当前场景中的常见路由策略

在企业平台里，ChatModel 通常不会只对应一个模型，而是由网关路由。

典型路由包括：

- FAQ / 轻问答 -> 快速低成本模型
- Schema 生成 -> 结构化输出更稳的模型
- 复杂诊断 -> reasoning model
- 截图理解 -> multimodal model

因此，ChatModel 设计既要统一接口，也要适配动态路由。

## 11. 面向低代码平台如何设计 ChatModel 使用方式

在低代码平台里，ChatModel 的典型用途包括：

- 组件知识问答
- 页面需求转结构化说明
- Schema 生成
- 配置错误解释
- 运维问题总结

这意味着应特别支持：

- 结构化结果输出
- 检索上下文注入
- 工具调用结果回灌
- 多模态页面截图输入

## 12. 常见误区

### 12.1 把 ChatModel 设计成单纯字符串输入输出

这会直接限制 structured outputs、tool use 和 multimodal 能力。

### 12.2 把所有高级能力都藏到 provider-specific hacks 里

这会导致统一接口失去意义。

### 12.3 为了统一接口，抹平掉 reasoning、schema、tool 等关键差异

这样虽然“统一”了，但业务方无法真正利用模型能力。

## 13. 一句话总结

ChatModel 设计的核心，不是封装一个聊天接口，而是构建一个能统一承载文本生成、结构化输出、tool use、多模态输入和 reasoning 能力的响应生成契约，让上层场景面向能力开发，而不是面向供应商 API 开发。

## 14. 参考资料

- OpenAI Responses API: [https://platform.openai.com/docs/api-reference/responses](https://platform.openai.com/docs/api-reference/responses)
- OpenAI Structured Outputs: [https://platform.openai.com/docs/guides/structured-outputs](https://platform.openai.com/docs/guides/structured-outputs)
- Anthropic Messages API: [https://docs.anthropic.com/en/api/messages](https://docs.anthropic.com/en/api/messages)
- Anthropic Tool use docs: [https://docs.anthropic.com/en/docs/build-with-claude/tool-use](https://docs.anthropic.com/en/docs/build-with-claude/tool-use)
- Google Gemini API docs: [https://ai.google.dev/gemini-api/docs](https://ai.google.dev/gemini-api/docs)
