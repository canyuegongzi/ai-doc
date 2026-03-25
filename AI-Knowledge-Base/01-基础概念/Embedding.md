# Embedding

> 状态：第一版草案

## 1. 定义

Embedding 指的是把文本、代码、图片、文档或其他内容映射为连续向量表示的过程。

这些向量的目标，不是直接生成答案，而是让系统能够在向量空间里表达“语义相似性”“主题接近性”“结构相关性”或“跨模态对应关系”。

用一句话概括：

Embedding 的本质，是把内容转换成可比较、可检索、可计算的语义坐标。

## 2. 为什么 Embedding 是 AI 应用基础设施的一部分

在企业 AI 应用中，Embedding 通常不是面向用户直接暴露的能力，但它是很多核心能力的底层支柱。

典型场景包括：

- 语义检索
- RAG 召回
- 相似案例匹配
- 组件推荐
- 文档聚类
- 重复内容检测
- 异常检测
- 文本分类前处理
- 代码检索
- 多模态检索

对整体架构来说，LLM 负责“理解与生成”，Embedding 更偏向“表示与召回”。

## 3. Embedding 解决什么问题

### 3.1 解决语义检索问题

传统关键词检索依赖字面匹配，Embedding 让系统能够按语义相似性寻找内容。

例如：

- “插槽渲染失败”可以召回“slot 配置不生效”
- “采购审批表单”可以召回“审批类表单模板”

### 3.2 解决跨表达方式匹配问题

用户的问题表达方式与文档表达方式通常不一致。Embedding 可以缓解同义表达、描述风格差异、领域术语变体带来的匹配困难。

### 3.3 解决大规模候选集召回问题

在知识库、组件库、模板库、代码库、案例库规模较大时，不可能每次都把全部内容放入上下文。Embedding 使系统能够先快速找到少量高相关候选。

### 3.4 解决跨模态与跨语言对齐问题

现代 embedding 模型已经不只处理文本，还逐步支持：

- 多语言检索
- 文本到图片检索
- 图片到文本检索
- 文本到视觉文档检索
- 代码与自然语言检索

## 4. 它不解决什么问题

### 4.1 不直接生成答案

Embedding 负责表示与召回，不负责最终回答、总结或生成结构化结果。

### 4.2 不天然解决排序最优问题

召回出的候选通常仍然需要 rerank 或业务排序。

### 4.3 不天然解决事实正确性问题

Embedding 找到“相似内容”，不等于找到“正确答案”。

### 4.4 不天然解决权限和治理问题

企业系统里的向量召回仍然需要：

- 权限过滤
- 租户隔离
- 元数据约束
- 审计与成本治理

## 5. Embedding 的基本工作方式

一个典型流程通常是：

1. 将文档、代码、图片或结构化对象编码成向量。
2. 将用户查询也编码成向量。
3. 使用相似度函数计算查询向量与候选向量之间的接近程度。
4. 取最相近的一批结果作为候选召回。

常见相似度计算方式包括：

- Cosine similarity
- Dot product
- Euclidean distance

在很多现代 API 中，向量会做归一化处理，因此 cosine 和 dot product 在工程上经常都可使用。

## 6. Embedding 的核心能力结构

### 6.1 语义表示能力

能否把语义接近的内容映射到接近的向量空间位置。

### 6.2 区分能力

能否把细粒度但重要的差异区分开，而不是把一切“主题相关”内容都混成一团。

### 6.3 泛化能力

能否在训练外任务、陌生领域、长尾表达和跨语言场景中仍保持可用性。

### 6.4 长文本表示能力

对长文档、长代码、复杂页面配置、视觉文档等长输入是否仍能保留有效语义。

### 6.5 跨模态对齐能力

文本、图片、PDF、图表、界面截图等是否能进入统一或可对齐的向量空间。

## 7. Embedding 的常见类型

### 7.1 通用文本 Embedding

适用于大多数通用检索、分类、聚类和推荐场景。

### 7.2 检索专用 Embedding

针对 query-document 检索优化，通常会区分 query 与 document 编码方式。

### 7.3 多语言 Embedding

强调跨语言和多语种检索能力。

### 7.4 代码 Embedding

强调自然语言到代码、代码到代码的检索与匹配能力。

### 7.5 多模态 Embedding

强调文本、图片、视觉文档、图表等统一表示能力。

### 7.6 多向量 / Late Interaction Embedding

