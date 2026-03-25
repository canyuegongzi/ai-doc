# Human-in-the-Loop

> 状态：第一版草案

## 1. 文档定位

本文档用于定义 Agent 系统中的 Human-in-the-Loop（HITL）设计，重点回答：

- 为什么生产级 Agent 不能只依赖自动执行
- 人工确认、人工审批、人工补充输入在运行时中应如何建模
- 近两年行业为什么越来越把 HITL 作为 Agent Runtime 的正式基础设施

## 2. 什么是 Human-in-the-Loop

Human-in-the-Loop 指的是：

在 Agent 任务执行过程中，将人类判断、确认、审批、纠偏或补充输入显式纳入执行链路的一种系统设计。

它不是“失败后人工兜底”这么简单，而是：

- 在高风险节点插入人工决策
- 在不确定节点引入人工判断
- 在信息不足节点请求人工补充
- 在关键输出节点进行人工复核

用一句话概括：

HITL 的本质，是让 Agent 的自动化边界可控，而不是让系统永远独自运行。

## 3. 为什么 HITL 是企业 Agent 的基础能力

### 3.1 企业关注的不是“完全自治”，而是“可控完成任务”

很多业务动作一旦出错，影响远大于普通问答错误，例如：

- 发布页面
- 修改配置
- 删除资源
- 发送通知
- 写入生产数据
- 跨租户读取敏感信息

这类动作如果完全自动执行，风险不可接受。

### 3.2 大模型推理再强，也不等于可以替代业务责任主体

模型可以辅助判断，但在很多场景中：

- 审批责任仍属于人
- 业务例外判断仍属于人
- 风险承担仍属于人

因此，HITL 不是技术能力不足的补丁，而是企业治理的必需边界。

### 3.3 近两年的 runtime 设计已经把 HITL 前置成正式能力

LangGraph 直接把 human-in-the-loop、interrupt、resume 作为长期运行 agent 的核心能力之一；Anthropic 和 OpenAI 的 agent 指南也都在强调高风险工具、关键动作审批和人工监督。

这说明行业共识已经比较明确：

生产级 Agent 不应假设“所有步骤都自动完成”，而应假设“自动化与人工协作共存”。

## 4. HITL 解决什么问题

### 4.1 高风险动作确认

在执行前让人确认是否继续。

### 4.2 不确定结果复核

当模型或工具输出置信度不足时，引入人工判断。

### 4.3 上下文缺失补充

当系统缺少必要业务背景时，由人提供额外信息。

### 4.4 异常分支升级处理

当任务进入复杂异常状态时，由人接管或裁决。

### 4.5 责任归属与审计闭环

让关键动作的批准、拒绝和修改都有明确责任主体。

## 5. 近两年的关键变化

### 5.1 HITL 正从审批插件演进为 Runtime 原生节点

过去很多系统把人工确认做成外部审批流或消息通知，这会导致：

- 状态不连续
- trace 断裂
- replay 困难
- 恢复路径不稳定

现在更成熟的做法是：

把人工节点作为状态机和 runtime 的正式状态，例如 `HUMAN_REVIEW`、`APPROVAL_PENDING`、`MANUAL_OVERRIDE`。

### 5.2 行业开始强调“按风险等级决定是否人工介入”

不是所有任务都要人工确认，而是根据：

- 数据敏感度
- 工具风险级别
- 是否影响生产环境
- 是否跨租户
- 是否不可逆
- 是否涉及对外通知

来动态决定是否进入 HITL。

### 5.3 HITL 不再只是“确认按钮”，而是多种人工协作模式

现代 Agent 系统中的 HITL 通常至少包含：

- confirm：确认继续
- approve / reject：审批通过或拒绝
- revise：人工修改计划或参数
- supply-context：补充缺失信息
- take-over：人工接管执行

## 6. HITL 的典型类型

### 6.1 前置审批型

在执行高风险动作前要求人工审批。

典型场景：

- 发布生产配置
- 删除页面资源
- 批量发送通知

### 6.2 结果复核型

系统先生成建议或草稿，再由人工审核确认。

典型场景：

- 页面 schema 草稿生成
- 故障分析报告生成
- 修复建议生成

### 6.3 缺信息补充型

系统无法继续执行时，要求人工补充必要上下文。

典型场景：

- 缺失业务口径
- 缺少目标应用范围
- 缺少发布环境选择

### 6.4 异常升级型

当系统识别到异常、冲突或高不确定性时，升级人工处理。

### 6.5 人工接管型

由人工直接接管后续步骤，Agent 仅保留辅助能力。

## 7. HITL 在 Agent Runtime 中应如何建模

推荐把 HITL 视为正式状态，而不是消息通知后的外部动作。

常见设计方式：

- 状态机进入 `HUMAN_REVIEW`
- runtime 持久化当前上下文与待确认事项
- 生成给人工的 review payload
- 等待人工决策
- 根据人工结果转入 `APPROVED`、`REJECTED`、`REVISE_AND_RETRY`、`ESCALATE`

