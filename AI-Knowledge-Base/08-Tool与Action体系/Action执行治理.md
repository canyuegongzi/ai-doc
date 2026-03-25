# Action执行治理

> 状态：第一版草案

## 1. 文档定位

本文档用于定义 Action Execution Governance，重点回答：

- 当 Tool 不再只是查询，而是能触发真实动作时，系统应如何治理
- Action 与普通 Tool 的差异是什么
- 近两年为什么行业开始把 guardrails、approval、execution control 前置到动作执行层

## 2. 什么是 Action 执行治理

在企业 AI 系统中，并不是所有 Tool 都只是读取信息。

很多 Tool 实际上会产生外部副作用，例如：

- 创建草稿
- 修改配置
- 发布页面
- 删除资源
- 发送通知
- 创建工单

这类带有真实副作用的工具，更适合被理解为 Action。

Action 执行治理指的是：

对这些带副作用动作在执行前、执行中、执行后进行授权、校验、审批、约束、审计和恢复控制的一整套机制。

用一句话概括：

Action 执行治理的本质，是把“模型建议的动作”转化为“在企业边界内可被安全执行的动作”。

## 3. 为什么必须单独治理 Action

### 3.1 查询出错与动作出错的代价完全不同

- 查询工具出错，通常影响回答质量
- 动作工具出错，可能直接影响真实业务系统

### 3.2 模型可能会在局部看似合理、整体却不安全的情况下发起动作

例如：

- 证据还不充分就建议发布
- 环境识别错误却准备修改生产配置
- 对象识别模糊却准备删除资源

因此，Action 层必须有独立的治理闸门。

### 3.3 近两年的 agent guide 都在强调 action safeguards

无论是 OpenAI 还是 Anthropic，都越来越强调：

- 高风险工具需要额外保护
- 工具执行需要清晰边界
- 不能把真实执行完全交给模型自由决定

## 4. Action 执行治理要解决什么问题

### 4.1 动作前是否允许执行

要做：

- 权限校验
- 风险判断
- 环境判断
- 参数合法性检查

### 4.2 动作过程中如何受控

要做：

- timeout 控制
- 幂等控制
- 状态跟踪
- 中断与恢复

### 4.3 动作后如何留痕和补偿

要做：

- 审计记录
- execution receipt
- 回滚或补偿策略
- replay 限制

## 5. 近两年的关键变化

### 5.1 行业正在从“tool use”走向“action governance”

过去大家更多关注模型能不能调工具；现在越来越关注：

- 工具一旦能改真实系统，必须进入动作治理体系

### 5.2 高风险动作越来越倾向于分层执行

更成熟的系统通常会把动作拆成：

- dry-run / simulate
- plan / proposal
- approval
- execute
- verify

而不是一步直接执行。

### 5.3 执行控制越来越强调 receipt、checkpoint 和 recovery

动作一旦失败，关键问题不是“再试一次”这么简单，而是：

- 有没有真的执行成功
- 是否可以回滚
- 是否要人工处理
- 后续状态如何恢复

## 6. Tool 与 Action 的关系

不是所有 Tool 都是 Action。

可以理解为：

- Tool 是更大集合
- Action 是带副作用的 Tool 子集

典型区分：

### 6.1 普通 Tool

- 搜索
- 查询
- 校验
- 检索

### 6.2 Action Tool

- 创建
- 更新
- 删除
- 发布
- 通知
- 提交任务

Action Tool 应默认进入更严格的治理路径。

## 7. 推荐的 Action 执行链路

一个较稳的动作执行链路通常包括：

1. Planner / Runtime 产出动作意图
2. Schema 校验与参数标准化
3. 权限与风险校验
4. 必要时 dry-run / simulate
5. 必要时进入 HITL / approval
6. 执行动作
7. 获取 execution receipt / result
8. 结果验证
9. 审计与后续补偿/恢复

这比“模型直接调用执行工具”要稳得多。

## 8. 推荐的治理控制点

### 8.1 参数治理

- 参数合法性校验
- 高风险字段检测
- 默认值收敛
- 环境参数约束

### 8.2 权限治理

- 角色与作用域校验
- 参数级权限判断
- 只读 / 可写边界判断

### 8.3 风险治理

- 风险分级
- 触发 HITL
- 触发审批
- 控制是否允许自动执行

