# Tool风险分级

> 状态：第一版草案

## 1. 文档定位

本文档用于定义 Tool Risk Tiering 设计，重点回答：

- 为什么企业 Agent 系统中的工具必须按风险分级治理
- 风险分级应基于哪些维度，而不只是“读写二分法”
- 风险分级如何与授权、HITL、审计和运行策略联动

## 2. 为什么 Tool 需要风险分级

在 Agent 系统中，工具不是中性接口。

不同工具可能带来完全不同的后果：

- 查询页面信息
- 读取日志
- 生成草稿
- 修改配置
- 发布页面
- 删除资源
- 向外部用户发送消息

这些动作的失败成本和责任边界完全不同。

如果不做分级，系统就很难决定：

- 什么工具可直接自动执行
- 什么工具需要额外授权
- 什么工具必须走 HITL
- 什么工具根本不应暴露给模型

用一句话概括：

Tool 风险分级的本质，是把不同工具可能带来的业务影响、数据影响和外部副作用显式建模，并将其转化为运行时控制策略。

## 3. 近两年的关键变化

### 3.1 行业共识已从“能不能调工具”转向“调错工具会怎样”

随着模型原生 tool use 成熟，真正的难点不再是让模型调用工具，而是防止它在错误时间、错误上下文、错误参数下调用高风险工具。

### 3.2 风险治理正在前移到 Tool Definition 和 Registry

成熟系统越来越倾向于在 Tool 协议和注册中心里就定义：

- riskLevel
- sideEffectLevel
- requiresApproval
- allowedEnv
- auditLevel

而不是把风险逻辑散落在调用代码中。

### 3.3 风险分级越来越强调参数和上下文敏感性

同一个工具在不同参数和环境下，风险等级可能不同。

例如：

- `publishPage` 到 test 可能是 medium
- `publishPage` 到 prod 可能是 high

因此，静态风险等级只是基础，动态风险评估越来越重要。

## 4. 风险分级要解决什么问题

### 4.1 自动化边界划分

决定什么动作可以自动执行，什么动作必须有人介入。

### 4.2 运行策略选择

决定超时、重试、审批、审计等运行策略。

### 4.3 工具暴露裁剪

高风险工具不应在所有场景都暴露给模型。

### 4.4 合规与责任管理

高风险动作要有更严格的留痕和追责机制。

## 5. 推荐的风险评估维度

建议至少从以下六个维度评估工具风险：

### 5.1 数据风险

该工具是否会：

- 访问敏感数据
- 跨租户读取
- 涉及 PII / 凭证 / 配置密钥

### 5.2 动作副作用风险

该工具是否会：

- 写入系统
- 覆盖配置
- 删除资源
- 触发外部通知
- 改变生产状态

### 5.3 可逆性风险

该动作是否容易回滚。

### 5.4 传播范围风险

错误执行会影响：

- 单页面
- 单应用
- 单租户
- 多租户
- 外部用户

### 5.5 环境风险

相同动作在：

- dev
- test
- staging
- prod

风险完全不同。

### 5.6 决策不确定性风险

当模型对参数、目标对象、证据链缺乏高置信度时，风险应提升。

## 6. 推荐的分级方式

企业内部可以按四级设计：

### 6.1 L0：无副作用 / 低风险读取

例如：

- 搜索知识
- 查询组件定义
- 读取公开文档

特点：

- 通常可自动执行
- 审计要求相对轻
- 允许较宽暴露

### 6.2 L1：受限读取 / 中低风险分析

例如：

- 查询租户内页面配置
- 查询构建日志
- 查询发布记录

特点：

- 需权限校验
- 通常仍可自动执行
- 应保留审计记录

### 6.3 L2：可控写入 / 中高风险变更

例如：

- 创建草稿
- 生成配置补丁
- 更新测试环境配置

特点：

- 需更强权限
- 推荐支持 dry-run
- 某些场景需 HITL