不再把整段内容压缩成单一向量，而是保留多个向量表示不同片段或 token，以提升细粒度匹配能力。

## 8. 当前行业前沿趋势

结合 2024-2025 年 embedding 领域的主流公开进展，可以看到一个很明显的变化：Embedding 已经从“RAG 里的基础配件”演化为一个高度专业化、任务化、跨模态化的独立赛道。

### 8.1 趋势一：从通用单向量走向任务化分化

过去很多 embedding 模型强调“一模通吃”，但现在主流厂商越来越强调按任务优化：

- retrieval
- text matching
- clustering
- classification
- code retrieval
- multimodal retrieval

例如 Voyage、Jina、Qwen、BGE 等近年的模型或文档都明确区分 query/document、检索/匹配/代码等不同任务模式。

这意味着企业在选型时不能只问“哪个 embedding 最强”，而要问“在哪类任务上最强”。

### 8.2 趋势二：多语言能力成为基础要求

过去英文检索是主流评测重点，但当前主流 embedding 模型越来越把多语言能力作为基础能力而不是附加能力。

例如：

- OpenAI 在 2024 年新 embedding 模型更新中强调非英语任务提升。
- BGE-M3 主打 100+ 语言、多功能与长文档能力。
- Cohere Embed 4 强调 100+ 语言检索。
- MMTEB 在 2025 年显著扩大了多语言评测覆盖面。

对企业场景而言，这意味着即使当前主要处理中文，也应优先关注模型的跨语言稳健性，因为文档、代码、接口、日志和第三方资料往往天然混合多语种。

### 8.3 趋势三：长上下文 Embedding 越来越重要

早期 embedding 模型通常只适合短文本或中等长度句段，但现在很多主流模型开始强化长输入能力。

例如：

- BGE-M3 支持到 8192 tokens。
- Voyage 4 系列文档强调 32k context。
- Cohere Embed 4 提供 128k context，并支持混合文本/图像文档。
- Jina Embeddings v4 支持视觉文档和 32k 级输入。

这意味着 embedding 已经不再只处理“句子”，而是在逐步处理：

- 长文档
- PDF
- 表格混排文档
- 技术手册
- 页面结构
- 视觉文档

### 8.4 趋势四：Matryoshka / 可裁剪维度成为实用能力

近一阶段一个非常实用的趋势是：embedding 不再只有单一固定维度，而是支持可裁剪输出维度。

OpenAI 在 2024 年 embedding 更新中强调了 `dimensions` 参数；Nomic、Voyage、Cohere、Jina 等近一代模型也都明显在强化可缩放维度和 Matryoshka 思路。

这对工程非常重要，因为它直接影响：

- 向量存储成本
- ANN 检索延迟
- 网络带宽
- 多层级索引设计
- 热冷数据分层

### 8.5 趋势五：多模态 Embedding 正在快速成为主战场

如果说 2023 年 embedding 主要还是文本赛道，那么 2024-2025 年明显的变化是：

- 文本与图片统一 embedding
- 文本与视觉文档统一 embedding
- PDF、表格、图表、截图、复杂业务文档直接向量化

Cohere Embed 4、Jina Embeddings v4、Nomic 的多模态对齐路线，以及越来越多视觉文档检索 benchmark，说明 embedding 的边界正在快速扩展。

这对企业场景尤其关键，因为企业知识并不只存在于纯文本中。

### 8.6 趋势六：多向量与 Hybrid Retrieval 思路持续增强

行业已经越来越清楚：单一 dense vector 并不总能覆盖复杂检索场景。

因此前沿 embedding 方向正在向两侧发展：

- 一侧是 dense + sparse + rerank 混合检索
- 一侧是 multi-vector / late interaction 表示

BGE-M3 同时支持 dense、sparse 和 multi-vector；Jina Embeddings v4 也支持 dense 和 late interaction 风格输出。这说明 embedding 正在从“单一向量接口”走向“更灵活的召回表示层”。

### 8.7 趋势七：Embedding 选型越来越强调“质量/成本/延迟/部署方式”联合优化

当前市场已经明显不是“单纯比榜”阶段，而是进入更工程化的竞争：

- 云 API 模型
- 开源自部署模型
- 小维度高性价比模型
- 特定领域模型
- 多模态统一模型
- 私有化部署可行性

企业实践里，embedding 模型选型越来越像数据库或检索引擎选型，而不是单纯换一个 API 名称。

