# Setter体系

> 状态：第一版草案

## 1. 文档定位

本文档用于定义低代码平台中的 Setter 体系，重点回答：

- Setter 在低代码平台中到底是什么，它和 props、组件物料、页面 Schema 分别是什么关系
- 为什么 Setter 体系对 AI 生成、配置诊断和编辑辅助非常关键
- 如何把 Setter 从“编辑器控件集合”提升为可被知识化和结构化建模的领域对象

## 2. 什么是 Setter

Setter 可以理解为：

在低代码编辑器中，用于编辑某个组件属性、行为配置或绑定关系的一类配置入口与交互机制。

它不是页面运行时对象，而是编辑时对象。

一个 Setter 通常负责：

- 如何展示一个字段的配置界面
- 如何约束输入方式
- 如何输出符合 Schema 的结构
- 在什么条件下显隐或联动

用一句话概括：

Setter 的本质，是“把组件配置能力转化为编辑器可操作交互”的桥梁。

## 3. 为什么 Setter 体系是低代码平台的重要领域知识

### 3.1 低代码平台不是只靠 props schema 就能编辑

即使组件支持某个 props 字段，如果没有合理 Setter，用户也很难正确配置。

### 3.2 Setter 体系决定了配置体验与可控性

Setter 会直接影响：

- 用户能否理解配置项
- 配置项是否容易出错
- 表达式和数据绑定是否易用

### 3.3 AI 场景高度依赖 Setter 语义

AI 若要真正辅助编辑器配置，不能只知道“字段名叫啥”，还必须知道：

- 这个字段适合用什么方式配置
- 是否支持表达式
- 是否支持数据绑定
- 是否有条件显隐和联动规则

## 4. Setter 体系要解决什么问题

### 4.1 把复杂 props 转化为可编辑交互

### 4.2 为不同类型字段提供合适输入控件

### 4.3 管理字段间联动与条件显隐

### 4.4 将编辑结果稳定映射回 Schema

## 5. 推荐的 Setter 分层

建议至少分成四层：

### 5.1 基础输入 Setter

例如：

- string input
- number input
- switch
- select

### 5.2 结构型 Setter

例如：

- object setter
- array setter
- map setter

### 5.3 绑定型 Setter

例如：

- data binding setter
- expression setter
- variable selector

### 5.4 复合型 Setter

例如：

- layout setter
- style setter
- event action setter
- slot content setter

## 6. Setter 与组件物料的关系

组件物料定义“这个组件能配什么”；
Setter 体系定义“这些能力如何在编辑器里被配置”。

可以理解为：

- 物料协议是静态能力边界
- Setter 体系是编辑交互层

如果没有 Setter，物料协议难以真正转化为可编辑体验。

## 7. Setter 与页面 Schema 的关系

Setter 本身不是 Schema 字段值，但它的输出通常会落到 Schema 中。

也就是说：

- Setter 负责编辑时输入和转换
- Schema 负责运行时持久化表达

因此，一个成熟的 Setter 体系必须明确：

- 输入是什么
- 输出映射到哪个 Schema 字段
- 是否存在转换逻辑

## 8. 推荐的 Setter 定义结构

一个简化的 Setter 定义可表示为：

```ts
interface SetterDefinition {
  setterName: string;
  title: string;
  valueType: string;
  componentType: string;
  supportsBinding?: boolean;
  supportsExpression?: boolean;
  defaultValue?: unknown;
  conditions?: unknown[];
  transform?: Record<string, unknown>;
}
```

这类定义的关键价值在于：

- 可被编辑器消费
- 可被 AI 理解
- 可被配置诊断系统引用

## 9. 常见 Setter 类型举例

### 9.1 文本型 Setter

适合简单字符串、标题、说明类字段。

### 9.2 枚举型 Setter

适合 mode、size、align 等有限选项字段。

### 9.3 结构型 Setter

适合复杂对象，如表格列定义、布局配置。

### 9.4 表达式型 Setter

适合动态计算值。

### 9.5 绑定型 Setter

适合从数据源、变量、上下文中选字段。

### 9.6 事件型 Setter

适合为组件绑定交互动作和逻辑流。

## 10. AI 场景下为什么要知识化 Setter 体系

### 10.1 配置问答依赖它

很多用户不是问“这个 props 叫啥”，而是问：

- 这个字段应该怎么配
- 为什么这个 Setter 不显示
- 这个字段能不能绑定数据源

### 10.2 页面生成依赖它

AI 如果要生成“可编辑、可维护”的页面草稿，需要知道：

- 哪些字段应使用静态值
- 哪些字段更适合表达式
- 哪些字段支持绑定

### 10.3 配置诊断依赖它

很多错误其实不是 Schema 本身错，而是 Setter 输出和字段语义不匹配。

## 11. 推荐的 Setter 领域知识内容

为了让 AI 更好理解 Setter，建议知识化时至少补充：

- 适用字段类型
- 支持的输入方式
- 是否支持表达式 / 绑定
- 常见配置错误
- 与其他 Setter 的联动关系
- 输出映射示例

## 12. 面向低代码平台的典型问题类型

围绕 Setter，AI 常见要处理的问题包括：

- 为什么这个字段显示为输入框而不是下拉
- 为什么数据绑定 Setter 没有候选字段
- 为什么表达式配置后不生效
- 某个复杂对象 Setter 生成的结构是否合法
- slot / event Setter 应该怎么配置

## 13. AI 友好的 Setter 建模建议

### 13.1 明确 supportsBinding / supportsExpression

### 13.2 为复杂 Setter 提供输出示例

### 13.3 将条件显隐规则结构化

### 13.4 为 Setter 保留常见错误和最佳实践知识

这会直接提升 AI 配置辅助能力。

## 14. 常见误区

### 14.1 把 Setter 当成纯前端实现细节

这样后续很难做 AI 辅助和规则校验。

### 14.2 只有控件类型，没有输出映射语义

AI 和诊断系统都难以稳定理解。

### 14.3 不记录 Setter 是否支持表达式和绑定

生成和修复会很容易出错。

### 14.4 复杂 Setter 没有结构示例

后续配置错误率会很高。

### 14.5 Setter 与物料协议脱节

编辑器和运行时会长期不一致。

## 15. 一句话总结

Setter 体系的核心，是把组件配置能力转化为编辑器可理解、用户可操作、Schema 可落地的输入机制，并通过结构化定义把这些编辑语义沉淀为可被 AI 消费的知识对象；对低代码 AI 来说，真正懂配置，不只是懂 props，还要懂 Setter。