### 6.4 L3：高风险不可忽视动作

例如：

- 发布到生产
- 删除资源
- 批量修改配置
- 对外发送通知
- 跨边界数据写入

特点：

- 默认不自动执行
- 强制 HITL / approval
- 强审计和严格 replay 策略

## 7. 静态风险与动态风险

### 7.1 静态风险

由工具定义时给出基础风险等级。

### 7.2 动态风险

基于本次调用上下文提升或降低风险，例如：

- 目标环境
- 目标资源范围
- 是否批量执行
- 是否存在高风险参数
- 当前是否缺少充分证据

推荐做法是：

- Tool 协议定义基础风险
- Runtime 在调用前计算动态风险
- 两者共同决定最终策略

## 8. 风险分级如何驱动运行时策略

### 8.1 工具暴露策略

高风险工具默认不暴露给普通场景。

### 8.2 授权策略

风险越高，需要的角色和权限越严格。

### 8.3 HITL 策略

L2 某些场景进入人工确认，L3 通常默认必须 HITL。

### 8.4 重试策略

高风险写工具不应盲目自动重试。

### 8.5 审计策略

风险越高，审计粒度越细、保留周期越长。

## 9. 风险分级与 Tool Schema 的关系

风险很多时候取决于参数，而不只是工具名。

例如：

- `deleteMode=soft` 与 `deleteMode=hard`
- `targetEnvironment=test` 与 `targetEnvironment=prod`
- `notifyAudience=internal` 与 `notifyAudience=external`

因此，Schema 中应明确暴露风险关键字段，便于运行时做动态升级。

## 10. 面向低代码平台的风险分级示例

### 10.1 L0

- `searchComponentMaterial`
- `searchSchemaTemplate`
- `getSetterDoc`

### 10.2 L1

- `getPageInfo`
- `getBuildLogSummary`
- `getReleaseHistory`

### 10.3 L2

- `createPageDraft`
- `generateSchemaPatch`
- `updateTestEnvConfig`

### 10.4 L3

- `publishPage`
- `deletePage`
- `batchRepairSchema`
- `sendTenantNotification`

同一个工具若支持多环境，应根据 `targetEnvironment` 再做动态升级。

## 11. 常见误区

### 11.1 只按 read / write 二分风险

这太粗糙，无法覆盖环境、副作用、传播范围差异。

### 11.2 风险等级只写在文档里，不进入运行时

那它就无法真正驱动授权和 HITL。

### 11.3 所有写操作都一刀切禁止

这会让系统失去实用价值。更合理的是分层开放、逐步扩展。

### 11.4 高风险工具仍允许自动重试

这可能导致重复执行和扩大事故影响。

### 11.5 风险只看工具，不看参数和上下文

这会漏掉很多真实风险来源。

## 12. 一句话总结

Tool 风险分级的核心，是把不同工具及其参数在不同上下文中的潜在影响显式分层，并将其映射到工具暴露、授权、HITL、审计和重试策略上；近两年的最佳实践也越来越明确，Agent 的安全边界不是靠“少做工具”实现的，而是靠细粒度风险建模实现的。

## 13. 参考资料

- OpenAI, Building guardrails for agents: [https://openai.com/index/building-guardrails-for-agents/](https://openai.com/index/building-guardrails-for-agents/)
- OpenAI, A practical guide to building agents: [https://openai.com/business/guides-and-resources/a-practical-guide-to-building-ai-agents/](https://openai.com/business/guides-and-resources/a-practical-guide-to-building-ai-agents/)
- Anthropic, Building Effective AI Agents: [https://www.anthropic.com/engineering/building-effective-agents](https://www.anthropic.com/engineering/building-effective-agents)
- Anthropic, Writing effective tools for agents: [https://www.anthropic.com/engineering/writing-tools-for-agents](https://www.anthropic.com/engineering/writing-tools-for-agents)
