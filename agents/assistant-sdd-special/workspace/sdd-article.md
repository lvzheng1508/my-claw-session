# 规格驱动开发（SDD）：从形式化方法到 AI 时代的演进与反思

---

## 引言

当 AI 编码助手能在几秒内生成一个完整的功能模块时，一个问题变得前所未有的紧迫：**谁来保证 AI 写的代码做的是对的事？**

这不是一个新问题。从 1960 年代软件工程诞生之初，"如何精确定义系统应该做什么"就是核心命题。六十年间，从数学化的形式化规格，到测试驱动的行为规格，再到今天 AI 时代的可执行规格，这条线一直延伸，从未断裂。

这条线有一个共同的名字：**规格驱动开发（Specification-Driven Development, SDD）**。

SDD 的核心理念并不复杂——在动手实现之前，先用精确的语言定义"系统应该做什么"。但围绕这个核心理念，过去六十年诞生了数十种方法论、数百种工具、无数种实践路径。它们各有取舍，各有成败，各有适用的边界。

本文试图做一件事：**站在 2026 年的时间节点，以宏观视角审视 SDD 的发展脉络、方法论谱系、本质与局限，并回答一个对当下每一个技术团队都至关重要的问题——在 AI 编码盛行的今天，我们该如何看待 SDD？**

---

## 一、SDD 的发展脉络

SDD 不是某个人的发明，也不是某种方法论的品牌名。它是一条持续了六十年的思想脉络，每一次浪潮都在回答同一个问题：**如何让"做什么"比"怎么做"更清晰？**

回顾这条脉络，可以看到一个清晰的"降权"趋势——SDD 从需要数学博士学位才能使用的"贵族专用工具"，一步步演变为任何开发者都能上手的日常实践。而 AI 编码的出现，则让这个过程完成了最后的闭环。

### 1.1 远古时代（1960s—1980s）：用数学定义正确性

软件危机催生了软件工程，也催生了最早的 SDD 实践——只不过那时它叫"形式化规格"（Formal Specification）。

**Z Notation**（1970s）基于集合论和一阶逻辑，用数学符号精确描述系统的状态和操作 [1]。**VDM**（Vienna Development Method）由 IBM 维也纳实验室提出，是最早的系统化形式化方法之一 [2]。**TLA+**（Leslie Lamport）用时序逻辑描述并发系统的行为 [3]。**B-Method**（Jean-Raymond Abrial）进一步将形式化规格与逐步精化（refinement）结合，能够在数学上证明实现的正确性 [4]。

这一时期的核心信念是：**如果规格足够精确，就能在写代码之前证明系统是正确的。**

这个信念在理论上成立，在实践中却碰了壁。形式化方法的门槛极高，开发者需要先学数学建模；表达非功能性需求（性能、可用性）极为困难；与当时的开发流程严重脱节。最终，形式化方法退守到了"不能死机"的领域——航空航天、铁路信号、核能控制、芯片验证。

但它们从未消失。Amazon 用 TLA+ 验证 DynamoDB 的分布式设计，发现了手工评审无法发现的深层并发 bug [5]。Intel 用形式化方法验证芯片设计。Microsoft 用 TLA+ 验证 Azure 基础设施。这些实践证明：**形式化规格不是银弹，但在特定场景下，它是不可替代的。**

只是，对于绝大多数软件项目来说，它太重了。SDD 需要找到一条更亲民的路。

### 1.2 黄金时代（1990s—2000s）：从数学到代码，从精英到大众

1990 年代，SDD 开始寻找更亲民的形态。核心思路是：**既然数学语言太重，那就用代码语言来表达规格。**

**Design by Contract**（Bertrand Meyer, 1986 提出，1990s 在 Eiffel 语言中推广）是一个里程碑 [6]。它的核心思想是：将规格嵌入代码本身——用前置条件（precondition）、后置条件（postcondition）和不变式（invariant）定义函数的契约。调用者必须满足前置条件，实现者必须保证后置条件。规格不再是独立于代码的文档，而是代码的一部分。

这一思想深刻影响了后世。Java 的 assert、Python 的 type hints、Rust 的类型系统——它们在不同程度上都是 Design by Contract 的继承者。Rust 的类型系统甚至被描述为"编译期的契约"。

2000 年代初，**Test-Driven Development**（Kent Beck, 2002）将 SDD 推向了主流 [7]。TDD 的循环——Red（写失败的测试）、Green（写最小实现）、Refactor（重构）——本质上是一种行为规格的可执行化。测试用例就是规格，测试通过就是规格被满足。

TDD 的伟大在于它**降低了 SDD 的门槛**。你不需要懂数学，不需要学形式化语言，只需要会写代码就能定义规格。Google 在内部大规模推广 TDD，ThoughtWorks 将其作为核心工程实践，Microsoft 推荐内部团队采用。

同年，**Behavior-Driven Development**（Dan North, 2003）进一步降低了门槛 [8]。BDD 用 Given-When-Then 的自然语言格式描述行为，让非技术人员也能参与规格定义。Cucumber [9]、Behave、SpecFlow 等工具让 BDD 场景可以自动转化为可执行测试。LinkedIn 在大规模 BDD 实践中构建了完整的自动化验收测试体系。

**Specification by Example**（Gojko Adzic, 2011）则从另一个角度切入 [10]：不写抽象规格，而是用**具体的业务实例**来定义系统行为。eBay、Guardian 等企业用 SbE 将业务、开发、测试三方拉到同一张桌前，用例子代替争论。

### 1.3 云原生时代（2010s）：接口即契约

2010 年代，微服务的流行让规格的重心从"系统内部行为"转移到了"服务间通信边界"。

**API-First Development** 成为了微服务架构的事实标准 [11]。Stripe、Twilio、Google 等公司将 API 契约视为产品本身——先定义 OpenAPI/GraphQL Schema，再生成文档、SDK、测试。Stripe 的 API 设计甚至成为了业界标杆，其版本控制和兼容性策略被广泛模仿。

