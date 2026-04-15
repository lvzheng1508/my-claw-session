# RAG 搜索能力演进：从向量检索到 Agentic Retrieval

> 当 AI 开始像人一样搜索，RAG 的含义就彻底变了。

## 引言

2020 年，Lewis 等人在论文 *Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks* 中提出了一个简洁的架构：用一个 Retriever 从外部知识库检索相关文档，再由一个 Generator 基于检索结果生成回答。这就是 RAG 的起点——一个 Retriever 加一个 Generator 的双塔结构 [1]。

六年后的今天，如果你还把 RAG 等同于"向量检索 + LLM 生成"，那已经严重过时了。

2026 年 3 月，微软在其官方文档中重新定义了 RAG："RAG 不再默认等于向量搜索，而是任何形式的外部知识增强生成。" [2] 同月，学术界发布了首篇 Agentic RAG 系统综述，将其形式化为"有限视野的部分可观察马尔可夫决策过程" [3]。而 Anthropic 在其工程博客中提出了一个更具启发性的观点——"Seeing like an Agent"：搜索应该变成一种渐进式的发现（progressive disclosure），而不是一次性的检索 [4]。

这条演进线的核心脉络是什么？**搜索的主动权正在从系统转移到模型。**

本文将从三个维度展开这条脉络：

1. **为什么向量检索不够了**——单一检索模式的根本局限
2. **搜索能力的扩展**——Graph RAG、Hybrid Search、Agentic Retrieval
3. **未来走向**——混合检索是终局，模型自主搜索是范式革命

---

## 一、向量检索：好用但不够用

### 1.1 Naive RAG 的黄金时代

早期的 RAG 实现几乎千篇一律：将文档切分成 chunk，用 embedding 模型向量化，存入向量数据库，查询时计算语义相似度，取 top-k 结果拼接到 prompt 中。这套流程被称为 **Naive RAG**。

在知识密集型问答场景中，Naive RAG 确实立竿见影。Lewis 等人的实验表明，RAG 模型在三个开放式 QA 任务上达到了当时的 SOTA，生成的语言比纯参数化模型更具体、更多样、更符合事实 [1]。

但问题很快就暴露了。

### 1.2 向量检索的五重局限

**局限一：语义漂移**

向量检索基于语义相似度，但"语义相近"不等于"内容相关"。查询"Java 性能优化"可能召回"JavaScript 性能优化"的文档——两者向量距离很近，但对用户毫无价值。

**局限二：精确匹配缺失**

向量检索在处理精确匹配场景时表现差强人意：错误码（如 `ERR_CONNECTION_REFUSED`）、产品 ID、专有名词、日期等。这些内容在向量空间中缺乏足够的语义区分度。

Azure AI Search 的技术文档明确指出了这一点："某些场景——如查询产品代码、高度专业化术语、日期和人名——关键词搜索因为能识别精确匹配而表现更好。" [5]

**局限三：实体关系缺失**

向量是扁平的。它知道"张三"和"李四"各自是谁，但不知道"张三认识李四"。这种实体间的关系信息在纯向量空间中无法有效表达。

这正是微软在 2024 年提出 GraphRAG 的直接动机 [6]。

**局限四：数据同步成本高**

每次文档变更，都需要重新执行 chunk → embed → index 的全链路。对于频繁更新的知识库，这个成本不可忽视。更棘手的是，增量更新在向量数据库中并不总是可靠的——新旧 chunk 之间的语义边界可能导致检索结果的不一致。

**局限五：Chunk 边界问题**

文档切分本身就是一个信息损失的过程。一个跨越 chunk 边界的论述，其上下文会被切断，导致检索到的片段缺乏完整语义。无论用固定长度、按段落还是按语义切分，这个问题都无法完全消除。

### 1.3 社区的应对

面对这些局限，社区并没有停留在抱怨层面，而是沿着两条路径推进：

- **路径 A**：在向量检索的基础上修补——加入 re-ranking、query rewriting、multi-hop retrieval 等机制。这被称为 **Advanced RAG** [7]。
- **路径 B**：重新思考检索本身的形态——引入图数据库、关键词搜索、结构化查询，甚至让模型自己决定怎么搜。这最终导向了 **Agentic RAG**。

