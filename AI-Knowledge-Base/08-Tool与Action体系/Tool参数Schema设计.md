# Tool参数Schema设计

> 状态：第一版草案

## 1. 文档定位

本文档用于定义 Tool Input Schema / Output Schema 的设计原则，重点回答：

- 为什么参数 Schema 是 Tool 质量的核心组成，而不是实现细节
- 面向模型使用时，Schema 设计与传统 API 参数设计有什么不同
- 如何通过 Schema 设计提升模型填参成功率、降低风险和提升治理能力

## 2. 为什么 Tool 参数 Schema 很重要

模型调用工具时，最容易出错的环节通常不是“要不要调这个工具”，而是：

- 参数名理解错
- 字段缺失
- 类型不对
- 枚举值选错
- 字段间约束没有满足
- 不该填的字段乱填

这说明参数 Schema 不是普通文档附属物，而是模型与工具交互稳定性的核心界面。

用一句话概括：

Tool 参数 Schema 的本质，是把自然语言意图压缩成系统可执行结构的关键转换层。

## 3. 近两年的关键变化

### 3.1 Schema 正从“接口定义”演进为“模型约束接口”

在传统 API 设计中，Schema 主要服务：

- 前后端协作
- 服务间集成
- 参数校验

而在 Agent 场景中，Schema 还要额外服务：

- 模型理解字段语义
- 模型正确生成参数
- runtime 进行结构化校验
- 高风险参数自动触发策略

### 3.2 结构化输出能力提升，反向提高了 Schema 重要性

OpenAI Structured Outputs、Anthropic tool use、Gemini function calling 等能力越来越成熟，意味着：

现在系统已经可以更稳定要求模型按结构输出。

但前提是 Schema 自身必须设计得合理。

### 3.3 工具实战开始强调“防错型 Schema”

近两年的最佳实践越来越强调：

不是把后端原始参数照搬给模型，而是重新设计一层面向模型的参数契约。

## 4. Tool 参数 Schema 需要解决什么问题

### 4.1 让模型理解输入字段含义

字段名、描述、枚举和示例共同决定模型是否能正确填参。

### 4.2 让 runtime 做强校验

避免非法请求直接流入执行层。

### 4.3 让治理策略有抓手

Schema 可以承载：

- 高风险字段识别
- 环境选择限制
- 跨租户参数拦截
- 只读 / 可写动作区分

### 4.4 让 replay、eval 和审计更稳定

结构化参数是后续比对、重放、回归测试的基础。

## 5. 面向模型的 Schema 设计与传统 API 设计有何不同

### 5.1 语义清晰优先于“后端字段原样暴露”

传统后端字段可能长这样：

- `bizCode`
- `resType`
- `opFlag`
- `envId`

这些字段对模型并不友好。

更适合模型的方式是：

- `applicationCode`
- `resourceType`
- `operationType`
- `targetEnvironment`

### 5.2 更强调枚举、默认值、约束和示例

模型不是人工开发者，它更依赖：

- enum
- description
- examples
- required
- mutually exclusive constraints

### 5.3 更强调防错而不是“完全灵活”

过于自由的 Schema 会显著提高模型出错概率。

## 6. 推荐的 Schema 设计原则

### 6.1 字段名要语义明确

避免使用：

- 模糊缩写
- 内部黑话
- 复用含义不清的通用字段

### 6.2 字段描述要写动作语义，而不是只写技术类型

不应只写：

- `string`
- `number`
- `array`

更应写清：

- 这个字段表示什么
- 什么时候必填
- 应填写什么范围
- 哪些值不允许出现

### 6.3 必填字段尽量少但关键

过多必填字段会导致模型填参失败率上升。

### 6.4 枚举优先于自由文本

只要业务上可以枚举，就尽量避免让模型自由生成字符串。

### 6.5 复杂对象应分层而不是一次塞平

对复杂工具输入，建议拆成明确子对象，例如：

- `target`
- `constraints`
- `executionOptions`
- `reviewContext`

### 6.6 高风险字段要显式标识

例如：

- `targetEnvironment`
- `forceOverwrite`
- `deleteMode`
- `notificationReceivers`

这些字段应配合 policy 做额外校验。

## 7. 推荐的输入 Schema 结构

一个较稳的输入 Schema 通常可以拆成：

### 7.1 目标对象

描述本次动作作用于谁。

例如：

- pageId
- componentName
- appCode
- tenantCode