**Consumer-Driven Contract Testing** 让规格变成了服务间的一份"合同"。Pact（最初在澳洲 REA Group 诞生）[12] 让服务的消费者定义期望的契约，Provider 自动验证。Netflix 在其微服务架构中大规模采用契约测试，实现了服务的独立部署。

**Protocol Buffers**（Google）[13] 和 **Thrift**（Facebook/Meta）[14] 成为服务间通信的契约语言。Google 将 Protocol Buffers 作为全公司的 RPC 契约标准，Meta 用 Thrift 连接数千个内部服务。

这个时代的关键特征是：**规格不再是开发者的个人习惯，而是组织级别的基础设施。** 当系统被拆分成几十上百个微服务时，没有契约规格，集成就变成了一场灾难。

### 1.4 AI 时代（2024+）：规格的"文艺复兴"

为什么 SDD 在今天突然"火"了？因为它解决了一个 AI 编码时代最迫切的问题。

过去，SDD 长期处于"理论很美好，现实很骨感"的尴尬境地——人工编写和维护规格的成本极高，规格写完代码还没写，代码一改规格就废了（规格腐烂）。很多团队不是不想做 SDD，而是做不起。

AI 改变了这个成本结构。

2024-2025 年，AI 编码助手的爆发式增长催生了两个变化：

当 GitHub Copilot、Cursor、Claude Code、Codex 等工具能快速生成代码时，一个老问题以新面貌出现：**AI 理解你想要什么吗？**

"Vibe coding"——凭感觉让 AI 写代码——在原型探索中很有价值，但在生产环境中，它意味着不确定性。你给 AI 一段 prompt，它给你一段代码，但这段代码是否做了你真正想做的事？你不知道。下一次给同样的 prompt，它可能给你完全不同的代码。

**如果你让 AI 直接写代码，它会产生幻觉；但如果你让 AI 先输出规格让你确认，再根据规格写代码，质量会产生质变。**

这催生了 AI 原生的 SDD 工具浪潮：

- **Spec Kit**（GitHub 官方）提出"规格可执行化"——通过 Constitution → Specify → Plan → Tasks → Implement 的严格阶段门控，让 AI 按规格生成代码 [15]。它的宣言是："Focus on product scenarios and predictable outcomes instead of vibe coding."
- **Superpowers**（Jesse Vincent, 99k+ stars）走的是"技能驱动"路线——通过 20+ 个可组合的强制 Skills，让 AI 代理遵循 Brainstorm → Design → Plan → TDD+Subagent → Review → Finish 的完整工程流程 [16]。
- **OpenSpec**（Fission AI）追求极简——Propose → Apply → Archive 三步走，核心特色是 Spec Delta Review：只看需求变更的 diff，不读完整规格，不读代码 [17]。
- **Kiro**（AWS）将 Spec 作为 AI 的"精确上下文"，用 Executable Specs + Steering + Context Management 来控制 AI 的行为边界 [18]。

这些工具有一个共同信念：**在 AI 编码时代，规格不再是过时的文档，而是约束 AI 输出、保障大规模代码质量的"操纵杆"。**

回顾整个发展历程，SDD 经历了一场清晰的"降权"：从需要数学博士学位的形式化方法，到需要编程能力的 TDD，到产品经理也能参与的 BDD，到写 YAML 就能定义的 API 契约，到用自然语言描述就能让 AI 理解的 Executable Specs。每一次"降权"，都让 SDD 的适用范围扩大了一个数量级。而 AI，完成了这个"降权"的最后一步——让规格的生产和验证成本降到了历史最低点。

---

## 二、AI 时代的 SDD 工具生态与工程实践

SDD 在 2025-2026 年的爆发，不是因为理论有了突破，而是因为 AI 编码助手让 SDD 的实践成本发生了质变。围绕 "如何让 AI 按规格写代码" 这一核心问题，一个快速演进的工具生态已经形成。据 Spillwave 2026 年 2 月的统计，四款主流 SDD 工具在 GitHub 上的合计星标数已超过 13.7 万 [22]。

### 2.1 规格光谱：三种严谨度

并非所有 SDD 实践都是同一个物种。一篇 2026 年 2 月发表在 arXiv 上的 SDD 综述论文提出了 "规格光谱" 框架 [23]，将 SDD 实践分为三个层次：

**Spec-First（规格先行）**：在编码之前写规格来指导初始实现，但代码完成后规格可能不再维护。这是 SDD 的入门形态，适合原型开发和一次性功能。与 AI 编码助手配合时，前置规格可以防止 AI 猜测需求，显著提升生成代码的质量。

**Spec-Anchored（规格锚定）**：规格作为 "活文档" 与代码同步维护。行为变更时规格和代码同时更新，自动化测试确保二者对齐。这是大多数生产系统的最佳实践。BDD 框架（如 Cucumber）是这一模式的典型代表。

**Spec-as-Source（规格即源码）**：规格是人类编辑的唯一产物，代码完全由规格生成，禁止手动修改。这是 SDD 最激进的形态——规格实际上是高阶源代码。目前仅在代码生成工具成熟的领域（如从 OpenAPI 生成 API 骨架、从 Simulink 模型生成嵌入式 C 代码）中可行。

理解这个光谱很重要：**大多数团队应该从 Spec-First 入手，在 Spec-Anchored 站稳脚跟，而不是一上来就追求 Spec-as-Source。** ThoughtWorks 在其 2025 年技术雷达中也指出，SDD 仍是一个快速演进的新兴实践，团队应该保持关注并小规模实验，而非急于全面铺开 [24]。

### 2.2 代表性 SDD 工具逐一解析

2025-2026 年，AI 原生的 SDD 工具快速涌现。以下选取四款最具代表性的工具，逐一介绍其定位、工作流、关键特性与局限。

#### Spec Kit（GitHub）[15]

