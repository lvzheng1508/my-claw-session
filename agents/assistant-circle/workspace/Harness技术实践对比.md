# Harness Engineering:AI 代理工程方法论与实践全景

## 引言

"Harness"并不是一个新出现的概念。早在 2023 年,AutoGPT、LangChain 等项目已经在做"类似 Harness 的事情"--只是当时没有这样命名。各大模型厂商用的词汇也各不相同:Anthropic 说 Agent / Workflow,OpenAI 说 Agents / Tools / Codex,Google 说 Agent / ADK。

但到了 2024-2025 年,随着 AI Agent 工程实践的深入和复杂化,"Harness"这个术语在工程圈逐渐被用来描述一个共同的关注点:**围绕大语言模型的控制与编排层**。

它不是一个被正式定义的标准术语,而更像是工程实践中的一个"收敛性抽象"--不同团队在解决长时间运行、复杂任务执行、可控性与可观测性问题时,逐渐走向相似的系统结构。

本文尝试用 **Harness Engineering** 来统称这些实践,并梳理各家的差异化路径。

这篇文章的核心不是"总结行业已有共识",而是**提出一个抽象框架,对齐各方的工程实践**。其中必然包含主观判断,欢迎讨论。

过去一年,Anthropic、Cursor、OpenAI 等头部玩家在 Harness Engineering 方面展开了密集的工程探索。他们的实践各有侧重,底层模式正在趋同,但细节差异仍然显著。

---

## Harness:从软件工程的"老古董"到 AI 时代的新生命

在进入正文之前,值得先追溯一下"Harness"这个词的来龙去脉。它不是 AI 时代的发明,而是一个在软件工程中存在了几十年的"老古董"。

### 从工程学到软件工程

在传统工程学中,Harness 的原始含义是"马具"或"电线束"。想象一下:为了让一台复杂的发动机在实验室里运转,你需要一堆电线和支架把它固定住,给它供电,连接仪表来读取数据。这套固定的装置就是 Harness。

到了软件工程,这个概念演化为 **Test Harness(测试治具)**:为了测试某个组件而搭建的一套"脚手架"代码。它通常包括:

- **驱动程序(Driver)**:负责发送输入,触发被测代码
- **桩程序(Stub)**:模拟外部依赖,隔离被测组件
- **监控工具(Monitor)**:收集输出,记录结果

核心逻辑很直观:你的代码是"马",Harness 是套在马身上的"具",用来控制和测量马跑得快不快。

几十年来,Test Harness 是软件工程的基础设施--JUnit、PyTest 等测试框架本质上都是某种形式的 Test Harness。它解决的核心问题是:**如何可靠地、可重复地、可观测地运行一个黑盒组件。**

### 为什么 AI 领域重新关注 Harness?

AI 模型(尤其是 LLM)和传统的函数有一个根本区别:**随机性和黑盒属性**。你不能像调用 `sort([3,1,2])` 那样确定性地验证一个 LLM 的输出。同一个 prompt,模型可能给出不同的回答;同一个任务,多次运行的结果可能差异巨大。

这意味着传统的 Test Harness 不够用了。你需要一套更复杂的脚手架来:

- **控制**:给模型合适的输入,管理上下文状态
- **测量**:评估输出质量,而不只是"跑过没报错"
- **协调**:组织多个模型实例协作完成复杂任务
- **持久化**:跨会话保持状态,支持长时间运行