### 8.4 执行治理

- timeout
- idempotency key
- retry policy
- side-effect control

### 8.5 审计治理

- 记录动作意图
- 记录审批决策
- 记录 execution receipt
- 记录最终结果与异常

## 9. 推荐的 Action 模型字段

建议 Action 对象至少具备：

- `actionType`
- `targetResource`
- `targetEnvironment`
- `riskLevel`
- `approvalRequired`
- `dryRunSupported`
- `idempotencyKey`
- `executionReceipt`
- `rollbackSupported`
- `auditLevel`

一个简化示例：

```ts
interface ActionRequest {
  actionType: string;
  targetResource: {
    resourceType: string;
    resourceId: string;
  };
  targetEnvironment?: string;
  input: Record<string, unknown>;
  riskLevel: 'L1' | 'L2' | 'L3';
  approvalRequired: boolean;
  dryRunSupported: boolean;
  idempotencyKey?: string;
}
```

## 10. 推荐的分层动作策略

### 10.1 建议型动作

系统只生成建议，不直接执行。

### 10.2 草稿型动作

系统生成草稿或待确认对象，由人最终确认。

### 10.3 低风险自动动作

仅在低风险、可逆、权限充足场景下允许自动执行。

### 10.4 高风险受控动作

必须经过 HITL、审批、强审计和严格恢复设计。

## 11. Action 执行治理与 Runtime 的关系

Runtime 不应只负责调用 Action handler，还应负责：

- 决定是否能进入动作阶段
- 决定是否需要人工节点
- 记录动作生命周期
- 在中断后恢复执行状态
- 在失败后进入补偿或升级路径

换句话说：

Action 治理不是 handler 内部的 if/else，而是 Runtime 层的正式控制逻辑。

## 12. 面向低代码平台的典型场景

### 12.1 页面发布

建议链路：

- `checkPublishRisk`
- `validatePageSchema`
- `createPublishPlan`
- `HITL approval`
- `publishPage`
- `verifyPublishResult`

### 12.2 配置修复

建议链路：

- `generateSchemaPatch`
- `dryRunPatch`
- `HITL approval`
- `applySchemaPatch`
- `verifySchema`

### 12.3 运维动作

建议链路：

- `generateRollbackPlan`
- `checkReleaseImpact`
- `HITL approval`
- `rollbackRelease`
- `verifyHealthStatus`

### 12.4 通知动作

建议链路：

- `generateNotificationDraft`
- `reviewAudienceAndContent`
- `HITL approval`
- `sendTenantNotification`

## 13. 常见误区

### 13.1 认为 Action 只是普通 Tool 的一个 handler

这会忽略副作用治理和责任边界。

### 13.2 没有 dry-run / simulate 层

高风险动作缺少中间缓冲，会显著增加事故概率。

### 13.3 认为审批只在最后一步做就够了

很多风险其实来自目标对象、环境和参数，在更前面就应被拦截。

### 13.4 执行完成后不做 verify

这会导致系统只知道“已发起动作”，却不知道“动作是否真的达到目标状态”。

### 13.5 没有补偿与恢复思路

高风险动作一旦失败，后续系统很容易陷入半完成状态。

## 14. 一句话总结

Action 执行治理的核心，是把带副作用的工具调用提升为正式的受控动作生命周期管理，并在执行前、执行中、执行后分别建立权限、风险、审批、审计、验证和恢复机制；近两年的最佳实践也越来越明确，真正可落地的 Agent 不是“会调用动作”，而是“会在边界内安全执行动作”。

## 15. 参考资料

- OpenAI, Building guardrails for agents: [https://openai.com/index/building-guardrails-for-agents/](https://openai.com/index/building-guardrails-for-agents/)
- OpenAI, A practical guide to building agents: [https://openai.com/business/guides-and-resources/a-practical-guide-to-building-ai-agents/](https://openai.com/business/guides-and-resources/a-practical-guide-to-building-ai-agents/)
- Anthropic, Building Effective AI Agents: [https://www.anthropic.com/engineering/building-effective-agents](https://www.anthropic.com/engineering/building-effective-agents)
- Anthropic, Effective harnesses for long-running agents: [https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents)
- LangGraph overview: [https://docs.langchain.com/oss/python/langgraph/overview](https://docs.langchain.com/oss/python/langgraph/overview)