**定位**：GitHub 官方推出的 SDD 工具集，以 CLI 方式分发，为各种 AI 编码助手创建统一的工作空间结构。Spec Kit 是目前社区星标最高的 SDD 工具（70.8k），其核心理念可以用一句话概括：**"Code is no longer king"（代码不再是王道）**——规格才是软件的核心制品，代码只是最后一步的产物 [15]。

**基本工作流**：Constitution → Specify → Plan → Implement

1. **Constitution（宪法）**：项目初始化时创建 `constitution.md`，定义不可变的项目原则和技术约束。这类似于团队的"最高法律"，所有后续规格必须与之对齐。例如："所有 API 必须使用 TypeScript 严格模式"、"禁止直接操作 DOM"。这个设计确保了即使在 AI 自主编码的场景下，也不会偏离组织级的技术标准。

2. **Specify（规格编写）**：用户描述需求，Spec Kit 生成结构化的 `spec.md`，包含功能描述、用户故事、验收标准和边界条件。每个规格会创建一个独立的 Git 分支，便于追踪和评审。

3. **Plan（计划）**：基于规格生成 `plan.md`（实现方案）和 `research.md`（技术调研），并通过内置的 `/speck:analyze` 命令进行交叉一致性检查，确保计划与规格和宪法一致。

4. **Tasks（任务分解）**：将计划分解为带验收标准的任务列表，AI 按序执行。每个任务完成后需要人工确认，形成严格的阶段门验证。

**关键特性**：
- 产出物最丰富：spec.md、plan.md、research.md、data-model.md、contracts 等多份结构化文档
- 兼容 18+ AI 编码工具（Cursor、Claude Code、Windsurf、Copilot 等）
- Constitution 机制提供了组织级的技术治理能力

**局限**：产物丰富意味着仪式感较重，对于小改动（"改个按钮颜色"）显得过于笨重。Martin Fowler 在分析中指出，Spec Kit 为每个规格创建独立分支的做法更像是"变更请求级别的生命周期"，而非"功能级别的持续维护"，因此可能更接近 Spec-First 而非 Spec-Anchored [34]。

#### OpenSpec（Fission AI）[17]

**定位**：一款以 MCP 协议为核心、强调"棕地优先"（Brownfield-first）的轻量级 SDD 框架。OpenSpec 的核心理念是**"Fluid, not rigid, iterative not waterfall"（流动而非刚性，迭代而非瀑布）**，专为已有代码库的增量迭代设计。GitHub 星标 24.9k。

**基本工作流**：Explore → Propose → Apply → Archive

1. **Explore（探索）**：用户描述需求意图，AI 在现有代码库上下文中理解问题范围。这一步不生成任何文件，纯粹是对话式的需求澄清。

2. **Propose（提案）**：在 `openspec/changes/` 目录下创建一个以变更名命名的独立文件夹，内含 `proposal.md`（变更提案）和 `tasks.md`（任务清单）。每个变更的文件完全隔离在自己的文件夹中，不会与其他变更互相干扰。

3. **Apply（执行）**：通过 `/opsx:apply` 命令，AI 逐条读取 `tasks.md` 中的任务并执行实现。执行完成后，通过自定义的 `openspec-review` 技能进行代码审查和规范一致性检查。

4. **Archive（归档）**：将完成的变更目录整体移入 `openspec/changes/archive/`，Delta Specs（增量规范）合并到主规范目录 `openspec/specs/`。历史变更不再参与日常开发的上下文窗口，有效控制上下文熵增。

**关键特性**：
- 核心概念仅四个：specs、changes、delta specs、artifacts，上手极快
- 变更隔离机制：每次变更独立文件夹，防止不同需求互相干扰
- Delta Specs：只审查变更部分的 diff，而非整个规范文件
- 兼容 20+ AI 工具，通过 MCP 协议注入

**局限**：轻量意味着某些环节的"强制力"不足——例如，OpenSpec 本身没有内置测试覆盖要求，需要用户通过自定义技能（如 `openspec-e2e`）来补强。对于需要严格阶段门验证的大型项目，可能不如 Spec Kit 那样"开箱即用"。

#### Superpowers（Jesse Vincent）[16]

**定位**：一个面向 Claude Code 的强制工程化技能系统（16.7k GitHub 星标）。与其他 SDD 工具不同，Superpowers 不提供自己的规格模板或工作流引擎，而是通过一套**可组合的技能（Skills）**强制 AI 遵循特定的工程实践。它的哲学是：**不给 AI 自由发挥的空间，而是用技能把 AI 的每一步行为都约束住。**

**基本工作流**：Brainstorm → Design → Plan → TDD → Review → Finish

1. **Brainstorm（头脑风暴）**：AI 分析需求，提出多种实现方案并评估优劣。

2. **Design（设计）**：选定方案后，生成设计文档，定义架构决策和数据模型。

3. **Plan（计划）**：将设计分解为具体任务，每个任务包含实现细节和验收标准。

4. **TDD（测试驱动实现）**：这是 Superpowers 的核心差异化——强制要求先写测试再写实现。AI 必须通过所有测试后才能进入下一步。

5. **Review（代码审查）**：内置代码审查技能，自动检查代码质量、安全漏洞和规范一致性。

6. **Finish（收尾）**：确认所有任务完成，更新文档。

**关键特性**：
- SubAgent 并行执行：复杂任务可分派给多个子代理，每个子代理在独立上下文中工作
- 强制 TDD：测试不是可选的"最佳实践"，而是流程中的硬性门槛
- 技能可组合：用户可以编写自定义技能来扩展或覆盖默认行为

**局限**：学习曲线陡峭（技能系统本身就是一个需要理解的抽象层），且目前主要支持 Claude Code 生态，对其他 AI 工具的兼容性不如 OpenSpec 和 Spec Kit。

#### Kiro（AWS）[18]

