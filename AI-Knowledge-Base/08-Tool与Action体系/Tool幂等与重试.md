# Tool幂等与重试

> 状态：第一版草案

## 1. 文档定位

本文档用于定义 Tool Idempotency / Retry 设计，重点回答：

- 为什么 Agent 场景中的工具重试问题比传统 API 更复杂
- 哪些工具适合自动重试，哪些工具必须谨慎甚至禁止自动重试
- 幂等设计如何与 Runtime、风险分级、审计和回放协同

## 2. 为什么 Tool 的幂等与重试很重要

Agent 系统中的工具调用比传统人工操作更容易出现以下情况：

- 模型重复触发同一动作
- 网络抖动导致调用结果不确定
- Runtime 在 checkpoint 恢复后重新执行某一步
- 长任务中断后从中间恢复
- 工具返回超时，但后端实际已执行成功

如果没有幂等和重试设计，系统可能出现：

- 重复创建草稿
- 重复发送通知
- 重复写入配置
- 重复发布
- 状态不一致

用一句话概括：

Tool 幂等与重试的本质，是保证 Agent 在不稳定运行环境中仍能把“可能重复发生的动作”收敛为“可控且可解释的执行结果”。

## 3. 近两年的关键变化

### 3.1 长运行 Agent 和 durable execution 让幂等问题变得更核心

随着 LangGraph 等运行时强调 checkpoint、resume、interrupt，任务恢复已成为常态。

这意味着：

同一步工具调用可能因为恢复、重放或不确定超时而被再次触发。

### 3.2 模型循环调用让“重复调用”成为常见问题

Tool-Based Agent 里，模型可能在：

- 观察结果不充分
- 理解状态错误
- 参数轻微变化

时重复选择同一工具。

因此，幂等不仅是后端服务问题，也是 Agent Runtime 问题。

### 3.3 行业实践越来越强调“按工具类型定义 retry policy”

不是所有工具都适合同一种重试策略。更成熟的系统会按：

- tool category
- side effect
- risk level
- timeout semantics

定义不同的 retry / dedupe 行为。

## 4. 什么是幂等

幂等可以理解为：

同一个动作在相同意图和相同关键参数下被重复执行时，不会产生额外不可控副作用。

典型例子：

- 重复查询页面信息，通常天然幂等
- 重复创建草稿，如果能识别同一 request key 并返回同一草稿，也可以做到幂等
- 重复发送通知，通常若无去重设计就不是幂等

## 5. 什么是重试

重试不是简单“失败了再调一次”，而是：

在明确错误类型、风险等级和工具副作用的前提下，对可恢复失败进行受控再尝试的运行策略。

关键点在于：

- 不是所有失败都适合重试
- 不是所有工具都适合自动重试
- 重试策略必须结合幂等能力

## 6. 推荐的工具幂等分类

### 6.1 天然幂等工具

典型是只读查询类工具，例如：

- 查询配置
- 查询日志
- 查询组件协议

通常可安全重试。

### 6.2 可设计成幂等的写工具

例如：

- 创建草稿
- 提交任务
- 生成修复建议单

如果支持：

- requestId
- idempotencyKey
- dedupe window

则可以实现稳定重试。

### 6.3 弱幂等工具

即重复执行虽然不完全相同，但副作用可被约束或合并。

例如：

- 更新同一资源的同一 patch
- 对同一对象做相同状态设置

### 6.4 非幂等高风险工具

例如：

- 发送外部通知
- 删除资源
- 触发支付或外部计费动作
- 发布正式环境版本

这类工具通常不应做无条件自动重试。

## 7. 推荐的重试分类

### 7.1 可自动重试

通常适用于：

- 读取类工具
- 幂等写工具
- 明确的瞬时错误场景

### 7.2 条件重试

需满足额外条件，例如：

- 已确认后端未执行成功
- 当前处于 dry-run / test env
- 有 request key 做去重

### 7.3 禁止自动重试

通常适用于：

- 高风险非幂等工具
- 外部副作用不可逆工具
- 已经触发不确定实际落地结果的动作

### 7.4 人工确认后重试

适用于：

- 高风险但可恢复的工具
- 状态不一致需要人工裁决的场景

## 8. 推荐的重试判断维度

建议至少考虑以下维度：

### 8.1 错误类型

例如：

- timeout
- rate limit
- transient network error
- validation error
- permission denied
- business conflict

其中：

- `validation error` 和 `permission denied` 通常不应自动重试
- `timeout` 是否可重试要看工具是否幂等以及后端是否可能已执行成功