---

## 二、搜索能力的扩展

### 2.1 Advanced RAG：在向量检索基础上修补

Advanced RAG 本质上是对 Naive RAG 的"打补丁"。核心改进包括：

**Query Rewriting（查询改写）**：将用户的原始查询改写为更适合检索的形式。例如，将模糊的"这个 bug 怎么修"改写为具体的错误描述。

**Re-ranking（重排序）**：先用向量检索召回较多候选（如 top-50），再用一个更精确的模型（如 cross-encoder 或 LLM）对结果重新排序。Cohere Rerank 是这一方向的代表产品。

**Multi-hop Retrieval（多跳检索）**：对于需要多步推理的复杂问题，执行多次检索并综合结果。例如回答"A 公司的 CEO 的母校在哪里？"需要先找到 CEO 是谁，再查找其教育背景。

CRAG（Corrective RAG）[8] 提出了更进一步的思路：在检索后增加一个质量评估环节，如果检索结果的置信度低于阈值，就触发 web 搜索来补充。这本质上是给检索加了一个"纠错机制"。

这些改进确实提升了效果，但有一个根本问题没有解决：**检索策略是预定义的，模型无法根据具体任务动态调整。**

### 2.2 Graph RAG：补上关系推理

2024 年 4 月，微软研究院发布了 GraphRAG 论文 [6]，核心思路是：用 LLM 从文档中抽取实体和关系，构建知识图谱，再预生成社区摘要。查询时，基于社区摘要生成局部回答，最终汇总为全局回答。

GraphRAG 解决了一个传统 RAG 难以应对的问题——**全局性问答**。比如"这个数据集的主要主题是什么？"这类需要对整个语料进行归纳的问题，传统 RAG 几乎无能为力，而 GraphRAG 通过社区摘要可以给出全面且有深度的回答。

但 GraphRAG 也有明显的局限：

- **建图成本高**：需要 LLM 对每个文档进行实体抽取，API 调用成本和时间成本都不低
- **批处理模式**：不适合频繁更新的数据
- **查询延迟高**：社区摘要的生成和汇总通常需要秒级到十秒级的延迟

为了解决动态数据场景的问题，Zep 团队推出了 **Graphiti** [9]——一个开源的时序上下文图引擎。与传统知识图谱不同，Graphiti 中的每个事实都有一个**有效性窗口**（valid_at / invalid_at）：当信息变更时，旧事实不会被删除，而是被标记为失效。这使得系统可以查询"现在是什么情况"和"过去某个时间点是什么情况"。

Graphiti 还支持**增量图构建**——新数据进入后立即整合，无需全量重算。其检索方式是混合的：语义搜索 + BM25 关键词 + 图遍历，延迟通常在亚秒级。

### 2.3 Hybrid Search：混合检索成为主流

单一检索模式的局限已经得到业界广泛认可。Azure AI Search、Elasticsearch、Pinecone 等主流搜索平台都已原生支持混合检索。

Azure AI Search 的实现方式最具代表性 [5]：

- **单请求执行**：一个查询同时触发全文搜索（BM25）和向量搜索（HNSW）
- **并行执行**：两种搜索同时进行，不增加顺序延迟
- **RRF 合并**：用 Reciprocal Rank Fusion 算法合并两路结果

```json
{
  "search": "historic hotel walk to restaurants and shopping",
  "vectorQueries": [
    {
      "kind": "vector",
      "vector": [/* embedding */],
      "k": 50,
      "fields": "DescriptionVector"
    }
  ],
  "queryType": "semantic"
}
```

这种混合检索的好处是互补：向量搜索擅长语义匹配（"告诉我关于 AI 安全的内容"），关键词搜索擅长精确匹配（"查找 ERR_CONNECTION_REFUSED 的解决方案"），语义排序器（Semantic Ranker）则进一步通过机器阅读理解提升结果质量。

2026 年的企业实践中，混合检索栈已经扩展为 **Sparse（BM25）+ Dense（向量）+ Graph（图遍历）+ SQL（结构化查询）+ Grep（代码搜索）** 的组合，根据查询类型动态选择最优路径 [10]。