**定位**：AWS 推出的 AI 编码 IDE，基于 Code OSS（VS Code 的开源基础）深度定制。与前三款"嵌入现有工具"的框架不同，Kiro 本身就是一个完整的开发环境。其核心理念是**将规格驱动开发内建到 IDE 的每一个环节**，让开发者无需额外安装任何插件即可使用 SDD 工作流。AWS CEO Matt Garman 在 re:Invent 2025 上宣称，Kiro 能够"独立弄清楚如何完成工作"[27]。

**基本工作流**：Prompt → Specs → Task Plan → Implement → Tests

Kiro 提供两种工作模式：

1. **Vibe Mode（即兴模式）**：类似传统的"对话式 AI 编码"，适合快速原型和探索性开发。不强制走规格流程。

2. **Spec Mode（规格模式）**：严格遵循规格驱动工作流——

   - **Requirements（需求）**：将用户的自然语言提示转化为结构化的需求文档，每个需求以 User Story（"As a... I want to..."）格式编写，附带 GIVEN/WHEN/THEN 格式的验收标准。
   - **Design（设计）**：基于需求生成设计文档，定义架构方案、接口设计和边界条件。
   - **Task Plan（任务计划）**：创建带依赖关系的任务序列，每个任务包含实现细节和成功标准。用户可以逐个审批、重定向或让 AI 自主执行。
   - **Implement（实现）**：按任务序列逐步生成代码和测试。Kiro 支持长时间自主运行（"code for days"），但这也带来了风险（见下文）。

**关键特性**：
- **Agent Hooks（代理钩子）**：事件驱动的自动化机制，例如保存文件时自动重新生成测试、提交代码时自动安全扫描、API 变更时自动更新文档。配置文件为 `.kiro/hooks.yaml`。
- **Steering Files（引导文件）**：类似 CLAUDE.md 的项目级约束文件（`.kiro/steering.md`），定义代码规范、架构约束、测试要求等。支持全局和项目两级配置。
- **跨会话持久化**：规格和上下文在不同会话之间保持，不会因为关闭 IDE 而丢失。
- **深度 AWS 集成**：原生支持 SageMaker、Lambda、Aurora DSQL 等 AWS 服务。

**局限与警示**：2025 年 12 月，Kiro 因一次事故引发行业广泛关注——在获得"修复系统"权限后，AI 自主决定删除并重建整个环境，导致 AWS Cost Explorer 在中国区域中断 13 小时 [33]。这个事件揭示了自主 AI 编码的深层风险：**规格驱动并不意味着安全**。即使有结构化的规格，AI 的实现路径选择仍可能产生灾难性后果。Martin Fowler 也指出，Kiro 目前更接近 Spec-First 层级，其规格的长期维护策略尚不明确 [34]。

### 2.3 横向对比

基于上述介绍，将四款工具的关键维度对比如下：

| 维度 | Spec Kit（GitHub）[15] | OpenSpec（Fission AI）[17] | Superpowers（Jesse Vincent）[16] | Kiro（AWS）[18] |
|------|------|------|------|------|
| 产品形态 | CLI 工具集 | MCP 协议服务 | Claude Code 技能系统 | VS Code 分支（IDE） |
| 核心理念 | 规格即产品，代码服务于规格 | 棕地优先，"流动而非刚性" | 技能驱动，强制工程化流程 | 提示词→规格→任务计划→实现 |
| 工作流 | Constitution → Specify → Plan → Implement | Explore → Propose → Apply → Archive | Brainstorm → Design → Plan → TDD → Review → Finish | Prompt → Specs → Task Plan → Implement → Tests |
| 执行深度 | 委托式：由连接的 AI Agent 执行 | 轻量委托：/opsx:apply 读任务清单执行 | 深度编排：SubAgent 并行执行 + TDD | 自主编排：按任务序列逐个实现，支持长时间自主运行 |
| 上下文治理 | 结构化制品传递上下文 | 变更隔离：每次变更独立文件夹 | 技能系统约束行为 | Steering Files + 跨会话持久化 |
| 独特能力 | Constitution 组织级约束 | Delta Specs 增量规范 | 可组合技能覆盖全流程 | Agent Hooks 事件驱动自动化 |
| 兼容平台 | 18+ AI 工具 | 20+ AI 工具 | Claude Code/Codex/OpenCode 等 | 独立 IDE（基于 Code OSS），深度集成 AWS |
| 适用场景 | 新项目、全流程规范化 | 存量项目迭代、中小团队快速落地 | 复杂工程、需要强纪律约束 | AWS 生态、需要 IDE 集成的团队 |
| 学习曲线 | 中高（产物丰富但命令集多） | 低（核心概念仅四个） | 高（技能系统复杂） | 低（双模式：Vibe / Spec） |

**选型建议**：

- **快速上手 / 存量项目**：OpenSpec。核心概念只有四个，五分钟内可以理解并开始使用，变更隔离机制天然适合已有代码库的增量开发。
- **全流程标准化 / 新项目**：Spec Kit。GitHub 官方背书，Constitution + 结构化产物提供组织级技术治理能力，适合需要严格规范化的大型项目。
- **强工程纪律 / 复杂系统**：Superpowers。强制 TDD + SubAgent 并行执行 + 代码审查，适合对质量有极端要求的场景。
- **IDE 集成 / AWS 生态**：Kiro。下载安装即可使用，双模式切换灵活，Agent Hooks 自动化能力突出。但需注意其生态相对封闭，且自主运行模式存在安全风险。

值得注意的是，这些工具并非互斥。国内开发者社区已有将 OpenSpec 与 Superpowers 整合的实践 [28]——用 OpenSpec 管理规格变更，用 Superpowers 的技能系统补强测试和审查能力。Martin Fowler 也观察到，这些工具虽然都自称 SDD，但"差异相当大"，在选择时不应将它们视为同质化的替代品 [34]。

### 2.4 国内工程实践

与海外工具生态相比，国内开发者的 SDD 实践有自己的特色：多模型协同 + 中文工程规范。

**实践一：多 AI 协同的 SDD 全流程交付（阿里云开发者社区）**

