# VisionModel设计

> 状态：第一版草案

## 1. 文档定位

本文档用于定义企业 AI 平台中的 `VisionModel` 设计，回答：

- 视觉理解能力在模型接入层中应如何统一抽象
- VisionModel 与 ChatModel 的边界应该如何划分
- 在截图、PDF、图表、设计稿、页面渲染问题分析等场景中，VisionModel 应如何接入企业系统

## 2. 为什么 VisionModel 已经值得单独设计

过去很多系统把图像理解简单视为“ChatModel 附带一个 image input”。

但从当前行业演进来看，这种理解已经开始不够。

原因包括：

- 多模态模型已成为主流模型家族的重要分支
- 视觉输入类型越来越复杂，不只是单张图片
- 文本、图像、PDF、截图、图表、UI 界面往往混合出现
- 企业场景中视觉理解越来越接近正式能力，而不是演示能力

因此，VisionModel 更适合作为一个独立能力抽象，而不是完全隐没在 ChatModel 之下。

## 3. 当前行业最佳实战与趋势

### 3.1 多模态已从附加能力变成基础能力

GPT-4o、Gemini 1.5 / 2.5、Claude vision、Qwen-VL 等都已经把文本 + 图像甚至更复杂视觉文档输入纳入主流模型接口。

这说明模型层设计不应再默认“文本是主场、图像是特例”，而应把多模态当成长期主线。

### 3.2 企业场景中的视觉输入越来越“文档化”

VisionModel 不只是处理自然图片，而越来越多地处理：

- 页面截图
- 设计稿
- 图表
- PDF
- 表格截图
- 混排文档
- 视觉化报表

这意味着 VisionModel 的输入对象更接近“视觉文档理解”，而不是单纯 image captioning。

### 3.3 多模态能力与长上下文、tool use 正在结合

Gemini 1.5 / 2.5、GPT-4o 等公开资料都反映出：

- 长上下文 + 视觉理解
- 多模态输入 + reasoning
- 多模态输入 + tool use

正在成为主流组合。

因此 VisionModel 抽象不应只考虑“看图说话”，而要考虑与更大系统的协同。

## 4. VisionModel 的职责边界

### 4.1 应负责什么

- 接收视觉输入
- 支持视觉 + 文本联合输入
- 输出视觉理解结果
- 支持结构化结果
- 输出 usage 和 model metadata

### 4.2 不应负责什么

- 不直接负责 OCR pipeline 的所有前后处理
- 不直接负责图像存储
- 不直接负责业务工作流和 Tool 调度
- 不直接承担截图来源权限控制

这些应由数据层、业务层和治理层分别承担。

## 5. 推荐支持的输入类型

建议 VisionModel 抽象至少支持以下输入类型：

- image
- screenshot
- pdf page image
- chart / diagram image
- mixed visual document

并允许和文本说明混合输入，例如：

- “请根据这张截图诊断页面渲染问题”
- “请根据这份 PDF 页面和文本要求提取结构化信息”

## 6. 推荐的输出类型

建议统一支持：

- 自然语言说明
- 结构化字段抽取
- 分类结果
- 风险提示
- 视觉对象摘要

如果与结构化输出能力结合，最好支持：

- JSON Schema
- 表格抽取结果
- UI 结构摘要

## 7. VisionModel 与 ChatModel 的关系

### 7.1 为什么不完全合并

虽然许多现代模型在同一 API 中支持文本与图像，但平台抽象层仍有必要单独定义 VisionModel，因为：

- 视觉输入处理策略不同
- 场景和调用习惯不同
- 成本、延迟和模型能力差异明显
- 未来可能存在独立视觉模型和 unified multimodal model 并存的情况

### 7.2 为什么也不能完全割裂

在很多情况下，VisionModel 实际上仍然是一种 response generation model 的多模态特化能力。因此更合理的做法是：

- ChatModel 和 VisionModel 属于不同 capability
- 但可以共享一部分响应契约设计
- 由模型网关决定是否由同一底层模型提供这两类能力

## 8. 推荐的输入契约

建议 VisionRequest 至少支持：

- `inputs`: 视觉输入列表
- `textInstruction`: 文本任务说明
- `outputSchema`: 可选结构化输出约束
- `options`: 例如 max tokens、reasoning mode、streaming
- `metadata`: 来源信息与 trace 信息

## 9. 推荐的能力声明

VisionModel 建议至少声明：

- supportsImageInput
- supportsPdfLikeInput
- supportsStructuredOutput
- supportsStreaming
- supportsReasoningControls
- maxImageCount
- maxDocumentLength

## 10. 面向低代码平台的典型使用场景

### 10.1 页面截图诊断

通过页面截图辅助分析：

- 布局错位
- 组件未渲染
- 样式异常
- 可见错误提示

### 10.2 设计稿理解

用于：

- 从设计稿提取页面结构
- 辅助识别组件布局
- 生成页面草稿建议

### 10.3 图表和视觉文档理解

用于：

- 报表截图理解
- PDF 页面结构提取
- 视觉化业务说明抽取

这说明在低代码平台里，VisionModel 是潜在高价值能力，而不仅是补充功能。

## 11. 常见误区

### 11.1 认为 vision 只是 OCR

OCR 只是视觉理解的一个子问题。现代 VisionModel 还包括布局理解、对象关系理解、图表理解和上下文推理。

### 11.2 认为 vision 只是“看图说话”

在企业系统里，vision 更常见的价值是结构化抽取、界面理解和诊断辅助。

### 11.3 直接把图片塞给 ChatModel，不做能力抽象

这样短期可行，长期会让多模态能力无法治理、无法路由、无法观测。

## 12. 一句话总结

VisionModel 设计的核心，是把视觉理解能力从“聊天接口附属输入”提升为正式的平台能力抽象，让系统能够统一处理截图、PDF、图表、设计稿和视觉文档，并将其纳入模型网关、路由和治理体系中。

## 13. 参考资料

- OpenAI Vision guide: [https://platform.openai.com/docs/guides/images-vision](https://platform.openai.com/docs/guides/images-vision)
- OpenAI GPT-4o System Card: [https://cdn.openai.com/gpt-4o-system-card.pdf](https://cdn.openai.com/gpt-4o-system-card.pdf)
- Google Gemini API image understanding: [https://ai.google.dev/gemini-api/docs/image-understanding](https://ai.google.dev/gemini-api/docs/image-understanding)
- Gemini 1.5 technical report: [https://arxiv.org/abs/2403.05530](https://arxiv.org/abs/2403.05530)
- Gemini 2.5 report: [https://storage.googleapis.com/deepmind-media/gemini/gemini_v2_5_report.pdf](https://storage.googleapis.com/deepmind-media/gemini/gemini_v2_5_report.pdf)