### 2.4 Agentic RAG：让模型自己检索

如果说 Hybrid Search 是在系统层面做"多路召回 + 智能融合"，那么 Agentic RAG 则是把检索的决策权交给了模型本身。

**A-RAG：层次化检索接口**

2026 年 2 月，A-RAG 论文 [11] 提出了一个精炼的框架：直接向模型暴露三种检索工具——

| 工具 | 用途 | 适用场景 |
|------|------|---------|
| **Keyword Search** | BM25 关键词匹配 | 精确匹配：错误码、ID、专有名词 |
| **Semantic Search** | 向量语义匹配 | 概念性查询：模糊描述、跨语言 |
| **Chunk Read** | 读取特定文档块 | 细粒度信息提取：具体段落、代码片段 |

模型不是被动接受检索结果，而是**自主决定用哪个工具、搜什么关键词、读哪些文档块**。论文实验表明，A-RAG 在多个开放式 QA benchmark 上，用更少的检索 token 超越了现有方法。

A-RAG 的关键洞察是：**现有 RAG 系统要么是单次检索拼接到输入中，要么预定义工作流让模型按步执行，两种范式都不允许模型参与检索决策。** 而模型的推理能力恰恰是检索决策中最宝贵的资源。

**SoK: Agentic RAG 系统综述**

2026 年 3 月发布的 SoK 综述 [3] 首次对 Agentic RAG 进行了系统化定义，将其形式化为**有限视野的部分可观察马尔可夫决策过程（Finite-horizon POMDP）**，并拆解为四大模块：

- **Planning（规划）**：模型如何规划检索策略——分解查询、确定检索顺序、评估何时停止
- **Retrieval Orchestration（检索编排）**：如何协调多步检索——并行 vs 串行、工具选择、结果整合
- **Memory（记忆）**：如何管理和利用历史信息——短期工作记忆、长期持久记忆、跨会话记忆
- **Tool Invocation（工具调用）**：如何调用外部工具——API 调用、数据库查询、文件系统访问

这篇综述还指出了 Agentic RAG 的系统性风险：**级联幻觉传播**（一次检索错误导致后续推理全部偏移）、**记忆投毒**（恶意注入的错误信息被模型"记住"）、**检索错位**（模型选择的检索策略与实际需求不匹配）。

**Progressive Disclosure：渐进式信息发现**

Anthropic 在 Claude Code 的实践中提出了另一个重要概念——**Progressive Disclosure** [4]。

Claude Code 最初使用传统 RAG：预索引代码库，每次响应前检索相关片段喂给模型。但这种方式有几个问题：需要索引和配置，在不同环境下容易出故障，更重要的是，Claude 被动接受上下文，而不是主动发现上下文。

Anthropic 的替代方案是给 Claude 一个 **Grep 工具**，让它像开发者一样在代码库中搜索：按文件名、按符号名、按内容模式。这不是简单的"把 grep 包装成 API"，而是让模型拥有了一种全新的检索原语。

更进一步，Anthropic 通过 **Agent Skills** 和 **Subagents** 实现了渐进式信息加载：Claude 可以读取 skill 文件，skill 文件引用其他文件，形成递归的发现链。例如 Claude Code Guide subagent 专门负责文档搜索——在自己的上下文中完成检索，只把最终答案返回给主 agent，保持主上下文干净。

这种设计的哲学是：**不预先决定模型需要什么信息，而是让模型在需要时自己去发现。** 上下文窗口是最宝贵的资源，只在需要时才加载信息，避免 context rot（上下文污染）。

**Azure Agentic Retrieval：产品级落地**

微软在 2026 年将 Agentic Retrieval 做成了产品级服务 [12]。Azure AI Search 的 Agentic Retrieval 管线工作流程如下：

1. **查询规划**：LLM 分析对话历史，将复杂查询分解为多个子查询
2. **并行执行**：所有子查询同时执行，每个子查询可以是关键词、向量或混合搜索
3. **语义重排序**：每个子查询的结果独立进行语义重排序
4. **结果合成**：合并所有结果，输出统一响应（包含 grounding data、引用来源、执行计划）