一篇发表在阿里云开发者社区的实践文章 [29]，记录了基于 SDD 范式构建 Claude + Codex + Gemini 多 AI 协同工作流，并成功从零交付一个跨境保险产品的全过程。该实践有几个关键设计：

- **工具选型**：选择 OpenSpec 作为 SDD 框架，原因包括轻量可嵌入（CLI 注入即可启用）、多 AI 友好（原生支持多种 AI 工具，Claude 服务异常时可切换模型顶上）、变更可管理（changes/ 提案机制让每次修改可评审、可归档）。
- **模型分工**：Claude 作为协调者（理解需求、分配任务、最终决策），Codex 作为高级工程师（复杂代码实现、调试、重构），Gemini 作为长文本分析师（大量文档分析、全局视图、模式发现）。三者通过 MCP 协议注入 Claude，Claude 按全局规则自动判断何时调用哪个模型。
- **工程规范**：通过 CLAUDE.md 定义强制性全局工具规则——"对于任何非平凡任务，必须自问：Codex 能帮忙吗？Gemini 能帮忙吗？"工具调用是默认行为，而非可选项。

这个实践的核心洞察是：**在多模型协作场景中，统一的 SDD 规格成为共享语境与约束边界。各模型基于同一份 Spec 工作，避免因理解偏差导致的返工或冲突。**

**实践二：OpenSpec 工程化落地的深度踩坑（johng.cn）**

一位国内开发者记录了基于 OpenSpec + Claude Code 进行 SDD 实践的完整过程 [30]，并开源了实践项目 [31]。该实践覆盖了从工具配置到工程约束、从上下文治理到迭代熵管理的一系列真实工程问题，是目前能看到的最详细的 OpenSpec 中文实战记录。

几个有价值的工程经验：

- **错误驱动的规范积累**：CLAUDE.md 不需要从零手写，而是 "边用边补"——每当 AI 在开发过程中犯了某种重复性错误，立即将正确约束追加到 CLAUDE.md 中，让后续所有会话自动继承这份经验积累。
- **强制测试验收**：OpenSpec 本身没有内置测试覆盖要求，该实践通过自定义 openspec-e2e 和 openspec-review 技能，将回归测试从 "可选" 变成 "必须"。
- **上下文熵治理**：采用 "小粒度变更 + 及时归档" 策略，避免历史变更产物污染当前上下文窗口，导致模型 "降智"。
- **模型分层使用**：探索阶段使用大上下文高推理模型（如 Claude Opus 4.6 1M），实现阶段使用高性价比模型（如 GLM-5），有效控制成本。

**实践三：SDD 的适用性反思（腾讯云开发者社区）**

一篇在腾讯云开发者社区广泛传播的文章 [32]，以实践者视角介绍了 SDD 在日常工作中的具体应用。文章的核心价值不在于工具推荐，而在于它坦诚地列出了 SDD 的边界：探索性需求（"先做个 MVP 试试水"）、超简单功能（改文案调颜色）、紧急修复（线上 Bug 分秒必争）、创意型工作（UI 设计、算法优化），这些场景下 SDD 不仅无益，反而有害。

该实践给出了一个实用的判断标准：**如果这个任务是可描述、可验证、重复性的，那就适合 SDD。**

**实践四：反对的声音——"谨防 SDD，它不是银弹"**

在 SDD 热度攀升的同时，也有理性的反对声音。微信公众号 "古法编程研究所" 发表文章《谨防 Spec Driven Development 它不是银弹》[33]，提醒开发者不要盲目追捧。这与本文第四章 "没有银弹" 的立场一致——SDD 有明确的适用边界，把它当万能药只会让团队失望。

### 2.5 大厂实践速览

将传统 SDD 方法论与 AI 时代实践综合来看，各大公司的 SDD 应用分布在不同的严谨度层级上：

| 公司 | 方法论 | 场景 | 规格光谱位置 | 核心价值 |
|------|--------|------|------------|----------|
| Amazon | TLA+ | DynamoDB/S3 分布式设计 | Spec-Anchored | 发现深层并发 bug [5] |
| Google | TDD + Protocol Buffers | 全公司工程实践 | Spec-Anchored | 代码质量 + 服务契约标准化 |
| Stripe | API-First + Spec Kit | 支付 API 产品 | Spec-as-Source（API 层） | API 设计标杆 |
| Netflix | Contract Testing + API-First | 微服务架构 | Spec-Anchored | 服务独立部署 |
| Microsoft | TLA+ + Type System | Azure 基础设施 | Spec-Anchored | 基础设施正确性验证 |
| Tesla | Model-Based Development | 自动驾驶控制 | Spec-as-Source | 安全关键系统验证 |
| GitHub | Spec Kit（自研） | AI 编码工具链 | Spec-as-Source | SDD 工具标准化 |
| AWS | Kiro（自研） | AI 编码工具链 | Spec-First → Anchored | 规格驱动的 AI 编码 |
| 国内开发者社区 | OpenSpec + 多模型协同 | 跨境保险等复杂业务 | Spec-First → Anchored | 多 AI 协同可靠交付 [29] |

---

## 三、SDD 的本质

剥开方法论的外衣，SDD 的本质是什么？

### 3.1 规格是认知工具

人脑无法同时处理复杂系统的所有细节。规格是一种**外部化思维的工具**——把"系统应该做什么"从大脑中搬到纸上（或文件中），让思考变得可审视、可讨论、可迭代。

这不是文档的问题，这是**认知容量的问题**。没有规格，团队对"要构建什么"的理解就散落在每个人的大脑里，每次对齐都需要大量沟通。有了规格，这些理解就有了共享的载体。

### 3.2 规格是验证锚点

没有规格，你怎么知道代码写对了？靠感觉？靠手工测试？靠"跑起来没问题"？

规格提供了**客观的验证标准**——不是"我觉得它应该这样"，而是"规格说它应该这样，测试证明它确实这样"。这个锚点让"做对了没有"从主观判断变成了客观事实。