### 8.2 工具副作用类型

只读工具和执行工具的重试策略必须不同。

### 8.3 风险等级

风险越高，自动重试越保守。

### 8.4 是否支持幂等键

如果后端支持 `idempotencyKey`，运行时可以更大胆地进行受控重试。

### 8.5 当前执行状态

例如：

- 在 replay 模式下通常不应重放高风险工具
- 在恢复模式下应优先检查历史执行结果，而不是盲目重试

## 9. 推荐的幂等设计手段

### 9.1 idempotency key

为一次动作生成稳定键，例如：

- taskId + toolName + targetResource + operationHash

### 9.2 request dedupe

在时间窗口内对相同请求做去重。

### 9.3 upsert / merge 语义

让某些写操作天然更接近幂等。

### 9.4 状态检查后执行

执行前先查当前状态，确认是否已达目标状态。

### 9.5 execution receipt

工具执行后返回 receipt / operationId，供恢复和 replay 判断是否已执行成功。

## 10. Runtime 应如何处理重试

推荐的运行时策略：

1. 根据 Tool 协议读取 `idempotent`、`retryPolicy`、`riskLevel`
2. 遇到失败先判断错误类型
3. 对可能已成功落地的调用先查 execution receipt 或目标状态
4. 只有在满足策略时才重试
5. 每次重试都写入 audit / trace

换句话说：

重试不应是工具执行器内部的黑盒逻辑，而应成为 Runtime 可见、可审计、可配置的正式行为。

## 11. 与 Risk、Audit、Replay 的关系

### 11.1 与 Risk 的关系

风险越高，自动重试越应保守，甚至禁用。

### 11.2 与 Audit 的关系

每次重试都必须记录：

- retry count
- retry reason
- previous error
- idempotency key
- final outcome

### 11.3 与 Replay 的关系

回放系统通常应默认：

- 读取 recorded outputs
- mock 高风险工具结果
- 禁止真实重放非幂等高风险动作

### 11.4 与 HITL 的关系

当系统无法判断某一步是否已经真实执行成功时，应允许人工确认后再决定是否重试。

## 12. 面向低代码平台的典型建议

### 12.1 可安全重试的工具

- `searchComponentMaterial`
- `getPageInfo`
- `validatePageSchema`
- `getBuildLogSummary`

### 12.2 适合设计为幂等的工具

- `createPageDraft`
- `createDiagnosisTask`
- `generateSchemaPatch`

建议增加：

- `requestId`
- `draftKey`
- `operationHash`

### 12.3 不应盲目自动重试的工具

- `publishPage`
- `deletePage`
- `sendTenantNotification`
- `batchRepairSchema`

这些工具应结合：

- 风险分级
- 审批令牌
- execution receipt
- 人工确认

## 13. 常见误区

### 13.1 所有工具统一三次重试

这在 Agent 场景中非常粗糙，且容易造成事故。

### 13.2 把幂等问题完全留给后端系统

Runtime 若不感知幂等键和执行状态，恢复与 replay 仍然不稳定。

### 13.3 timeout 就默认重试

很多时候 timeout 只是“调用方没收到结果”，不代表服务端没执行成功。

### 13.4 高风险工具缺少 execution receipt

恢复时就很难判断要不要继续。

### 13.5 replay 真实重放高风险工具

这可能直接造成二次事故。

## 14. 一句话总结

Tool 幂等与重试的核心，是让 Agent 在长运行、可恢复、可能重复触发的环境下，仍能把工具调用控制在可预测副作用范围内；近两年的最佳实践也越来越明确，生产级 Agent 系统不能只谈工具调用成功率，更要谈重复调用时是否依然安全、可解释、可恢复。

## 15. 参考资料

- LangGraph durable execution: [https://docs.langchain.com/oss/python/langgraph/durable-execution](https://docs.langchain.com/oss/python/langgraph/durable-execution)
- Anthropic, Effective harnesses for long-running agents: [https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents)
- OpenAI, A practical guide to building agents: [https://openai.com/business/guides-and-resources/a-practical-guide-to-building-ai-agents/](https://openai.com/business/guides-and-resources/a-practical-guide-to-building-ai-agents/)
- AWS, Making retries safe with idempotent APIs: [https://aws.amazon.com/builders-library/making-retries-safe-with-idempotent-APIs/](https://aws.amazon.com/builders-library/making-retries-safe-with-idempotent-APIs/)