这个管线支持 Knowledge Base 对象和 MCP 端点，可以直接被其他 agent 和 chat 应用消费。微软将其定位为 Microsoft Foundry IQ 的知识层，说明 Agentic RAG 已经从概念验证进入了生产环境。

---

## 三、应用场景

### 3.1 知识库与文档问答

这是 RAG 最经典的应用场景。企业将内部文档（技术文档、产品手册、政策文件等）索引后，员工可以通过自然语言查询获取准确答案。

**Dify** [13] 是这一领域的代表性开源平台，提供了完整的 RAG 管线：文档摄入（支持 PDF、PPT、Word 等）→ 切分与索引 → 检索 → 重排序 → 生成。Dify 的特点是将 RAG 管线可视化为 Workflow，支持与 50+ Agent 工具集成。

**RAGFlow** [14] 则专注于深度文档理解，支持复杂格式文档（表格、扫描件、多栏排版）的智能切分，并提供可视化 chunking 界面供人工干预。它还融合了 Agent 能力，支持 agentic workflow 和 MCP。

### 3.2 代码辅助

代码检索是 RAG 演进中最具颠覆性的领域。传统的代码 RAG 将代码库向量化后检索，但这种方式在处理跨文件依赖、符号引用追踪等场景时力不从心。

**Grep-first Retrieval** 正在成为主流范式。Claude Code、Cursor、Codex 等工具都采用了类似策略：模型直接用 Grep/Glob 搜索符号名、追踪依赖链、读取相关文件，而不是被动接受向量检索的结果。

这种方式的优势是：
- **精确性**：符号名搜索比语义搜索更精确
- **可解释性**：每一步搜索都有明确的查询字符串
- **可控性**：开发者可以追踪模型的检索路径

Claude Code 的文档明确描述了这种范式 [15]：Claude 拥有 Read、Grep、Glob 等工具，可以自主搜索代码库、追踪依赖、构建上下文。配合 CLAUDE.md（持久化指令）和 Skills（按需加载的领域知识），形成了一套完整的代码检索工具链。

### 3.3 Agent 记忆系统

随着 AI Agent 从单轮对话走向多轮交互、从个人助手走向自主系统，**记忆**成为了一个核心问题。

**Mem0** [16] 是这一领域的代表性项目，定位为"AI Agent 的通用记忆层"。其核心架构：

- **三级记忆**：User（用户维度）、Session（会话维度）、Agent（Agent 维度）
- **动态提取**：从对话中自动提取关键信息并存储
- **混合检索**：语义搜索 + BM25 关键词 + 实体增强
- **Graph Memory 变体**：基于图记忆的增强版，能捕获实体间关系

Mem0 的论文 [17] 报告了令人印象深刻的性能数据：在 LOCOMO benchmark 上，比 OpenAI Memory 准确率高 26%，延迟低 91%，token 消耗少 90%。

Mem0 与传统 RAG 的关键区别在于：传统 RAG 是"给定一个查询，从文档库中检索"，而 Mem0 是"从持续的对话流中动态提取、整合和检索信息"。它不是在检索外部知识，而是在管理和检索 Agent 自身的经验。

**Zep / Graphiti** [9] 采用了不同的技术路径——时序知识图谱。每个事实都有有效性窗口，系统能理解"Kendra 喜欢 Adidas 鞋（截至 2026 年 3 月）"这种带时间维度的信息。这比单纯的向量检索更适合需要追踪状态变化的场景。

### 3.4 搜索增强

Perplexity、You.com 等搜索引擎是 RAG 在消费端的应用。它们将 web 搜索与 LLM 生成结合，提供带引用的搜索结果。这类应用通常采用混合检索策略：关键词搜索用于精确匹配（新闻标题、产品名），向量搜索用于语义匹配（概念性查询），再配合 re-ranking 提升结果质量。

---

## 四、开源实践盘点