### 3.3 规格是通信协议

不同角色之间的沟通成本是软件工程最大的隐性成本。开发者说"我已经实现了"，产品经理说"这不是我要的"，测试说"这跟需求文档不一致"——三方的分歧往往不是因为谁错了，而是因为**没有共享的规格作为通信协议**。

SDD 提供了这个协议。当规格足够清晰时，"系统应该做什么"不再需要反复讨论——读规格就够了。

### 3.4 规格是知识资产

代码告诉你"系统怎么做"，规格告诉你"系统为什么这么做"。

当团队成员离职、项目交接、系统需要重构时，代码是必要的但不充分的。规格是系统知识的结构化沉淀——它比代码更能表达**意图**。

这也是为什么 AI 时代的 SDD 工具（如 OpenSpec）强调"持久化上下文"——规格与代码同仓库，跨 chat session 持久化。在 AI 编码助手频繁切换上下文的今天，规格是防止知识丢失的最后防线。

### 决定 SDD 实践效果的关键因素

理解了 SDD 的本质，就能理解为什么有些团队的 SDD 实践效果显著，有些则沦为形式主义。

**组织层面：**
- **业务方的参与度**是 SDD 最常见的成败手。如果规格变成开发者的自说自话，它就失去了最重要的价值——确保"做什么"是业务真正需要的。
- **组织对"写规格"的文化认同**决定了 SDD 能否持续。如果把写规格看作"浪费时间"或"走流程"，SDD 就会退化成形式主义。

**方法论层面：**
- **规格粒度**的选择是核心难点。太粗等于没写，太细变成过度设计。最佳粒度取决于项目的不确定性程度。
- **可执行性**决定了规格是否会腐烂。不可执行的规格（纯文档）最容易与实现脱节。

**技术层面：**
- **工具链成熟度**直接影响 SDD 的落地成本。BDD 有 Cucumber，API-First 有 Swagger，形式化方法至今缺乏好用的工具链。
- **与 CI/CD 的集成**决定了规格验证能否成为开发流程的一部分，而不是额外的负担。

**人员层面：**
- **规格编写能力**是一种稀缺的混合技能——需要同时理解业务和技术。
- **抽象能力**是从具体场景中提炼通用规格的关键。

---

## 四、没有银弹：SDD 的已知问题

Fred Brooks 在 1986 年的论文《没有银弹》中指出 [19]，软件工程中不存在任何单一技术能带来数量级的效率提升。SDD 也不例外。

### 4.1 规格与实现的鸿沟

规格描述的是"应该做什么"，但"怎么做"的复杂性往往超出规格的覆盖范围。

一个排序算法的规格可以是"输出有序序列"，但实现中涉及的内存管理、边界条件、性能优化——这些"怎么做"的问题，规格往往无能为力。更不要说非功能性需求：性能、可用性、安全性、可观测性——它们很难用行为规格精确表达。

### 4.2 规格腐烂（Specification Rot）

规格一旦写完就容易过时。代码改了，规格没改；需求变了，规格没变。随着时间推移，规格与实现之间的 Gap 越来越大，最终规格变成了一堆没人信也没人看的废纸。

可执行规格（TDD、BDD、契约测试）能缓解但不能完全解决这个问题。测试可能通过，但测试覆盖的场景可能已经不再是业务真正需要的。

### 4.3 过度规格化（Over-specification）

过早定义细节会限制实现方案的选择。你规定"必须使用 B+ 树索引"，后来发现 Hash 索引更适合——但规格已经锁死了。

敏捷方法论的核心主张是"拥抱变化"，但规格天然倾向于"锁定"。这是一对根本性的张力。

### 4.4 成本与收益的不对称

写好规格需要大量前期投入。在短期项目、探索性项目、小团队中，SDD 的投入产出比可能是负的。一个两周的内部工具项目，花三天写规格，这合理吗？

SDD 的价值在大型、长期、多人协作的项目中最显著。但在这些项目中，写规格的难度也最大——因为系统本身就复杂。

### 4.5 "规格即测试"的幻觉

TDD 的测试覆盖不等于规格覆盖。你只能测试你想到的场景——遗漏的规格等于遗漏的缺陷。

### 4.6 业务理解的局限性

规格只能规格化"已知的需求"。但很多创新性产品在开发过程中才发现真正需求——用户不知道自己要什么，直到你做出来给他们看。

过早规格化可能扼杀探索空间。这不仅是理论上的担忧——很多成功的产品都不是按规格做出来的。

### 4.7 适用边界

| 场景 | SDD 适用度 | 原因 |
|------|-----------|------|
| 安全关键系统 | ★★★★★ | 错误代价极高，形式化验证不可替代 |
| 大型分布式系统 | ★★★★☆ | 接口契约和服务间协作至关重要 |
| 平台/SDK/API 产品 | ★★★★☆ | 契约即产品，规格即文档 |
| 中大型业务系统 | ★★★☆☆ | 视团队能力和文化而定 |
| 探索性/创新性产品 | ★★☆☆☆ | 需求不稳定，过早规格化有害 |
| 小型内部工具 | ★★☆☆☆ | 成本不成比例 |
| 快速原型/MVP | ★☆☆☆☆ | 速度优先，规格是负担 |

---

## 五、AI 时代的 SDD：旧问题与新答案

### 5.1 AI 编码改变了什么？

AI 编码助手改变了"写代码"的成本，但没有改变"写对代码"的难度。

当 AI 能在几秒内生成一个功能模块，瓶颈从"如何实现"转移到了"实现什么是对的"。这个转移让 SDD 的价值发生了质变：

**过去，SDD 是"锦上添花"——有它更好，没它也能干。现在，SDD 正在变成"不可或缺"——没有规格的 AI 编码，就像没有航线的自动驾驶。**

### 5.2 递归问题：谁来保证规格是对的？

这里有一个看似无解的递归问题：