## 9. 当前行业中的代表性路线

这里不做完整型号罗列，而是从路线层面梳理当前行业格局。

### 9.1 API 商业模型路线

代表厂商包括：

- OpenAI
- Cohere
- Voyage AI

特点：

- 开箱即用
- 模型更新快
- 通常具备较强通用检索能力
- 更适合快速验证和托管服务场景

### 9.2 开源通用高性能路线

代表模型与社区包括：

- BGE / FlagEmbedding
- Nomic
- Jina
- Qwen Embedding

特点：

- 可本地部署
- 可微调
- 适合企业私有化和成本敏感场景
- 社区迭代快

### 9.3 任务化与领域化路线

越来越多模型开始区分：

- 通用检索
- 代码检索
- 法律检索
- 金融检索
- 视觉文档检索

这意味着 embedding 模型已经和 rerank 一样，逐步从“通用模型选择”走向“场景适配选择”。

## 10. 企业选型时应重点看什么

Embedding 模型选型不应只看榜单名次，建议至少同时看六类因素。

### 10.1 任务匹配度

先看你的核心任务到底是：

- query-document retrieval
- semantic similarity
- clustering
- code retrieval
- multilingual retrieval
- multimodal retrieval

### 10.2 语言覆盖

企业环境往往不是纯中文或纯英文，要关注：

- 中文能力
- 英文能力
- 中英混合能力
- 多语言迁移能力

### 10.3 输入长度

如果知识源包含长文档、PDF、代码文件、大段 schema，输入长度能力会直接影响切分策略。

### 10.4 维度与成本

维度越高通常存储和检索成本越高，不一定总是最优。

### 10.5 部署方式

要明确是：

- API 托管
- 私有化部署
- 本地离线部署
- 混合架构

### 10.6 与现有检索体系兼容性

例如是否支持：

- query/document 区分编码
- 维度裁剪
- 多向量输出
- sparse/dense hybrid
- rerank 配套

## 11. 通用公开评测入口

这里不列具体模型排名，只列后续可持续跟踪的公开评测与基准入口。

### 11.1 文本 Embedding 综合评测