建议人工节点至少记录：

- reviewType
- riskReason
- requestedAction
- currentPlanSummary
- evidenceSummary
- candidateOutputs
- approverIdentity
- decision
- decisionReason
- decisionTime

## 8. 什么时候必须进入 HITL

建议至少在以下条件下默认进入人工介入：

### 8.1 高风险写操作

包括：

- 删除
- 发布
- 覆盖更新
- 对外发送
- 生产环境写入

### 8.2 跨权限边界访问

包括：

- 跨租户
- 跨空间
- 涉及敏感字段
- 涉及管理员权限提升

### 8.3 结果置信度或证据不足

包括：

- retrieval 证据不充分
- 多工具结果冲突
- planner 置信度低
- validation 未通过但系统仍有候选方案

### 8.4 不可逆动作

如果动作无法安全回滚，通常应默认要求人工确认。

## 9. HITL 的输入输出设计

### 9.1 给人工看的信息应该足够少但足够准

不要把完整 trace 原样丢给人。更合理的是结构化呈现：

- 当前目标
- 当前状态
- 为什么需要人工判断
- 已有哪些证据
- 候选动作是什么
- 风险点在哪里
- 推荐决策是什么

### 9.2 人工返回结果应结构化

建议至少支持：

```json
{
  "decision": "approve",
  "comment": "允许在测试环境继续执行",
  "constraints": {
    "targetEnv": "test"
  }
}
```

这样 runtime 才能稳定消费，而不是只接收自然语言评论。

## 10. HITL 与 Planner、State、Guardrails 的关系

### 10.1 与 Planner 的关系

Planner 应识别：

- 当前是否需要人工节点
- 需要何种人工节点
- 人工应看到哪些关键信息

### 10.2 与状态机的关系

状态机负责：

- 进入人工状态
- 暂停执行
- 接收人工决策
- 恢复后续状态

### 10.3 与 Guardrails 的关系

Guardrails 负责定义：

- 什么风险等级必须走 HITL
- 哪类工具默认不能跳过人工确认
- 哪些情形允许人工 override

### 10.4 与 Audit 的关系

HITL 的每一次人工决策都必须被审计，否则就失去治理意义。

## 11. 面向低代码平台的典型场景

### 11.1 页面发布 Agent

在以下节点应进入 HITL：

- 发布到正式环境前
- 发现 schema 校验存在警告但非阻断时
- 涉及高影响页面覆盖时

### 11.2 配置纠错 Agent

在以下节点应进入 HITL：

- 自动修复建议涉及批量改动
- 修复方案存在多个候选路径
- 修复影响业务逻辑或数据绑定

### 11.3 运维分析 Agent

在以下节点应进入 HITL：

- 系统建议回滚版本
- 系统建议切换配置开关
- 需要人工确认问题根因归类

### 11.4 逻辑编排 Agent

在以下节点应进入 HITL：

- 自动新增关键流程节点前
- 自动修改已有流程依赖前
- 自动生成含外部通知动作的 DAG 前

## 12. 常见误区

### 12.1 认为有审批系统就等于做好 HITL

如果审批流与 Agent Runtime 脱节，状态、trace、replay 仍然是不连续的。

### 12.2 所有任务都强制人工确认

这会让系统退化成低效半自动流程，失去自动化价值。

### 12.3 给人工太多原始信息

人工 review 的目标不是读完整运行日志，而是做关键判断。

### 12.4 人工结果不结构化

这会导致 runtime 难以恢复和继续执行。

### 12.5 人工决策不纳入审计

没有审批者、原因和时间戳，后续就无法追责或复盘。

## 13. 一句话总结

Human-in-the-Loop 的核心，不是给 Agent 补一个人工确认按钮，而是把人工判断正式纳入 Agent Runtime、状态机和治理体系中，让自动化边界、责任边界和风险边界都可被显式管理；近两年的最佳实践也越来越明确，企业级 Agent 的成熟度，往往取决于它是否具备高质量的 HITL 设计。

## 14. 参考资料

- LangGraph, Human-in-the-loop: [https://docs.langchain.com/oss/python/langgraph/human-in-the-loop](https://docs.langchain.com/oss/python/langgraph/human-in-the-loop)
- LangGraph overview: [https://docs.langchain.com/oss/python/langgraph/overview](https://docs.langchain.com/oss/python/langgraph/overview)
- Anthropic, Building Effective AI Agents: [https://www.anthropic.com/engineering/building-effective-agents](https://www.anthropic.com/engineering/building-effective-agents)
- Anthropic, Effective harnesses for long-running agents: [https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents)
- OpenAI, A practical guide to building agents: [https://openai.com/business/guides-and-resources/a-practical-guide-to-building-ai-agents/](https://openai.com/business/guides-and-resources/a-practical-guide-to-building-ai-agents/)