1. AI 写代码 → 谁来保证代码做的是对的事？→ 用规格验证。
2. 规格也是 AI 生成的 → 谁来保证规格是对的？→ 人工审查。
3. 人工审查规格 → 是否吞噬了 AI 带来的效率优势？

这不是一个纯理论问题。如果一个团队引入 SDD，让 AI 生成规格、AI 写代码、然后人工审查规格和代码——这个流程的效率可能还不如直接让人来写。

但有解。

**关键思路一：规格可执行化。** 如果规格本身就是可运行的测试或断言，那么规格的正确性可以通过运行来验证。BDD 场景、契约测试、属性测试——它们都是"可执行的规格"。规格不是被"阅读"的，而是被"运行"的。

**关键思路二：分层验证。** 不需要人工一次性审查所有规格。可以分四层：
- 第一层：AI 自动验证规格的自洽性（逻辑矛盾、遗漏场景）
- 第二层：自动化测试验证代码符合规格
- 第三层：集成测试和端到端测试验证行为符合预期
- 第四层：人工只在关键节点介入（验收标准、非功能性需求、业务规则）

**关键思路三：增量演进。** 不追求一开始就写出完美的规格。OpenSpec 的 Spec Delta 模式是一个很好的实践——每次变更只附带规格变更的 diff，人工只需 review delta，不需要每次都读完整规格。这类似于代码 review：你不会每次从头审查整个代码库，只看 diff。规格 review 也应该如此。

**关键思路四：规格生成 + 规格验证的闭环。** AI 生成规格 → AI 自动验证规格（自洽性、一致性、完整性）→ 发现问题 → 自动修正 → 再验证。人工只需要在最终验收时介入。这类似于编译器的 lint + type check——大部分问题机器自己能发现，人只需要处理机器处理不了的那一小部分。

### 5.3 真正的风险：忠实实现错误的规格

AI 写错代码是容易发现的问题——测试会失败。但 AI 忠实地实现了一个错误的规格——测试会全部通过，但做出来的东西是错的。

这才是 AI 时代 SDD 面临的最核心挑战。它不再是"代码是否符合规格"的问题，而是"规格是否符合业务意图"的问题。

这意味着 SDD 在 AI 时代的重心需要上移：

- **传统 SDD** 的重心在"规格 → 代码"的映射——确保代码正确实现了规格。
- **AI 时代的 SDD** 的重心在"业务意图 → 规格"的映射——确保规格正确表达了业务意图。

后者的难度远大于前者。因为"业务意图"往往是模糊的、隐含的、随着时间演变的。将模糊的意图转化为精确的规格，这本身就是一种稀缺的能力。

### 5.4 SDD 作为 AI 编码的"编译器"

可以做一个类比：高级语言需要编译器来保证代码的语法和类型正确性。AI 生成的代码需要规格作为"编译目标"来保证它做的是对的事。

没有规格的 AI 编码，就像没有类型系统的编程——能跑，但你不知道对不对。类型系统不能保证程序做的是对的（类型正确的程序仍然可能有逻辑错误），但它能在编译期排除一大类错误。规格也一样——它不能保证系统做的是对的，但它能排除"代码做了规格没说的事"和"代码没做规格说的事"这两类最基本的错误。

### 5.5 Vibe Coding 与 Spec-Driven 的张力

Kiro 的口号"Go from vibe coding to viable code" [18] 精准地捕捉了一个张力。

Vibe coding——凭感觉让 AI 写代码——在原型探索、快速验证想法、学习新技术时有其价值。它的优势是速度和灵活性。但如果把它用于生产系统，结果往往是不可预测的。

Spec-driven coding 通过规格约束 AI 的行为，提供可预测性和可重复性。但如果在探索阶段就要求严格的规格，可能会扼杀创新。

**什么时候该 vibe，什么时候该 spec？** 这不是一个非此即彼的选择，而是一个光谱：

- **纯探索阶段**（验证想法是否可行）→ Vibe coding 为主
- **原型阶段**（验证方案是否可行）→ Vibe coding + 轻量规格
- **开发阶段**（构建生产系统）→ Spec-driven 为主
- **维护阶段**（演进现有系统）→ Spec-driven + 增量演进

关键是在合适的时机从 vibe 切换到 spec，而不是一开始就规格化一切。

### 5.6 规格的粒度悖论

AI 编码时代还带来了一个新问题：**规格的粒度如何选择？**

规格太粗，AI 会自由发挥，导致输出不可控。规格太细，等于你用自然语言重写了一遍代码——AI 的价值在哪里？

这不是一个有标准答案的问题。但在实践中，有几个原则可以参考：

- **聚焦行为，不聚焦实现**。好的规格说"用户点击购买按钮后，库存应当减少 1"，而不是说"调用 InventoryService.decrement() 方法"。
- **聚焦约束，不聚焦自由度**。好的规格定义"必须做什么"和"不能做什么"，把"怎么做"的自由度留给 AI。
- **聚焦不变量，不聚焦步骤**。好的规格定义系统的状态约束（"账户余额不能为负"），而不是实现步骤（"先查询余额，再扣减，再更新"）。

### 5.7 人在 SDD 中的角色演变

在传统 SDD 中，人的角色是"规格的编写者和验证者"。在 AI 时代，这个角色正在发生变化：

- **不再是规格的主要编写者**——AI 可以根据需求描述生成初版规格。
- **更像是规格的审查者和决策者**——判断 AI 生成的规格是否准确反映了业务意图，在关键节点做出决策。
- **最终的业务意图定义者**——这是 AI 无法替代的角色。AI 不知道"为什么要做这个功能"，只有人知道。

这实际上提高了对人的要求：你需要从"写规格的人"变成"审查和决策规格的人"。前者需要的是表达能力，后者需要的是判断力和业务理解力。

---

## 结语

从 1960 年代的形式化规格到 2026 年的 AI 原生 SDD 工具，SDD 的发展是一部不断在"精确性"和"实用性"之间寻找平衡的历史。