| 项目 | 定位 | 检索策略 | 关键特性 |
|------|------|---------|---------|
| **Mem0** | AI 记忆层 | 语义 + BM25 + 实体 | 三级记忆、Graph Memory、+26% vs OpenAI Memory |
| **Graphiti** | 时序知识图谱 | 语义 + BM25 + 图遍历 | 事实有效性窗口、增量图构建、sub-200ms |
| **LlamaIndex** | 数据框架 | 多种索引可选 | 300+ 集成、LlamaParse 文档解析、LlamaAgents |
| **LangGraph** | Agent 编排 | 取决于实现 | 状态持久化、Human-in-the-loop、记忆管理 |
| **RAGFlow** | RAG 引擎 | 混合检索 + 重排序 | 深度文档理解、可视化 chunking、Agent 能力 |
| **Dify** | LLM 应用平台 | 可配置管线 | 可视化 Workflow、50+ Agent 工具、企业级 |
| **Khoj** | 个人 AI 助手 | 语义搜索 | 自托管、多端访问、Agent 自定义 |
| **Microsoft GraphRAG** | 图增强 RAG | 图遍历 + LLM 摘要 | 实体抽取、社区摘要、全局问答 |

---

## 五、未来：检索即推理

### 5.1 混合检索是工程层面的终局

没有单一检索模式能覆盖所有场景。向量检索擅长语义匹配但不擅长精确匹配，关键词搜索擅长精确匹配但缺乏语义理解，图遍历擅长关系推理但建图成本高，SQL 擅长结构化查询但需要预定义 schema。

未来的检索系统必然是**混合的**——不是简单地把多种检索结果拼在一起，而是让系统（或模型）根据查询类型动态选择最优的检索路径。

Azure AI Search 的 Hybrid Search 和 Agentic Retrieval [5][12] 已经在产品层面验证了这个方向。开源生态也在跟进：Mem0 的混合检索（语义 + BM25 + 实体）、Graphiti 的混合检索（语义 + BM25 + 图遍历）都是例证。

### 5.2 Agentic Retrieval 是范式层面的革命

比混合检索更深刻的变革是：**搜索的决策权从系统工程师转移到了模型本身。**

A-RAG 的三层检索接口 [11]、Claude Code 的 Progressive Disclosure [4]、Azure 的 Agentic Retrieval [12] 都指向同一个方向：模型不再是被动的"检索结果消费者"，而是主动的"检索策略制定者"。

这个变革的意义在于：

- **适应性**：模型可以根据具体任务动态调整检索策略，而不是被限制在预定义的管线中
- **可扩展性**：随着模型能力提升，检索质量自然提升，无需重新设计系统
- **简洁性**：不需要为每种查询类型设计专门的检索管线，模型自己会选

但这也带来了新的挑战：

- **级联风险**：一次检索错误可能导致后续推理全部偏移 [3]
- **成本控制**：模型自主检索可能产生大量的工具调用，需要有效的 cost-aware 编排
- **可观测性**：当模型的检索路径不可预测时，如何 debug 和审计？

### 5.3 长上下文 vs RAG：不是替代关系

随着 128K、200K 甚至更长上下文窗口的普及，"长上下文是否替代 RAG"成为了一个热门话题。答案是明确的：**不会。**

长上下文解决了"一次能看多少"的问题，但没有解决"从海量数据中找到最相关的内容"的问题。当你的知识库有百万 token 级别的数据时，把它们全部塞进上下文窗口既不经济（token 成本高），也不高效（噪声多、信号弱）。

RAG 的核心价值不是"扩展上下文窗口"，而是**精准检索**——用最小的 token 开销找到最相关的信息。这个价值不会因为上下文窗口变大而消失。

### 5.4 RAG 的未来是"检索即推理"

回顾 RAG 的演进历程：

- **Naive RAG**：检索是一个预处理步骤，与生成是分离的
- **Advanced RAG**：检索和生成之间的交互加深了（re-ranking、multi-hop）
- **Agentic RAG**：检索本身就是推理的一部分——模型在检索中思考，在思考中检索

这个趋势的终局是什么？**检索即推理（Retrieval as Reasoning）。** 搜索不再是生成前的独立步骤，而是与推理深度融合的连续过程。模型像一个经验丰富的研究者一样：先快速浏览（keyword search），发现线索后深入阅读（semantic search），找到关键段落时精读（chunk read），必要时回头查找更多信息（multi-hop）。

这不是科幻。A-RAG [11]、Claude Code [4][15]、Azure Agentic Retrieval [12] 已经在实践中验证了这个范式。

---

## 总结