- MTEB Leaderboard: [https://huggingface.co/spaces/mteb/leaderboard](https://huggingface.co/spaces/mteb/leaderboard)
- MTEB 主页: [https://huggingface.co/mteb](https://huggingface.co/mteb)
- MTEB 介绍: [https://huggingface.co/blog/mteb](https://huggingface.co/blog/mteb)

### 11.2 多语言 Embedding 评测

- MMTEB Collection: [https://huggingface.co/collections/mteb/mmteb](https://huggingface.co/collections/mteb/mmteb)
- MIRACL 项目: [https://github.com/project-miracl/miracl](https://github.com/project-miracl/miracl)
- MIRACL Leaderboard 组织页: [https://huggingface.co/miracl-benchmark](https://huggingface.co/miracl-benchmark)

### 11.3 信息检索通用基准

- BEIR 项目: [https://github.com/beir-cellar/beir](https://github.com/beir-cellar/beir)
- BEIR Wiki: [https://github.com/beir-cellar/beir/wiki](https://github.com/beir-cellar/beir/wiki)
- BEIR 数据集页: [https://huggingface.co/datasets/BeIR/beir](https://huggingface.co/datasets/BeIR/beir)

### 11.4 多模态 / 视觉文档检索相关

- Jina Embeddings v4 模型页: [https://huggingface.co/jinaai/jina-embeddings-v4](https://huggingface.co/jinaai/jina-embeddings-v4)
- Jina Embeddings v4 技术报告: [https://arxiv.org/abs/2506.18902](https://arxiv.org/abs/2506.18902)
- Cohere Embed 4: [https://cohere.com/blog/embed-4](https://cohere.com/blog/embed-4)

### 11.5 厂商模型入口

- OpenAI Embeddings: [https://platform.openai.com/docs/models/text-embedding-3-large](https://platform.openai.com/docs/models/text-embedding-3-large)
- OpenAI 新 embedding 模型说明: [https://openai.com/index/new-embedding-models-and-api-updates/](https://openai.com/index/new-embedding-models-and-api-updates/)
- Voyage Embeddings 文档: [https://docs.voyageai.com/docs/embeddings](https://docs.voyageai.com/docs/embeddings)
- Cohere Embed 文档: [https://docs.cohere.com/docs/cohere-embed](https://docs.cohere.com/docs/cohere-embed)
- BGE-M3: [https://huggingface.co/BAAI/bge-m3](https://huggingface.co/BAAI/bge-m3)
- Qwen3 Embedding: [https://qwenlm.github.io/blog/qwen3-embedding/](https://qwenlm.github.io/blog/qwen3-embedding/)
- Nomic Embed: [https://huggingface.co/nomic-ai/nomic-embed-text-v1.5](https://huggingface.co/nomic-ai/nomic-embed-text-v1.5)

## 12. Embedding 与 RAG 的关系

在 RAG 体系中，Embedding 通常承担第一阶段语义召回能力。

但需要强调：

- Embedding 不等于 RAG
- Embedding 只是 Recall 层的重要能力之一
- 真正高质量 RAG 还需要 query 理解、hybrid recall、rerank、context builder 和答案生成

## 13. Embedding 与 Rerank 的关系

两者经常一起出现，但职责不同。

- Embedding 负责高召回、粗筛选
- Rerank 负责精排序、细判断

可以理解为：

Embedding 像“先拉一个候选池”，Rerank 像“再做一轮更精细的判断”。

## 14. 面向低代码平台如何理解 Embedding

在低代码平台场景中，Embedding 不应只面向“文档段落”，还应考虑对以下对象做向量化：

- 组件说明
- 组件 props 与 setter 定义
- 页面 Schema 片段
- 页面模板摘要
- 逻辑编排节点说明
- 错误案例与修复经验
- 数据源定义

这意味着低代码平台的 embedding 策略，最好是“文本 + 结构化语义 + 领域元数据”联合设计，而不是简单把所有内容原样切块后编码。

## 15. 常见误区

### 15.1 认为 embedding 模型是可以随便替换的底层件

实际上，embedding 模型会直接影响：

- 召回质量
- 切分策略
- 向量库成本
- 多语言表现
- 多模态能力

### 15.2 只看单一榜单名次

榜单重要，但不能替代场景匹配、成本、延迟和部署方式判断。

### 15.3 只做 dense retrieval

对于复杂业务检索，dense 召回往往还需要配合：

- BM25
- metadata filter
- rerank
- hybrid search
- rule-based recall

### 15.4 把 query 和 document 用完全相同方式处理

很多现代 embedding 模型已经明确区分 query 与 document 编码模式，忽略这一点会直接影响检索效果。

### 15.5 忽略 chunk 和 metadata 设计

再强的 embedding，如果输入对象切分混乱、元数据缺失、权限没处理，也很难获得好结果。

## 16. 一句话总结

Embedding 的本质，是把内容映射为可检索、可比较的语义表示；而当前行业前沿的核心趋势，已经从“做一个通用文本向量模型”演进为“多语言、长上下文、多模态、可裁剪维度、任务化分化和 hybrid/multi-vector 并行发展”的专业化基础设施赛道。

## 17. 参考资料

以下资料用于本篇第一版整理，优先使用官方文档、模型卡和原始研究入口：

- OpenAI: New embedding models and API updates, January 25, 2024: https://openai.com/index/new-embedding-models-and-api-updates/
- OpenAI model docs: text-embedding-3-large: https://platform.openai.com/docs/models/text-embedding-3-large
- MTEB 主页: https://huggingface.co/mteb
- MTEB Leaderboard: https://huggingface.co/spaces/mteb/leaderboard
- BEIR: https://github.com/beir-cellar/beir
- MIRACL: https://github.com/project-miracl/miracl
- MMTEB Collection: https://huggingface.co/collections/mteb/mmteb
- Voyage Embeddings docs: https://docs.voyageai.com/docs/embeddings
- Cohere Embed docs: https://docs.cohere.com/docs/cohere-embed
- Cohere Embed 4 announcement, April 15, 2025: https://cohere.com/blog/embed-4
- BGE-M3 model card: https://huggingface.co/BAAI/bge-m3
- BGE-M3 documentation: https://bge-model.com/bge/bge_m3.html
- Qwen3 Embedding, June 5, 2025: https://qwenlm.github.io/blog/qwen3-embedding/
- Nomic Embed Text v1.5 model card: https://huggingface.co/nomic-ai/nomic-embed-text-v1.5
- Jina Embeddings v4 model card: https://huggingface.co/jinaai/jina-embeddings-v4
- Jina Embeddings v4 technical report, June 23, 2025: https://arxiv.org/abs/2506.18902