每一次浪潮都试图回答同一个问题：**如何让"做什么"比"怎么做"更清晰？** 每一次浪潮都比上一次走得更远——从数学语言到编程语言，从编程语言到自然语言，从自然语言到 AI 可理解的上下文。

但核心从未改变：**在动手实现之前，先用精确的语言定义"系统应该做什么"。**

AI 编码没有让这个原则过时，反而让它变得更加紧迫。当"写代码"变得廉价，"定义正确性"就成了稀缺能力。SDD——无论它以什么形态出现——都是构建这种能力的框架。

它不是银弹。它有过适用边界，有成本，有学习曲线。但在 AI 编码助手日益普及的今天，**没有规格的 AI 编码，比没有规格的人工编码更危险**——因为错误积累的速度更快了。

SDD 在 AI 时代的真正价值，不在于让 AI 写出更好的代码，而在于**让团队更清楚地知道自己要 AI 写什么样的代码**。

这，才是规格驱动开发六十年来不变的本质。

---

## 参考来源

[1] Z Notation. Wikipedia. https://en.wikipedia.org/wiki/Z_notation

[2] VDM (Vienna Development Method). Wikipedia. https://en.wikipedia.org/wiki/Vienna_Development_Method

[3] TLA+. https://lamport.azurewebsites.net/tla/tla.html

[4] B-Method. Wikipedia. https://en.wikipedia.org/wiki/B-Method

[5] Newcombe, C. et al. "Formal Methods in Practice at Amazon Web Services." In: Abstract State Machines, Alloy, B, TLA, VDM, and Z (ABZ 2014). Springer, 2014. https://lamport.azurewebsites.net/tla/formal-methods-amazon.pdf

[6] Meyer, B. "Object-Oriented Software Construction." Prentice Hall, 1988 (2nd ed. 1997). Design by Contract 官方介绍: https://www.eiffel.com/values/design-by-contract/introduction

[7] Beck, K. "Test-Driven Development: By Example." Addison-Wesley, 2002.

[8] North, D. "Introducing BDD." Better Software Magazine, 2006. https://dannorth.net/introducing-bdd/

[9] Cucumber. 官方网站: https://cucumber.io

[10] Adzic, G. "Specification by Example." Manning Publications, 2011. 官方网站: https://specificationbyexample.com

[11] OpenAPI Specification. https://spec.openapis.org/oas/latest.html

[12] Pact. Consumer-Driven Contract Testing. 官方网站: https://pact.io

[13] Protocol Buffers. Google 开源项目. https://protobuf.dev

[14] Apache Thrift. https://thrift.apache.org

[15] Spec Kit. GitHub 官方仓库: https://github.com/github/spec-kit

[16] Superpowers. GitHub 仓库: https://github.com/obra/superpowers

[17] OpenSpec. GitHub 仓库: https://github.com/Fission-AI/OpenSpec | 官方网站: https://openspec.pro

[18] Kiro. AWS AI 编码工具. 官方网站: https://kiro.dev

[19] Brooks, F. "No Silver Bullet: Essence and Accidents of Software Engineering." IEEE Computer, 1986. https://writings.stephenwolfram.com/2016/09/no-silver-bullet/

[20] Claessen, K. & Hughes, J. "QuickCheck: A Lightweight Tool for Random Testing of Haskell Programs." In: Proceedings of the ACM SIGPLAN International Conference on Functional Programming (ICFP), 2000.

[21] Simulink. MathWorks. 用于基于模型的设计. https://www.mathworks.com/products/simulink.html

[22] Spillwave. "Agentic Coding: GSD vs Spec Kit vs OpenSpec vs Taskmaster AI — Where SDD Tools Diverge." 2026-02. https://pub.spillwave.com/agentic-coding-gsd-vs-spec-kit-vs-openspec-vs-taskmaster-ai-where-sdd-tools-diverge-0414dcb97e46

[23] SDD Survey Paper. "Specification-Driven Development with AI Coding Agents: A Survey." arXiv:2602.00180v1, 2026-02. https://arxiv.org/abs/2602.00180

[24] ThoughtWorks Technology Radar. "Spec-Driven Development." 2025. https://www.thoughtworks.com/radar

[25] GSD (Get Shit Done). 社区 SDD 工具. GitHub: https://github.com/gsd-build/get-shit-done

[26] OpenSpec + Superpowers 整合实践. 国内开发者社区讨论, 2026. 引用自 Spillwave 对比文章 [22] 的延伸分析.

[27] Amazon Web Services. "AWS CEO promises Kiro can independently figure out how to get work done." re:Invent 2025.

[28] johng.cn. "OpenSpec 工程实践：SDD 在真实项目中的落地." 2026. https://johng.cn/ai/openspec-engineering-practice-sdd-in-action/

[29] 阿里云开发者社区. "多 AI 协同 SDD 全流程交付跨境保险产品." 2026. https://developer.aliyun.com/article/1708977

[30] johng.cn. OpenSpec Practice 开源项目. GitHub: https://github.com/gqcn/openspec-practice

[31] 腾讯云开发者社区. "规范驱动开发（SDD）实践指南." 2026. https://cloud.tencent.com/developer/article/2648323

[32] 微信公众号 "古法编程研究所". 《谨防 Spec Driven Development 它不是银弹》. 2026. https://mp.weixin.qq.com/s/GsgPv0zivy8KbiJ0WsAK4w

[33] heyuan110.com. "Kiro Review 2026: Amazon's Spec-Driven AI Agent That Codes for Days." 2026-03. https://www.heyuan110.com/posts/ai/2026-03-10-kiro-review/

[34] Martin Fowler. "Understanding Spec-Driven-Development: Kiro, spec-kit, and Tessl." 2026. https://martinfowler.com/articles/exploring-gen-ai/sdd-3-tools.html

---

*本文写于 2026 年 4 月。SDD 的方法论和工具生态仍在快速演进中，文中观点基于截至写作时的实践和观察。*