RAG 六年的演进史，本质上是一部**搜索主动权转移的历史**：

| 阶段 | 谁决定搜什么 | 谁决定怎么搜 | 检索模式 |
|------|------------|------------|---------|
| Naive RAG | 系统（embedding） | 系统（top-k） | 纯向量 |
| Advanced RAG | 系统（+ re-ranking） | 系统（+ multi-hop） | 向量为主 |
| Graph RAG | 系统（实体抽取） | 系统（图遍历） | 向量 + 图 |
| Hybrid RAG | 系统（多路召回） | 系统（RRF 融合） | 向量 + 关键词 + 图 |
| Agentic RAG | **模型** | **模型** | **模型自主选择** |

从系统预定义到模型自主决策，这不是技术细节的变化，而是范式的转移。正如 Anthropic 的 Thariq Shihipar 所说："Designing the tools for your models is as much an art as it is a science. It depends heavily on the model you're using, the goal of the agent and the environment it's operating in." [4]

最好的检索系统，是能让模型"愿意用、用得好"的系统。而最好的 RAG，是让模型像人一样思考、搜索、发现的 RAG。

---

## 参考文献

[1] Lewis, P., et al. (2020). *Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks*. NeurIPS 2020. [arXiv:2005.11401](https://arxiv.org/abs/2005.11401)

[2] Microsoft Azure. (2026). *What Is RAG (Retrieval-Augmented Generation)?* [链接](https://azure.microsoft.com/en-us/resources/cloud-computing-dictionary/what-is-retrieval-augmented-generation-rag)

[3] Yadav, U., et al. (2026). *SoK: Agentic Retrieval-Augmented Generation: Taxonomy, Architectures, Evaluation, and Research Directions*. [arXiv:2603.07379](https://arxiv.org/abs/2603.07379)

[4] Shihipar, T. (2026). *Seeing like an Agent: How We Design Tools in Claude Code*. Anthropic Blog. [链接](https://claude.com/blog/seeing-like-an-agent)

[5] Microsoft Azure. (2026). *Hybrid Search Overview*. [链接](https://learn.microsoft.com/en-us/azure/search/hybrid-search-overview)

[6] Edge, D., et al. (2024). *From Local to Global: A Graph RAG Approach to Query-Focused Summarization*. [arXiv:2404.16130](https://arxiv.org/abs/2404.16130)

[7] Gao, Y., et al. (2024). *Retrieval-Augmented Generation for Large Language Models: A Survey*. [arXiv:2312.10997](https://arxiv.org/abs/2312.10997)

[8] Gu, J., et al. (2024). *Corrective Retrieval Augmented Generation*. [arXiv:2401.15884](https://arxiv.org/abs/2401.15884)

[9] Graphiti by Zep. (2025). *Build Real-Time Knowledge Graphs for AI Agents*. [GitHub](https://github.com/getzep/graphiti)

[10] Microsoft Azure. (2026). *Hybrid Retrieval Stack*. [链接](https://learn.microsoft.com/en-us/azure/search/hybrid-search-overview)

[11] Zhu, C., et al. (2026). *A-RAG: Scaling Agentic Retrieval-Augmented Generation via Hierarchical Retrieval Interfaces*. [arXiv:2602.03442](https://arxiv.org/abs/2602.03442)

[12] Microsoft Azure. (2026). *Agentic Retrieval Overview*. [链接](https://learn.microsoft.com/en-us/azure/search/agentic-retrieval-overview)

[13] Dify. (2026). *Open-Source LLM App Development Platform*. [GitHub](https://github.com/langgenius/dify)

[14] RAGFlow. (2026). *Open-Source RAG Engine*. [GitHub](https://github.com/infiniflow/ragflow)

[15] Anthropic. (2026). *Claude Code Documentation*. [链接](https://docs.anthropic.com/en/docs/claude-code)

[16] Mem0. (2026). *Universal Memory Layer for AI Agents*. [GitHub](https://github.com/mem0ai/mem0)

[17] Chhikara, P., et al. (2025). *Mem0: Building Production-Ready AI Agents with Scalable Long-Term Memory*. [arXiv:2504.19413](https://arxiv.org/abs/2504.19413)