### 7.2 动作参数

描述本次动作要做什么。

例如：

- operationType
- patch
- query
- schemaDraft

### 7.3 约束参数

描述边界条件。

例如：

- targetEnvironment
- dryRun
- allowPartial
- maxResults

### 7.4 审批 / 追踪参数

例如：

- requestReason
- changeTicketId
- approvalToken

## 8. 输出 Schema 也应正式设计

很多系统只重视输入 Schema，但在 Agent 系统里，输出 Schema 同样重要。

原因是输出结果会继续被：

- planner 使用
- validation 使用
- downstream tool 使用
- audit / replay 使用

推荐输出至少区分：

- `status`
- `summary`
- `data`
- `warnings`
- `nextSuggestedAction`
- `error`

## 9. 示例：一个更适合 Agent 的 Schema 设计

不推荐：

```json
{
  "pageId": "string",
  "env": "string",
  "flag": 1
}
```

更推荐：

```json
{
  "type": "object",
  "properties": {
    "pageId": {
      "type": "string",
      "description": "目标页面的唯一标识"
    },
    "targetEnvironment": {
      "type": "string",
      "enum": ["dev", "test", "prod"],
      "description": "目标执行环境。涉及 prod 时通常需要人工审批"
    },
    "dryRun": {
      "type": "boolean",
      "description": "是否只做校验与风险评估，而不真正执行变更"
    }
  },
  "required": ["pageId", "targetEnvironment"]
}
```

## 10. Schema 与风险治理的关系

参数 Schema 不只是输入格式，它也是治理抓手。

例如可以基于字段做：

- 环境风险识别
- 敏感动作识别
- 高风险参数拦截
- 自动切换 HITL
- 审计重点字段记录

换句话说：

风险治理很多时候并不是靠额外逻辑“猜”，而是靠 Schema 设计时就把风险点显式化。

## 11. Schema 与 Prompt / Tool Description 的关系

即使有了 JSON Schema，也不代表模型一定理解字段语义。

更稳定的方式是：

- Schema 提供结构约束
- Tool description 提供工具边界
- 字段 description 提供参数语义
- few-shot examples 提供填参参照

这四者共同决定模型填参质量。

## 12. 面向低代码平台的参数 Schema 设计建议

在低代码平台中，建议把参数围绕平台对象来抽象，而不是暴露底层服务细节。

例如：

- 页面类工具重点围绕 `pageId`、`appCode`、`schemaDraft`
- 组件类工具重点围绕 `componentName`、`slotName`、`propsPatch`
- 运维类工具重点围绕 `buildId`、`releaseId`、`targetEnvironment`
- 发布类工具重点围绕 `riskCheckOnly`、`changeReason`、`approvalToken`

并尽量避免直接暴露后端内部字段如：

- 内部数据库主键
- 不可解释状态码
- 模糊标记位

## 13. 常见误区

### 13.1 把后端 DTO 原样暴露给模型

这通常既难理解又不安全。

### 13.2 参数字段太自由

模型容易输出格式正确但业务语义错误的值。

### 13.3 没有枚举和字段描述

会显著提高填参随机性。

### 13.4 输出 Schema 缺失

后续 planner、runtime、audit 都会变得不稳定。

### 13.5 不把风险字段显式化

会导致高风险动作难以在运行前被拦截。

## 14. 一句话总结

Tool 参数 Schema 设计的核心，不是把后端接口字段包装成 JSON，而是构建一层面向模型理解、面向 runtime 校验、面向治理控制的结构化动作契约；近两年的最佳实践也越来越明确，高质量 Agent 系统的稳定性，往往直接取决于参数 Schema 是否清晰、收敛、可校验。

## 15. 参考资料

- OpenAI, Structured outputs: [https://platform.openai.com/docs/guides/structured-outputs](https://platform.openai.com/docs/guides/structured-outputs)
- OpenAI Responses API tools guide: [https://platform.openai.com/docs/guides/tools](https://platform.openai.com/docs/guides/tools)
- Anthropic, Writing effective tools for agents: [https://www.anthropic.com/engineering/writing-tools-for-agents](https://www.anthropic.com/engineering/writing-tools-for-agents)
- Anthropic Tool Use docs: [https://docs.anthropic.com/en/docs/build-with-claude/tool-use](https://docs.anthropic.com/en/docs/build-with-claude/tool-use)
- JSON Schema: [https://json-schema.org/](https://json-schema.org/)