于是,**Evaluation Harness(评估框架)** 应运而生。一个标志性项目是 EleutherAI 的 [lm-evaluation-harness](https://github.com/EleutherAI/lm-evaluation-harness)--著名的 Hugging Face Open LLM Leaderboard(开源大模型排行榜,V1 版本)后台就是基于这个框架运行的。

从 Test Harness 到 Evaluation Harness,Harness 的含义在 AI 语境下发生了扩展:

| 阶段         | Harness 的职责                               | 控制对象             |
| ------------ | -------------------------------------------- | -------------------- |
| **传统软件** | 驱动、桩、监控                               | 确定性函数/组件      |
| **AI 评估**  | 批量推理、结果采集、指标计算                 | LLM 的输出质量       |
| **AI Agent** | 上下文管理、多代理协调、状态持久化、安全控制 | LLM 的行为和决策过程 |

第三行,就是本文所讨论的 **Harness Engineering** 的范畴。

所以,Harness Engineering 不是凭空出现的概念。它继承了软件工程几十年积累的思想--**如何可靠地、可重复地、可观测地运行一个复杂组件**--只是控制对象从确定性函数变成了一个具有随机性和自主性的 AI Agent。

理解了这个历史脉络,就能明白为什么 Harness Engineering 的关注点--安全、可观测性、状态管理、标准化--和传统软件工程中的 Test Harness 是一脉相承的。只是 AI 时代的"马"变得更聪明、更不可预测,"具"也需要更复杂。

---

## 一、核心挑战:为什么需要 Harness Engineering

早期的 AI 应用模式很简单:用户问一个问题,模型给一个答案。这是 Q&A(问答)模式,上下文仅限当前会话。

但很快,人们开始尝试更复杂的任务:

- "帮我重构这个模块" -- 需要读取多个文件、理解代码结构、做增量修改、测试、提交
- "写一个完整的博客系统" -- 需要规划、分步实现、跨会话持续工作
- "监控我的项目,发现 bug 自动修复" -- 需要长期运行、记忆状态、主动触发

这些任务有两个共同特征,使它们超出了传统 Q&A 模式的能力边界:

1. **任务复杂度高**:涉及大量文件的读写、多步推理、跨模块的修改,一次完整的任务可能需要数百次 tool call
2. **会话跨度长**:任务可能跨越多个会话--一个会话因上下文耗尽或中断而结束,下一个会话需要继续未完成的工作

需要注意的是,传统 Q&A 模式也可以是多轮对话(几十甚至上百轮),但它的上下文始终在一个连续的会话内,且任务相对封闭。而上面这些任务的特点是**复杂度高 + 会话跨度长**的组合--不是"轮数多"的问题,而是"单次会话内做不完"且"每次恢复都需要大量上下文"的问题。

这正是 Harness Engineering 所关注的问题域。

Anthropic 在多篇文章中深入分析了长时间运行代理面临的挑战。这些分析为 Harness Engineering 提供了重要的理论参照--本文将其提炼为三个核心设计约束。这三个挑战构成了 Harness Engineering 的核心设计约束,也是后续各家实践的出发点。

### 1.1 上下文腐烂(Context Rot)

LLM 有一个"注意力预算"--上下文窗口越大,准确召回信息的效率越低。token 越多,模型"看"不清前面的内容。

这不是上下文窗口不够大,而是模型本身的信息处理机制限制了它在超长上下文中的检索能力。Anthropic 将这种现象命名为 **Context Rot(上下文腐烂)**。

更直白地说:不是"窗口装不下",而是"装得下但看不清"。

这带来了一个工程问题:**如何在有限的注意力预算内,让模型看到最重要的信息?** 这个问题贯穿了所有 Harness Engineering 实践,从简单的 prompt 压缩到训练级别的 Self-Summarization(在模型训练阶段就把"如何有效压缩上下文"作为优化目标,而非仅在推理时通过 prompt 引导压缩),都是在回答这个问题。

### 1.2 记忆断裂(Context Reset)

每个新会话是全新的,模型不记得之前的任何东西。但长时间任务需要跨会话的持续状态。

**结构化工件(Structured Artifacts,即记录工作状态的文件)** 是应对记忆断裂的一种典型方案,Anthropic 在其工程文章中对此有详细描述:把关键状态写成文件(如 `claude-progress.txt`),作为人工可读的"交接文档",新会话读取它快速理解工作进展。这本质上是一种"面向机器的文档化"--不是让模型"记住",而是让状态"可被读取"。

这带来了第二个工程问题:**如何在新会话中快速恢复工作状态?** 不同厂商给出了不同答案:有的用文件 artifact,有的用自动摘要,有的用后台记忆系统。

### 1.3 自我评估偏差(Self-Evaluation Bias)

让模型自己评估自己写的代码,天然存在偏差--它会倾向于认为"我写的没问题"。学术界通常用 Sycophancy(阿谀顺从)或 Self-Reinforcement Bias 来描述这一现象,其成因不仅包括模型本身的倾向,也包括 RLHF 训练过程中对自身输出产生的高置信度。

一种被广泛采纳的解决之道是 **生成器-评估器分离(Generator-Evaluator Separation)**:两个独立的模型实例,一个负责生成(干活),一个负责验收。这个设计受 GAN(Generative Adversarial Network,生成对抗网络)的启发1--通过生成与评估的对抗关系来提升输出质量。Anthropic 在《Harness Design for Long-Running Apps》中对此有系统性阐述,并将其发展为 Planner-Generator-Evaluator 三代理架构。

这带来了第三个工程问题:**如何可靠地验证模型的工作质量?** 这个问题催生了多代理协调、硬层级架构、可观测性等一系列工程实践。

---

**这三个挑战--上下文腐烂、记忆断裂、自我评估偏差--是 Harness Engineering 的"问题三角"。** 后续所有厂商的实践,都可以理解为对这个三角不同角度的回应。

### Harness Engineering ≠ Workflow Orchestrator

一个容易混淆的问题是:Harness Engineering 和传统的 Workflow Orchestrator(如 Airflow、Temporal)有什么区别?

两者都涉及"编排",但本质不同:

- **Workflow Orchestrator** 编排的是**确定性流程**--步骤、分支、重试策略都是预定义的,执行过程可预测
- **Harness Engineering** 管控的是**非确定性决策**--LLM 的输出不可预测,Harness 的职责是在不确定性中建立可控性

更根本地说,Harness Engineering 不是流程驱动,而是**推理驱动**。整个系统的驱动力是模型的推理能力--每一次 tool call、每一个决策、每一步任务推进,都是模型在推理之后做出的选择。Harness Engineering 不指挥模型做什么,它做的是确保 Agent Loop 不偏离目标:

- **上下文管理**解决的不是"怎么编排流程",而是确保模型在做决策时能"看到"它该看到的信息--不因 Context Rot 而"看不清"
- **状态持久化**解决的不是"怎么串接步骤",而是确保模型在新会话中能"想起来"之前做了什么--不因 Context Reset 而"失忆"
- **质量验证**解决的不是"怎么执行计划",而是确保模型的输出"经得起检验"--不因 Self-Evaluation Bias 而"自我感觉良好"

打个比方:模型是驾驶员,Harness Engineering 是安全带、仪表盘和导航系统。不是安全带在开车,而是安全带让驾驶员能安心开得更快、更远。

这也意味着 Harness Engineering 的设计原则不是"用工程替代模型",而是"用工程释放模型"--让模型的推理能力在复杂、长时间的任务中得到最大程度的发挥。

更准确地说,Harness 是模型推理运行的**工程支撑层**,不是驱动执行的主体,而是让模型能可靠运行的基础设施:保证模型看到的上下文是清晰的、用到的工具是可靠的、产出的结果是可验证的。执行什么、怎么执行,始终是模型的推理在决定;Harness 负责的是--无论模型决定做什么,这件事都能可靠地发生。

这也是为什么 Harness Engineering 的关注点--上下文腐烂、记忆断裂、自我评估偏差--在传统编排系统中不存在。传统系统不需要担心"编排器看不清前面做了什么",但 LLM 需要。

---

## 二、Anthropic:模型驱动的 Harness Engineering

### 2.1 理论框架:Workflows vs Agents

Anthropic 围绕 Harness Engineering 主题发布了一系列工程文章,形成了清晰的时间线:

| 时间         | 文章                                        | 核心主题                                                                        |
| ------------ | ------------------------------------------- | ------------------------------------------------------------------------------- |
| 2024 年底    | Building Effective Agents                   | 基础框架:Workflow vs Agent 分类,渐进式复杂度                                    |
| 2025 年      | Effective Context Engineering for AI Agents | 上下文工程:Context Rot、注意力预算                                              |
| 2025 年      | Effective Harnesses for Long-Running Agents | 长时间运行:Initializer Agent + Coding Agent,结构化工件                          |
| 2026 年 3 月 | Harness Design for Long-Running Apps        | 三代理架构(Planner-Generator-Evaluator),Sprint 契约,Context Reset vs Compaction |

在《Building Effective Agents》中,Anthropic 给出了一个清晰的分类框架,将 Agentic Systems 分为两类:

| 类型                  | 特征                 | 适用场景                             |
| --------------------- | -------------------- | ------------------------------------ |
| **Workflows(工作流)** | 预定义的代码路径编排 | 任务流程清晰、步骤固定、可控性要求高 |
| **Agents(代理)**      | LLM 动态自主决策     | 任务不确定、需要探索、灵活性优先     |

**核心原则**:找到尽可能简单的解决方案,只在需要时才增加复杂性。

这是一个渐进式模型:

```
增广 LLM(Augmented LLM)        → 模型 + 工具,单次交互
    ↓
组合式工作流(Composable Workflows) → 多步编排,预定义流程
    ↓
自主代理(Autonomous Agents)        → 模型自主决策,自主调用工具
```

大多数任务其实只需要第一步或第二步。只有当任务超出模型的独立能力边界时,才需要第三步--真正的自主代理。

Anthropic 没有使用"Harness"这个词,但他们所描述的这些系统--控制流编排、工具调度、上下文管理--正是 Harness Engineering 的核心关注点。

### 2.2 三代理架构:Planner → Generator → Evaluator

在《Harness Design for Long-Running Apps》中(注意,Anthropic 在这篇文章标题中直接使用了"Harness"一词),Anthropic 提出了一个受 GAN 启发的系统,直接回应了"问题三角":

- **Planner(规划者)**:理解用户目标,分解任务,创建 Sprint 契约
- **Generator(生成器)**:执行具体编码工作,留下结构化工件
- **Evaluator(评估器)**:验收生成器的工作,给出反馈

**对三个挑战的回应**:

| 挑战             | Anthropic 的解法                                                                |
| ---------------- | ------------------------------------------------------------------------------- |
| **上下文腐烂**   | Sprint 结构限制每次的工作范围,避免上下文过度膨胀;结构化工件让关键信息可快速检索 |
| **记忆断裂**     | Generator 留下 `progress.txt` 等 artifact,新会话读取快速恢复状态                |
| **自我评估偏差** | 生成器-评估器分离,两个独立实例各司其职                                          |

**关键设计**:

- **Sprint 契约机制**:每个 Sprint(冲刺,即一段限定范围的工作周期)开始前协商"完成"标准--明确这个周期做什么、做到什么程度算完成,避免评估时的模糊标准
- **增量推进**:Generator 不追求一次性完成,而是留下结构化 artifact(工件)
- **核心洞见**:Harness Engineering 的有趣组合空间不随模型改进而缩小,只是移动

### 2.3 Opus 4.6 的冲击

Anthropic 在其工程文章中描述了一个有趣的工程决策演进:

在 Sonnet 4.5 时代,Claude 表现出强烈的"上下文焦虑"(context anxiety)--这是一个**模型行为**,而非工程问题。当模型"感知到"自己快接近上下文窗口时,会自发地开始提前收尾,反复检查之前的工作,消耗大量 token 在"回头看"而非"向前推进"上。这不是因为外部信号触发,而是模型在训练数据中学到的隐式行为模式。

Anthropic 最终不得不采用 **Context Reset(上下文重置)** 策略来应对:彻底清空上下文,通过结构化 artifact 进行交接,让新 Agent 从干净状态重新开始。

这是一个关键的工程洞察:**Compaction(压缩继续)和 Reset(重置重来)是应对 Context Rot 的两种不同机制**,它们的效果截然不同:

- **Compaction**:同一个 Agent,将旧对话压缩成摘要后继续运行。Agent 的上下文缩短了,但它仍然"知道自己跑了很久"--摘要里有痕迹,上下文焦虑可能持续。压缩了上下文,但焦虑行为依然存在
- **Reset**:旧 Agent 终止,写一份结构化交接文档(如 `progress.txt`);新 Agent 启动,读取交接文档,从干净状态开始。新 Agent 没有"我跑了很久"的记忆,焦虑消失

代价是:Reset 需要设计一个足够好的结构化交接文档,让新 Agent 能无缝接手,否则信息会丢失。这就是为什么"Structured Artifacts"在 Anthropic 的工程设计中如此重要。

到了 Opus 4.6,Anthropic 在**模型训练/能力层面**改善了这个问题--模型不再有提前收尾的倾向("largely removed that behavior")。这意味着可以在单个长会话内持续工作,不再需要频繁的 Context Reset,Sprint(限定范围的工作周期)结构也因此可以简化甚至移除。

**Sprint 的工作机制值得补充说明**:Sprint 拆分本身是通过提示词实现的--系统提示定义了 Planner 的角色和工作流程,要求它将大任务分解为小工作周期。Planner 之所以能合理地拆分任务,依赖的是模型本身的通用推理和规划能力,而非专门为"任务拆分"训练过的。完整的链条是:**通用能力(模型自带) + 提示词(工程侧定义角色和流程) + Sprint 契约(工程侧要求明确完成标准)**,三者缺一不可。

这也解释了为什么 Opus 4.6 之后 Sprint 结构可以简化--提示词没有变,但**模型本身的规划能力提升了**,不再需要那么细的拆分就能处理好大任务。这再次印证了 Anthropic 的工程哲学:当模型有局限时,用工程补(细粒度 Sprint + Reset + Structured Artifacts);当模型进步时,第一反应是把工程简化掉。

但这并不意味着"不需要 Harness Engineering",而是说它**对三个挑战的回应方式在变化**:

| 挑战             | Sonnet 4.5 时代                             | Opus 4.6 之后                                                    |
| ---------------- | ------------------------------------------- | ---------------------------------------------------------------- |
| **上下文腐烂**   | Compaction 不够,必须 Context Reset          | 模型自身行为改善,可在单会话内持续工作,Sprint(工作周期)粒度可变粗 |
| **记忆断裂**     | 结构化 artifact 是刚需(每次 reset 都要交接) | 仍然需要,但频率降低,单会话内可自我维持                           |
| **自我评估偏差** | Evaluator 是必需品                          | 简单任务可能不需要,复杂任务仍需要                                |

这个演进过程很好地说明了前文提到的观点:**Harness Engineering 的有趣组合空间不随模型改进而缩小,只是移动。** 不是"不需要 Harness 了",而是"需要的 Harness 形态变了"--从 Reset-heavy 转向 Compact-friendly。

### 2.4 Claude Code:Anthropic 的工程实践落地

Claude Code 是 Anthropic 官方发布的终端代理工具,其源码在 2026 年 3 月通过 npm 的 .map 文件泄露,让我们得以窥见 Anthropic 如何将理论框架落地为工程实现。

Claude Code 的实现可以看作 Anthropic Harness Engineering 思想的"编码领域特化版"。

#### 回应上下文腐烂:多层次的压缩策略

Claude Code 实现了非常复杂的上下文管理系统:

1. **AutoCompact**:自动压缩历史,当接近上下文窗口时触发
2. **MicroCompact**:对工具结果进行微缩(比如 Bash 输出的详细内容)
3. **SessionMemory**:后台维护一个 markdown 文件,自动提取关键信息
4. **Compaction**:主压缩逻辑,使用 LLM 生成摘要

**关键细节**:

- 压缩不是简单的"总结前面说了什么",而是**结构化的状态提取**--保留活跃任务、决策、TODOs、承诺
- **MicroCompact** 针对工具结果:Bash 输出、文件内容、搜索结果等都可以被压缩为 `[Old tool result content cleared]`
- **SessionMemory 是后台运行的**:通过 fork 子代理提取信息,不阻塞主对话

这几乎是 prompt-level 压缩的"极致工程化"--四层系统各司其职,覆盖了从宏观摘要到微观工具结果的完整压缩链路。

#### 回应记忆断裂:Agent Memory + 结构化工件

Claude Code 实现了多层次的记忆系统:

- **Agent Memory**:
  - **user** 级别:`~/.claude/agent-memory/`
  - **project** 级别:`<cwd>/.claude/agent-memory/`
  - **local** 级别:`<cwd>/.claude/agent-memory-local/`
- **结构化工件**:AGENTS.md 项目配置、scratchpad 目录等

不同类型的 Agent 可以有独立的记忆空间,支持跨会话的知识积累。这比 Anthropic 文章中描述的 `claude-progress.txt` 更系统化。

#### 回应自我评估偏差:Coordinator Mode

Claude Code 实现了一个 Coordinator 模式。要理解这个模式,需要先理解它的三层架构:

**代码层(硬约束)**:Coordinator 拥有工具定义--如 `Task` 工具(spawn 子 Agent)、`SendMessage`(向已有 Agent 发消息)、`TaskStop`(终止 Agent)。这些是代码层面的能力,模型必须通过调用工具才能实际创建和管理 Worker。没有工具,光靠提示词无法派任务。

**提示词层(软约束)**:系统提示定义了 Coordinator 的角色和工作流程--"你是 Coordinator,你应该先 Research,再 Synthesis,再 Implementation,再 Verification"等。系统提示不是 if-else 语句,它是对模型行为的引导。模型读到它,倾向于按照它行事,但在特定情况下也可以偏离。

**模型推理层(动态)**:具体决策由模型推理完成--查哪些文件、需要几个 Worker、这个任务 spawn 还是自己做。模型的"决定"通过 tool call 具体化:当模型输出一个 `Task` 调用,决策就被执行。Harness 负责把 tool call 变成实际运行的子 Agent 进程,模型不需要知道背后的实现细节。

Claude Code 的系统提示中明确写道（源码见泄露仓库中的 `src/coordinator/coordinatorMode.ts` 和 `src/constants/prompts.ts`，后者负责主系统提示的组装）:

> "You are a **coordinator**. Your job is to: Help the user achieve their goal, Direct workers to research, implement and verify code changes, Synthesize results and communicate with the user"

Coordinator 的职责:

1. **并行研究**:启动多个 Worker 同时探索代码库不同角度
2. **合成发现**:Coordinator 必须理解 Worker 的发现,然后给出具体实现 spec
3. **实现委托**:用清晰的 prompt 委托给 Worker
4. **验证分离**:验证工作应该由 fresh Worker 完成,而非实现 Worker 自验证(Anthropic 的原始设计是两轮评估:Generator 先自我评估,Evaluator 再独立验收--Claude Code 将这一模式硬编码为"实现 Worker 不自验,由 fresh Worker 独立验证")

**关键设计**:

- Coordinator 不把"理解"委托给 Worker -- 它必须自己合成发现
- Workers 不互相通信,只和 Coordinator 通信
- Continue vs Spawn 的选择取决于**上下文重叠度**:高重叠用 continue,低重叠用 fresh spawn

这是 Anthropic 三代理架构(Planner-Generator-Evaluator)的编码领域特化实现--将 Planner 和 Evaluator 合并为一个 Coordinator,Generator 对应 Worker。需要指出的是,Claude Code 的 Coordinator 模式和 Anthropic 文章中的三代理架构是"相关但不同的实现":三代理是研究性质的工程实验,Coordinator 是受其启发的生产级简化版本。

Coordinator 的系统提示非常长且详细,包含了任务流程(Research(研究) → Synthesis(综合) → Implementation(实现) → Verification(验证))、并发管理原则、写 prompt 的最佳实践、处理 Worker 失败的策略等。这是一个**硬编码的经验总结**--不是让模型"自己学会如何协调",而是直接告诉它"这样最好"。它凝结了大规模实践中积累的教训。提示词写得如此详细,正是因为它是唯一的"约束手段"--越详细、越具体,模型偏离的概率就越低。

这其实就是 Anthropic 整个设计哲学的缩影:**模型只管推理和决策,Harness 只管执行和编排,两者通过工具定义这个接口连接。** 这也是为什么模型越强,Coordinator 就越可靠--不是因为代码变了,而是因为遵循提示词的能力和任务规划的能力都在提升。

#### 安全:细粒度的权限控制

Claude Code 的 Bash 工具有非常复杂的权限检查:

- 分段式命令检查:多 cd 命令需要审批
- cd + git 检查:防止 bare repo fsmonitor 绕过
- 管道段检查:防止跨段的安全问题

这是一个**安全优先**的实现--宁愿多问几次,也不冒安全风险。

### 2.5 Anthropic 的工程哲学

从理论到实践,可以总结出 Anthropic 的核心哲学:

1. **简单优先**:能用 Workflow 就别用 Agent,能单模型就别用多模型
2. **渐进式复杂度**:从简单方案开始,只在必要时增加复杂性
3. **结构化工件是关键**:跨会话状态不是"记忆",而是"可读的文件"
4. **分离关注点**:生成和评估分离、规划和执行分离
5. **侧重模型**:不是不做工程,而是工程哲学是"最小化工程干预,让模型发挥"。Anthropic 做了很多精细的工程(四层压缩、三代理架构),但出发点是"模型当前有局限,用工程补",而非"不信任模型"。一旦模型进步(Opus 4.6),第一反应是简化工程--这和 Cursor"用工程结构兜底"的思路形成鲜明对比

**"简单优先"的真正含义**

这五条原则中,"简单优先"是最容易被表面理解的。字面意思是"能简单就简单",但更准确的理解是:**只在被迫时才增加复杂性,且能说清楚为什么。**

Claude Code 本身就是一个有趣的注脚。它拥有四层压缩系统、Coordinator 模式、三级 Agent Memory、Bash 分段权限检查……从结果看相当复杂。但这些复杂性不是预先设计的顶层架构,而是逐层被具体问题逼出来的--先尝试简单方案,发现不够,再加一层;再不够,再加一层。每一层复杂性背后都有一个明确的问题驱动。这正是"简单优先"的真正含义:不是追求永远简单,而是确保每一层复杂性都有存在的理由。Anthropic 的思路更接近"大多数人的问题根本不需要这个级别的架构,你先用单 Agent 试试?"

---

## 三、Cursor:工程驱动的 Harness Engineering

Cursor 的实践是另一个极端:他们运行着真正的超大规模代理系统,35% 的 PR 由自主云 Agent 创建,单周可构建 100 万行代码。

如果说 Anthropic 侧重模型,Cursor 则在架构层侧重工程--Planner-Worker-Judge 硬层级、云 VM 持久化、可观测性基础设施,系统的可靠性高度依赖工程架构。但 Cursor 同时也在模型层发力:自研 Composer 模型并做了 Self-Summarization 等训练级优化,用模型能力来回应上下文腐烂等核心挑战。两条线并行不矛盾--工程架构确保大规模运行的可靠性,自研模型提供 prompt-level 工程无法达到的压缩效率。

### 3.1 从单代理到 Planner-Worker-Judge

Cursor 的演进非常清晰地展示了从"自协调"到"层级化协调"的转变,这个过程直接回应了自我评估偏差和上下文腐烂两个挑战。

#### **阶段 1:单代理**

- 一开始,他们尝试让一个代理完成所有工作
- **失败**:上下文爆炸(Context Rot)、任务堆积、无法并行

#### **阶段 2:自协调多代理**

- 多个代理互相通信、协商、分工
- **失败案例**:
  - 锁竞争:多个代理争抢同一个文件
  - 风险规避:没人愿意负责"端到端实现"
  - 无人总负责:最后谁来兜底?(自我评估偏差的系统性表现)

#### **阶段 3:Planner + Workers + Judge**

- **Planner**:持续探索代码库,创建任务(可递归子规划)
- **Workers**:只专注完成分配的任务,不互相通信
- **Judge**:每轮末尾决定是否继续

这是一个**硬层级**系统:Planner 居中调度,Workers 是无状态的执行单元,Judge 是质量控制点。

**对三个挑战的回应**:

| 挑战             | Cursor 的解法                                                              |
| ---------------- | -------------------------------------------------------------------------- |
| **上下文腐烂**   | Planner 递归子规划,把大任务分解为小上下文;Worker 无状态,每次只看自己那部分 |
| **记忆断裂**     | 云 VM 持久化环境 + 结构化日志;任务状态由 Planner 管理                      |
| **自我评估偏差** | Judge 独立于 Workers,每轮末尾做全局判断                                    |

**Worker 设计的殊途同归**:值得注意的是,Claude Code 的 Coordinator-Worker 模式和 Cursor 的 Planner-Worker 模式,虽然出发点不同(前者从三代理理论简化而来,后者从自协调失败的经验收敛而来),但最终在 Worker 设计上高度同构:上下文隔离、只与中枢节点通信、无状态执行。这说明在多代理系统中,"切断 Worker 间通信"已经是一个被广泛验证的工程最佳实践--不是让 Worker 扮演特定角色,而是控制它的上下文视野。

### 3.2 从工程实践到自主模型

前文分别介绍了 Anthropic/Claude Code 和 Cursor 的实践。两者都瞄准编码场景,但走了截然不同的路线。本节从三个维度做直接对比,可以清晰地看到 Cursor 从工程工具到自主模型的进化路径。

#### 工程级实践:代码理解与上下文压缩

**代码理解方式:基础设施 vs 压缩工程**

Claude Code 和 Cursor 在"让 Agent 理解代码库"这个问题上走了完全不同的路径:

- **Cursor**:代码库提前向量化索引(云端存储),Agent 需要理解代码时通过语义搜索取回相关片段放入 context。本质上是**用基础设施换 context**--索引预计算减少了进入 context 的内容,压缩压力小
- **Claude Code**:没有预索引,Agent 直接用 shell 工具读原始文件(cat、grep、find),读到的内容直接进入 context。本质上是**用压缩工程换 context**--不预计算,但用极致的压缩保证模型能持续看到它该看到的东西

这个差异解释了为什么 Claude Code 要做那么极限的压缩系统:因为它没有向量索引这层"预过滤",所有代码理解都走 context window,压缩是核心矛盾。Cursor 有索引,Agent 不需要把整个代码库塞进 context,压缩的压力小很多。

这不是谁更聪明,而是两种工程世界观的自然延伸:Cursor 不信任模型独立处理超大代码库,用基础设施兜底;Claude Code 信任模型的代码理解能力,用工程保证模型能持续看到它该看到的东西。

向量索引方案也存在时效性问题:代码频繁变动时,索引可能与实际文件不一致。Claude Code 直接读原始文件,不存在这个问题。大规模并行任务且代码库相对稳定时,索引更高效;个人开发且代码在密集修改中时,直接读文件更可靠。

**上下文压缩:客户端工程 vs 训练级优化**

面对上下文腐烂这一核心挑战,Claude Code 和 Cursor 在工程层面也走了不同的路径:

| 维度         | Claude Code                                                                    | Cursor                                                       |
| ------------ | ------------------------------------------------------------------------------ | ------------------------------------------------------------ |
| **压缩实现** | 四层系统(AutoCompact + MicroCompact + SessionMemory + Compaction),全部在客户端 | 工程层面依赖云端向量索引减少进入 context 的内容,降低压缩压力 |
| **压缩位置** | 客户端                                                                         | 索引在云端(预计算),压缩压力因此降低                          |
| **优势**     | 不依赖训练基础设施,可快速迭代                                                  | 索引预过滤大幅减少 context 负载                              |
| **劣势**     | 效率天花板受限于客户端工程                                                     | 需要云端索引基础设施,索引存在时效性风险                      |

Claude Code 的四层压缩系统将客户端工程做到了极致:AutoCompact 和 MicroCompact 是纯代码逻辑(触发条件判断、截断工具结果),SessionMemory 通过后台 fork 子代理提取关键信息,Compaction 用 LLM 生成结构化摘要。Cursor 在工程层面则选择用基础设施(云端向量索引)来减少进入 context 的内容,从而降低对压缩的需求。

但 Cursor 的野心不止于工程优化。在训练层面,Cursor 进一步推进了 Self-Summarization,标志着它从"工具"向"自主模型"的跨越。

#### 训练级压缩:Self-Summarization

"Self-reflection / summarization" 在强化学习和 LLM 研究中早已存在,但 Cursor 把它从研究概念推进到了**工程化、训练级落地**--这是目前少数将压缩机制推进到训练层面的工程实践之一。

**机制**:

1. 训练时,将多轮生成链通过摘要串联
2. 达到固定 token 长度 → 插入合成查询 → 模型自我总结 → 用压缩上下文继续
3. 自我摘要本身成为奖励信号的一部分

**结果**:对比精心调优的基于 prompt 的压缩基线,token 效率显著提升,可以从远超上下文窗口长度的轨迹中获取训练信号。

这不是"更聪明的压缩 prompt",而是"让模型学会如何压缩"。

值得思考的是:为什么 Claude Code 没有采用类似的训练级压缩?可能的原因有几个:(1)训练目标不同--Cursor 可以针对长周期自主编码这一个垂直场景做专项训练,而 Anthropic 需要训练通用模型;(2)工程哲学不同--Anthropic 的原则是"先从 prompt-level 解决,只在必要时才到训练层",四层压缩系统就是这个哲学的产物;(3)开放 API 的约束--专项训练的模型在短对话等场景下是否有副作用,需要权衡。当然,也可能 Anthropic 在训练中做了类似的事情,只是以能力改进的形式体现,没有单独命名。

**从工程到模型的进化**:在上下文压缩这个维度上,Cursor 的发展路径非常清晰--先用云端向量索引(工程基础设施)减少 context 负载,再用 Self-Summarization(训练级优化)让模型自身学会压缩。前者是工程能力,后者是模型能力。这个路径体现了 Cursor 从"编码工具"向"自主编码模型"的跨越。

#### 工程美学对比

| 维度               | Claude Code                                                                  | Cursor                                                       |
| ------------------ | ---------------------------------------------------------------------------- | ------------------------------------------------------------ |
| **整体架构复杂度** | 相对精简:主要本地运行,单进程                                                 | 更庞大:云端 VM、向量索引、训练基础设施、大规模并发调度       |
| **单点工程极限**   | 更极致:四层压缩、Bash 分段权限、Agent Memory 三级存储,将客户端工程做到天花板 | 相对克制:问题更多拆给基础设施解决(索引、训练级优化、VM 隔离) |
| **设计目标**       | 个人开发者的最佳 AI 协作工具                                                 | 自动化工程规模(数千 Agent 并行一周)                          |
| **代码理解方式**   | 直接读原始文件(无预索引)                                                     | 云端向量索引 + 语义搜索                                      |
| **压缩策略**       | 客户端工程(四层系统)                                                         | 工程层(云端索引) + 训练层(Self-Summarization)                |
| **多代理约束**     | 软约束(提示词引导 Coordinator 行为)                                          | 硬约束(代码层面切断 Worker 间通信)                           |

**观察**:Claude Code 和 Cursor 代表了两种截然不同的工程美学--前者在单点上把工程做到极致,后者用基础设施的复杂度换取规模能力。

### 3.3 可观测性:应对不确定性的基础设施

Cursor 在《Self-Driving Codebases》中强调:他们投入大量精力在可观测性上--日志所有 Agent 消息、系统动作、命令输出。

这不是锦上添花,而是系统规模的刚需:

- 当数千个 Agent 并行工作时,出问题是必然的
- 没有细粒度的日志,根本不知道是谁干了什么
- 可观测性让系统从"黑盒"变成"可调试的灰盒"

**对三个挑战的回应**:

可观测性不直接解决任何一个挑战,但它为所有挑战的**调试和改进**提供了基础设施。当上下文腐烂导致质量下降时,你需要日志来定位问题;当记忆断裂导致状态丢失时,你需要日志来重建上下文;当自我评估偏差导致验收不严时,你需要日志来追溯责任。

### 3.4 Cursor 的工程哲学

1. **规模优先**:从一开始就瞄准"数千 Agent 协作一周"的规模
2. **硬层级**:放弃自协调的浪漫,拥抱 Planner-Worker-Judge 的刚性结构
3. **训练驱动**:不满足于 prompt-level 调优,而是训练模型来适应 Harness
4. **可观测性是基础设施**:不是可选项,而是必需品

---

## 四、OpenAI Codex CLI:API 原生化的 Harness Engineering

OpenAI 的 Codex CLI（下文简称 Codex）有一个独特的架构优势:Responses API 将状态管理、上下文压缩等 Harness 基础设施内置到了 API 服务端。模型本身仍然输出标准的 text 和 tool call,区别在于 API 服务端帮 Harness 做了更多事情--Anthropic 的 Messages API 是无状态设计,压缩和状态管理全部由 Claude Code 在客户端自行实现;OpenAI 的 Responses API 是有状态设计,服务端可以存储会话状态、提供专用的压缩端点。前者是"客户端自己做",后者是"API 帮你做"。

### 4.1 Agent Loop:从概念到实现

Codex 在其工程博客《Unrolling the Codex Agent Loop》中详细拆解了 Agent Loop 的实现。这篇文章本身就将 Codex 的控制层称为 "harness",这为我们理解 OpenAI 的 Harness Engineering 思路提供了第一手资料。

Codex 的 Agent Loop 核心流程:

```
用户输入 → 构建 Prompt(instructions + tools + input)→ 模型推理 →
  → 输出文本(结束)或 请求工具调用(继续循环)
```

**关键设计**:Codex 使用 OpenAI 的 **Responses API**(而非传统的 Chat Completions API)来驱动 Agent Loop。Responses API 专门为 Agent 场景设计,支持:

- **结构化的 input 列表**:每个元素有明确的 role(system / developer / user / assistant)和 type
- **工具定义的标准化 schema**:Codex 提供的工具、Responses API 内置工具、用户 MCP 工具统一管理
- **原生 compaction 端点**:`/responses/compact` 端点专门用于上下文压缩

这意味着 Codex 将上下文压缩下沉到 API 服务端(通过 `/responses/compact` 端点),客户端不需要自行实现压缩算法。但 Agent Loop 的编排、工具执行和对话历史管理仍然在客户端完成--Codex 采用无状态请求模式,每次都发送完整对话历史,与 Claude Code 在状态管理上的工作方式类似。

### 4.2 Prompt 构建:多层指令注入

Codex 的 Prompt 构建机制体现了对"问题三角"中上下文管理的深度思考:

**Prompt 结构**(按优先级排列):

1. **System message**:模型级别的系统指令(由 Responses API 服务端控制)
2. **Instructions(developer message)**:开发者指令,包含沙箱策略、权限模式等
3. **Tools**:工具定义列表(Codex 工具 + API 内置工具 + MCP 工具)
4. **User instructions**:项目级配置,从多个来源聚合:
   - `AGENTS.override.md` 和 `AGENTS.md`(全局,`$CODEX_HOME`)
   - 项目根目录到当前目录的 `AGENTS.md` / `AGENTS.override.md`(有 32KB 限制)
   - Skills 配置(如果已配置)
5. **对话历史**:之前所有的 user/assistant 消息和工具调用结果

**对上下文腐烂的回应**:

Codex 的 Prompt 构建有两个关键特点:

- **静态内容前置**:instructions、tools 等 static content 放在 prompt 最前面,variable content(对话历史)放在最后。这不是随意的设计--它是为了最大化 **Prompt Caching** 的命中率。Prompt Caching 只对精确的前缀匹配生效,静态内容前置确保每次请求都能复用之前的计算结果
- **配置变更追加而非修改**:当沙箱配置、审批模式或工作目录在对话中途发生变化时,Codex 不会修改之前的消息(那会导致 cache miss),而是追加一条新消息来反映变更。为了 cache hit,Codex 团队"不遗余力"

这是一个很有 OpenAI 特色(或者说 API 厂商特色)的思路:**利用 API 层面的能力来解决 Harness 问题**。Prompt Caching 不是工程技巧,而是 API 提供的基础设施--Codex 的 Harness 设计充分利用了这一点。

### 4.3 Compaction:API 原生的上下文压缩

Codex 的上下文压缩也体现了 API 原生化的特点:

**演进过程**:

1. **早期**:用户手动执行 `/compact` 命令,Codex 调用 Responses API 生成摘要,用摘要替换完整对话历史
2. **现在**:Responses API 提供了专用的 `/responses/compact` 端点,Codex 自动在 token 超过阈值(`auto_compact_limit`)时触发

**Compaction 端点的特殊之处**:

压缩结果不是简单的文本摘要,而是返回一个 **item 列表**,其中包含一个特殊的 `type=compaction` 元素,带有 `encrypted_content`(加密的不透明内容)。根据 OpenAI 的描述,这个加密内容保留了模型对原始对话的某种连续性表示,使模型在后续推理中仍能"感知"到之前的上下文——但具体实现机制是不透明的,不应过度解读为某种神奇的记忆机制。

**与其他厂商的对比**:

| 维度         | Codex                               | Claude Code            | Cursor                     |
| ------------ | ----------------------------------- | ---------------------- | -------------------------- |
| **压缩触发** | 自动(token 阈值)                    | 自动(接近上下文窗口时) | 自动 + 训练级              |
| **压缩实现** | API 端点(`/responses/compact`)      | LLM 生成摘要(四层系统) | Self-Summarization(训练级) |
| **压缩产物** | item 列表 + 不透明的加密压缩表示    | 文本摘要 + 结构化状态  | 压缩后的上下文继续推理     |
| **独特优势** | 服务端压缩,不透明但维持上下文连续性 | 多层系统覆盖宏观到微观 | 训练级优化,token 效率最高  |

Codex 的 `encrypted_content` 设计提供了一种服务端的不透明压缩表示。原始对话的具体 token 丢失了,但通过加密内容维持了某种上下文连续性。这比纯文本摘要更底层,但具体保留了什么信息、如何保留,OpenAI 没有公开细节。

### 4.4 沙箱与安全:分层隔离

Codex 的沙箱设计体现了"分层隔离"的思路:

- **Codex 提供的 shell 工具**:在沙箱内运行,受 Codex 控制
- **MCP 工具**:不在 Codex 沙箱内,由 MCP 服务器自行实施安全策略
- **用户自定义工具**:同样不受 Codex 沙箱控制

这种分层不是"一刀切"的隔离,而是明确区分了"谁负责安全"--Codex 只对自己提供的工具负责,第三方工具的安全性由提供方负责。

### 4.5 AGENTS.md:标准化配置

OpenAI 和 Anthropic 都采用了项目级配置文件来引导代理行为:

- OpenAI:**AGENTS.md**
- Anthropic:**CLAUDE.md**(早期) → AGENTS.md(现在统一)

Codex 的 AGENTS.md 有一个独特的聚合机制:从全局目录(`$CODEX_HOME`)到项目根目录逐级收集,更具体的指令出现在更后面(优先级更高),且有 32KB 的总量限制。这比简单的"读一个文件"更精细,但也增加了配置管理的复杂度。

### 4.6 Codex 的工程哲学

从 Agent Loop 的拆解中,可以提炼出 Codex 独特的工程哲学:

1. **API 原生化**:将 Harness 基础设施(状态管理、压缩、缓存)下沉到 API 服务端,由 Responses API 直接提供,而非在客户端自行实现(Responses API、Compaction 端点、Prompt Caching)
2. **无状态请求**:每次请求都是完全自包含的 JSON payload,不依赖服务端状态。这牺牲了一些效率(无法使用 `previous_response_id`),但换来了 Zero Data Retention(ZDR)兼容性
3. **性能是首要约束**:Agent Loop 每次请求都要发送完整对话历史,成本随会话增长线性累积。Codex 通过 Prompt Caching 让静态前缀(指令、工具定义等)无需重复计算,显著降低 token 成本与延迟。为此不惜在设计中做出很多"不自然"的选择(配置变更追加而非修改、工具顺序必须一致)
4. **分层安全**:Codex 沙箱只管自己的工具,第三方工具的安全由提供方负责

---

## 五、OpenClaw:抽象优先的 Harness Engineering

OpenClaw 的定位不同于前几家--它不是专门的编码工具,而是一个**个人 AI 助理网关**,支持多渠道、多节点、多模型。OpenClaw 也有自己的 Harness 实践(可插拔 Context Engine、通用 Subagent 等),但由于定位不同,其 Harness 相关实践的精密度和复杂度与前述三个编码领域的产品不在同一量级。OpenClaw 的架构亮点更多体现在多渠道集成、插件化扩展和模型无关的抽象设计上,而非 Harness 本身。本文将其纳入讨论,是因为它提供了一个不同于编码工具的、更通用的 Harness 视角。

### 5.1 Context Engine:可插拔的上下文管理

OpenClaw 实现了一个可插拔的 Context Engine 接口:

```typescript
interface ContextEngine {
  readonly info: ContextEngineInfo;

  bootstrap?(params: { sessionId; sessionFile }): Promise<BootstrapResult>;
  maintain?(params: {
    sessionId;
    sessionFile;
    runtimeContext;
  }): Promise<MaintenanceResult>;
  ingest(params: { sessionId; message }): Promise<IngestResult>;
  assemble(params: { sessionId; tokenBudget; prompt }): Promise<AssembleResult>;
  compact(params: { sessionId; tokenBudget; force }): Promise<CompactResult>;

  prepareSubagentSpawn?(params: {
    parentSessionKey;
    childSessionKey;
    ttlMs;
  }): Promise<SubagentSpawnPreparation>;
  onSubagentEnded?(params: { childSessionKey; reason }): Promise<void>;
}
```

**对上下文腐烂的回应**:

OpenClaw 的思路是"抽象优先"--上下文管理不是硬编码的,而是通过接口暴露,可以替换不同实现。

特点:

- 支持 **Transcript Rewrite**:引擎可以请求安全的 transcript 重写
- **Subagent 生命周期感知**:可以在子代理启动/结束时做状态管理
- Compaction 实现相对简洁,基于 PI Agent Core 的 `generateSummary` API,分块压缩(BASE_CHUNK_RATIO = 0.4),保留结构化信息

与 Claude Code 的四层压缩系统相比,OpenClaw 更简单,但更灵活--相信社区会想出不同的上下文管理策略,框架只提供接口。

### 5.2 Subagent 系统:通用的多代理协调

OpenClaw 的子代理系统是通用的:

- 不局限于编码任务
- 支持不同的 `mode`(`run` vs `session`)
- 支持 `runtime="acp"`(ACP coding agent)或 `runtime="subagent"`(普通 subagent)
- **Workspace 继承**:子代理自动继承父级工作目录

**对自我评估偏差的回应**:

OpenClaw 没有像 Claude Code 那样硬编码 Coordinator 模式,也没有像 Cursor 那样实现 Judge Agent。它的多代理系统是通用的--不预设角色分工,由使用者通过 prompt 和工具组合来组织协调方式。

这反映了不同的工程世界观:

| 维度         | Claude Code                          | OpenClaw                      |
| ------------ | ------------------------------------ | ----------------------------- |
| **协调方式** | 硬编码 Coordinator 模式              | 通用 Subagent,灵活组合        |
| **验证机制** | fresh Worker 验证,硬编码在系统提示中 | 无预设验证机制,依赖使用者设计 |
| **适用场景** | 编码任务(领域专有)                   | 通用任务(领域无关)            |

### 5.3 Plugins:集成与扩展

OpenClaw 支持插件系统,覆盖 Channels(消息渠道)、Tools(工具扩展)、Memory(记忆后端)等维度。关键设计选择是 Core 保持精简,可选能力通过插件提供,插件优先发布到独立仓库。

### 5.4 System Prompt:动态组装

OpenClaw 的系统提示是动态组装的:

```typescript
type PromptMode = "full" | "minimal" | "none";

// full: 所有 section(主代理)
// minimal: 减少 section(子代理)
// none: 只有身份行(特殊用途)

// 包含的 section:
// - Skills、Memory、Tooling、Workspace、Runtime、Authorized Senders
```

子代理使用 `minimal` 模式减少上下文噪音,主代理使用 `full` 模式获得完整指导。这种动态组装本身也是对上下文腐烂的一种回应--通过控制注入给子代理的信息量,降低不必要的上下文膨胀。

### 5.5 OpenClaw 的工程哲学

1. **抽象优先**:Context Engine 可插拔、Subagent 通用化、Prompt 动态组装
2. **插件化**:Core 保持精简,可选能力通过插件提供
3. **模型无关**:支持多家提供商,不绑定特定模型
4. **社区驱动**:插件优先发布到独立仓库(如 ClawHub),核心合并门槛很高
5. **通用而非专有**:不针对特定领域优化,追求广泛的适用性

---

## 六、工程实践的差异化对比

前面五章分别梳理了各家的实践。现在,围绕 Harness Engineering 的三个核心挑战,做一次横向对比。

### 6.1 三个核心挑战的横向对比

下表从三个核心挑战出发,对比各家的回应方式:

| 厂商                 | 上下文腐烂(Context Rot)                                               | 记忆断裂(Context Reset)                                                                                 | 自我评估偏差(Self-Evaluation Bias)             |
| -------------------- | --------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------- | ---------------------------------------------- |
| **Claude Code**      | 四层压缩系统(AutoCompact + MicroCompact + SessionMemory + Compaction) | Agent Memory 三级存储 + SessionMemory + scratchpad                                                      | Coordinator 合成发现 + fresh Worker 独立验证   |
| **Cursor**           | Self-Summarization(训练级压缩)+ Planner 递归子规划拆分上下文          | 云 VM 环境持久化 + Planner 管理任务状态 + 结构化日志                                                    | Planner-Worker-Judge 硬层级,Judge 每轮全局判断 |
| **OpenAI Codex CLI** | API 原生 Prompt Caching(静态前置)+ Compaction 端点(不透明加密压缩)    | 无跨会话记忆(主动设计:每次任务从仓库快照开始,用确定性换安全性),单任务内通过 compaction 维持上下文连续性 | 无内置评估机制,依赖用户判断和可验证输出        |
| **OpenClaw**         | 可插拔 Context Engine 接口,可替换压缩实现                             | Skills 携带领域知识 + 子代理 Workspace 继承                                                             | 通用 Subagent,不预设验证机制,由使用者设计      |

**从这张表可以观察到几个有意思的模式**:

1. **上下文腐烂**:各家沿着两个维度展开--"怎么压缩"(prompt-level 到 training-level)和"怎么隔离"(Sprint 分块到 VM 隔离)
2. **记忆断裂**:存在一个"持久化 vs 隔离"的光谱--Claude Code 选择在隔离的会话间传递结构化信息,Cursor 选择让整个环境持久化,Codex 选择干脆不记忆
3. **自我评估偏差**:Claude Code/Cursor 都选择了"分离",但分离的粒度不同--Claude Code 在 Worker 级分离,Cursor 在角色级分离,OpenClaw 完全不预设

### 6.2 安全策略的差异化

安全虽然不是"问题三角"的直接组成部分,但它是所有 Harness Engineering 实践的基础约束。不同部署形态决定了不同的安全策略:

| 厂商             | 部署形态                   | 安全策略                                             | 核心思路             |
| ---------------- | -------------------------- | ---------------------------------------------------- | -------------------- |
| **Claude Code**  | 本地运行                   | 极细粒度权限(Bash 分段检查、cd+git 检查)             | 宁可多问,不可绕过    |
| **Cursor Cloud** | 云端运行                   | VM 隔离                                              | 隔离即安全           |
| **Codex CLI**    | 本地运行(API 调用云端模型) | 分层沙箱:Codex 工具沙箱化,MCP/第三方工具由提供方负责 | 明确划分安全责任边界 |
| **OpenClaw**     | 本地/自托管                | 权限系统 + 沙箱选项                                  | 强默认,可放宽        |

**观察**:安全策略的选择本质上取决于部署形态,而非工程哲学。云端运行天然有隔离优势,本地运行则需要更细粒度的权限系统。

---

## 七、重工程还是重模型:Harness Engineering 的核心争论

随着 Harness Engineering 实践的积累,一个核心争论浮现出来:**当模型能力足够强时,还需要复杂的工程脚手架吗?**

### 7.1 "重模型"派:相信模型

观点:

- 模型会越来越强,上下文窗口会越来越大
- 复杂的工程脚手架是"补丁",随着模型进步会被淘汰
- 应该简化 Harness Engineering,让模型自己学会如何组织任务

论据:

- Anthropic Opus 4.6 的出现(模型自身行为改善,减少了对 Context Reset 的依赖)让一些 Sprint 结构变得不必要
- 大型模型的规划能力在提升,可能不需要 Planner-Worker 的硬分层
- 过度工程化会限制模型的灵活性

代表:Anthropic(部分)、一些"模型原生"的研究者

### 7.2 "重工程"派:不相信模型

观点:

- 无论模型多强,总有边界
- 复杂任务的本质是"不确定性",需要工程化保障
- Harness Engineering 的价值不随模型进步而消失,只是转移

论据:

- Cursor 的经验:即使有强模型,大规模协作仍需要硬层级
- 安全性和可观测性是工程问题,不是模型问题
- 生成-评估分离避免自我偏差,这是架构优势,不是补丁

代表:Cursor、OpenClaw(部分)、企业级应用实践者

### 7.3 一个更细致的框架

或许,这不是二选一的问题。围绕"问题三角",我们可以更细致地分析:

**Harness Engineering 各组件的价值随模型进步的变化**:

| 组件              | 回应的挑战                | 模型弱时 | 模型强时          | 价值变化   |
| ----------------- | ------------------------- | -------- | ----------------- | ---------- |
| **上下文压缩**    | 上下文腐烂                | 必需     | 可选              | ↓ 降低     |
| **Sprint 结构**   | 上下文腐烂 + 自我评估偏差 | 必需     | 可选              | ↓ 降低     |
| **结构化工件**    | 记忆断裂                  | 必需     | 仍然需要,但可简化 | → 基本不变 |
| **生成-评估分离** | 自我评估偏差              | 必需     | 简单任务可选      | ↓ 降低     |
| **安全/权限**     | 基础约束                  | 必需     | **仍然必需**      | → 不变     |
| **可观测性**      | 三个挑战的调试            | 重要     | **更重要**        | ↑ 升高     |
| **标准化配置**    | 跨实现兼容                | 可选     | **必需**          | ↑ 升高     |

**观察**:

- **会降低的**:那些"补偿模型不足"的部分--上下文压缩、Sprint 分块、评估分离。随着模型进步,这些"补丁式"组件可以简化
- **不变的**:安全等非能力相关问题--无论模型多强,权限控制始终需要;结构化工件作为可读的状态持久化手段也有独立价值
- **会升高的**:那些"随着规模和复杂性更关键"的部分--系统越大,可观测性和标准化越重要

### 7.4 一个可能的共识方向

综合来看,一个可能的共识方向是:

> **Harness Engineering 的重心会从"补偿模型不足"转移到"保障系统可靠"。**

早期(模型较弱时):

- 上下文窗口小 → 需要压缩(回应上下文腐烂)
- 模型规划弱 → 需要硬分层(回应记忆断裂 + 自我评估偏差)
- 自我评估偏差大 → 需要分离(回应自我评估偏差)

未来(模型较强时):

- 模型很强 → 压缩和分层可以简化,上下文腐烂和自我评估偏差不再是最紧迫的问题
- 系统很大 → 可观测性、安全、标准化变得关键
- 任务很复杂 → 工程保障是刚需,记忆断裂仍是需要持续关注的问题

这不是"重工程还是重模型"的问题,而是**工程重心转移**的问题。

---

## 八、趋势与开放问题

### 8.1 正在形成的趋势

以下是各团队在实践中逐渐呈现出的共同方向,但需要注意,它们更多是"趋势"而非"行业共识":

1. **上下文管理从 prompt 级走向训练级**:Cursor 的 Self-Summarization 标志着一个值得关注的探索方向,但目前仅 Cursor 在训练层面推进了这一实践,其他厂商尚未跟进,能否成为趋势尚待观察
2. **在复杂任务和长流程场景中,多代理逐渐呈现为一种结构性趋势**:但需要注意,Anthropic 明确强调"能不用 Agent 就不用",很多实际系统仍然是 single-agent + tools
3. **硬层级在大规模协作中展现出更高的可靠性**:各团队有向 Planner-Worker 模式收敛的趋势,但这是在"大规模"前提下才成立的
4. **云沙箱成为大规模代理运行的主流部署形态**:隔离性、安全性、并行性的综合优势明显
5. **AGENTS.md 正在成为跨实现的项目配置事实标准**:OpenAI 和 Anthropic 的共同采纳加速了这一趋势
6. **可观测性从"锦上添花"变为"基础设施"**:系统规模越大,可观测性越关键
7. **自主化程度持续提升**:从"人在回路中"到"人定义 + 代理执行 + 人审查"
8. **领域知识的标准化封装(Agent Skills)**:Anthropic 推出的 Agent Skills 标准展示了一种新的趋势--通过"渐进式披露"的文件夹结构来打包指令和工具。这不仅直接回应了 Context Rot 问题(按需加载而非一次性塞入),也为 Agent 在不同垂直领域的能力扩展提供了一种标准化的工程实践

### 8.2 尚无定论的开放问题

1. **标准化需要走多深?** AGENTS.md 作为自然语言描述的起点很好,但当系统复杂度提升后,是否需要更结构化的配置格式?
2. **通用框架 vs 领域专用的边界在哪里?** OpenClaw 的抽象优先和 Claude Code 的领域专有各有优势,但适用场景的分界线在哪里?
3. **模型厂商能否在训练时就教会模型与 Harness Engineering 协作?** 比如,训练时直接包含多代理协调、结构化工件读写等场景?
4. **Harness Engineering 如何向业务层面延伸?** 目前的实践集中在编码领域,但方法论是否可以扩展到更广泛的业务流程管理?这是值得持续关注的方向,留待后续深入讨论。

---

## 九、总结

Harness Engineering 不是某个具体的产品,也不是一个被正式定义的标准术语。它是工程实践中的一个"收敛性抽象"--不同团队在解决同一个问题时,逐渐走向相似的系统结构。

它继承了软件工程几十年积累的 Test Harness 思想--**如何可靠地、可重复地、可观测地运行一个复杂组件**--只是控制对象从确定性函数变成了一个具有随机性和自主性的 AI Agent。

长时间运行代理面临的三个核心挑战--**上下文腐烂、记忆断裂、自我评估偏差**--构成了 Harness Engineering 的"问题三角"。各家大厂的实践,都可以理解为对这个三角不同角度的回应:

- **Anthropic / Claude Code**:侧重模型(工程上保守、模型上信任),理论框架先行,编码领域特化。四层压缩回应上下文腐烂,Context Reset + 结构化 artifact 回应记忆断裂,Coordinator 模式(Workflow+Agent 嵌套)回应自我评估偏差
- **Cursor**:架构层侧重工程,模型层侧重训练(自研 Composer),双轨并行。Self-Summarization 从训练层面解决上下文腐烂,硬层级 + Judge 回应自我评估偏差,可观测性作为基础设施
- **OpenAI Codex CLI**:侧重 API 架构,将 Harness 基础设施下沉到 API 服务端。Prompt Caching 解决性能问题,Compaction 端点提供不透明的上下文压缩,分层沙箱明确安全责任
- **OpenClaw**:侧重抽象,可插拔 Context Engine、通用 Subagent、插件化能力扩展

"重工程还是重模型"的争论,或许是一个错误的二分法。正在呈现的趋势是:**Harness Engineering 的重心会从"补偿模型不足"转移到"保障系统可靠"。**

上下文压缩、Sprint 结构、评估分离等"补丁式"组件可能会随着模型进步而简化,但安全、可观测性、标准化等"基础设施型"组件会变得更加重要。而记忆断裂--作为跨会话状态管理的本质问题--无论模型多强,都需要工程层面的持续关注。

Harness Engineering 目前仍以编码领域为主要实践场景,但它的方法论--状态管理、多代理协调、上下文工程、可观测性--有着更广阔的应用空间。如何将其延伸到业务层面,是值得持续关注的方向。

---

**脚注**

1 GAN(生成对抗网络)的核心是对抗训练--Generator 试图骗过 Discriminator,Discriminator 试图区分真假。Planner-Generator-Evaluator 实际上更接近流水线 + 检查点结构,Evaluator 不与 Generator 对抗训练,而是独立验收。更准确的技术参照是强化学习中的 Critic 模型,或学术界常用的 Best-of-N + verifier 范式。不过"受 GAN 启发"是 Anthropic 原文的表述,本文沿用。

---

## 参考来源

- Anthropic: Building Effective Agents, Effective Harnesses for Long-Running Agents, Effective Context Engineering for AI Agents, Harness Design for Long-Running Apps
- Cursor: The Third Era of AI Software Development, Scaling Long-Running Autonomous Coding, Towards Self-Driving Codebases, Training Composer for Longer Horizons, Composer 2 Technical Report
- OpenAI: Unrolling the Codex Agent Loop, Introducing Codex, Codex CLI Developer Documentation, Codex CLI (https://github.com/openai/codex)

- OpenClaw: https://github.com/openclaw/openclaw, VISION.md, source code
- Claude Code: leaked source code via npm .map files, coordinatorMode.ts, compact.ts, agentMemory.ts
- EleutherAI: lm-evaluation-harness (https://github.com/EleutherAI/lm-evaluation-harness)

---

_撰写于 2026-04-04,基于截至该日的公开文章和开源代码。_
_本文提出的"Harness Engineering"是一种事后分析视角和抽象框架,用于统称各团队在 Agent 控制与编排层上的工程实践。它不是行业标准术语,也不是某个厂商的官方定义。_
_文中对 Anthropic、Cursor、OpenAI 等厂商实践的分析基于其公开文章和开源代码,可能存在理解偏差。_
