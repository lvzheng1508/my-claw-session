# SDD 专题文章 — 完整研究笔记（v2）

> 目标：一篇面向技术管理者和开发团队的长文，宏观审视 SDD 的发展、优势、不足，尤其在 AI 开发盛行的今天如何看待 SDD。
> 更新日期：2026-04-07

---

## 一、AI 时代的 SDD 工具/框架（新增）

### 1. Spec Kit（GitHub 官方）
- **来源：** github/spec-kit（GitHub 官方开源项目）
- **定位：** "Toolkit to help you get started with Spec-Driven Development"
- **核心理念：**
  - "Spec-Driven Development flips the script on traditional software development"
  - 规格不再是被丢弃的脚手架，而是**可执行的**——直接生成工作实现
  - 聚焦产品场景和可预测的结果，而非"vibe coding"
- **工作流（五阶段）：**
  1. **Constitution** — 创建项目治理原则和开发准则（/speckit.constitution）
  2. **Specify** — 描述要构建什么，聚焦 what 和 why（/speckit.specify）
  3. **Plan** — 提供技术栈和架构选择（/speckit.plan）
  4. **Tasks** — 从实现计划创建可执行的任务列表（/speckit.tasks）
  5. **Implement** — 按计划执行所有任务（/speckit.implement）
- **特点：**
  - 阶段门控（Phase Gates）严格——必须完成前一阶段才能进入下一阶段
  - 重量级：大量 Markdown 工件、Python 环境依赖
  - 支持扩展系统（Extensions）和预设系统（Presets）
  - 丰富的社区扩展生态（30+ 扩展）
  - 支持 Claude Code、Codex 等主流 AI Agent
- **安装方式：** `uv tool install specify-cli --from git+https://github.com/github/spec-kit.git`

### 2. Superpowers
- **来源：** obra/superpowers（Jesse Vincent 开发，99k+ stars）
- **定位：** "An agentic skills framework & software development workflow for coding agents"
- **核心理念：**
  - 通过可组合的 Skills（技能）让 AI 代理像专业工程师一样工作
  - "让 AI 代理在写代码之前，先理解你要做什么，然后制定计划，最后才执行"
  - 技能是**强制性的**，不是建议——条件匹配时 AI 必须使用
- **核心工作流（六步）：**
  1. **Brainstorming** — 澄清需求，探索方案
  2. **Design Sign-off** — 分块展示设计，等待用户确认
  3. **Implementation Plan** — 拆解为 2-5 分钟的小任务
  4. **TDD + Subagent** — 测试驱动开发，子代理执行
  5. **Code Review** — 每个任务完成后审查
  6. **Finish Branch** — 合并/PR/清理
- **技能系统（20+ Skills）：**
  - 设计类：brainstorming, writing-plans
  - 开发类：subagent-driven-development, test-driven-development, executing-plans
  - 审查类：requesting-code-review, verification-before-completion
  - 调试类：systematic-debugging
  - 协作类：dispatching-parallel-agents, receiving-code-review
  - 元技能：writing-skills, using-superpowers
- **支持平台：** Claude Code, Cursor, Codex, OpenCode, Gemini CLI, Windsurf, Kiro, OpenClaw 等 14+ 款工具
- **特点：**
  - 不要求严格的阶段门控，更灵活
  - 强制 TDD（测试驱动开发）
  - 子代理架构支持并行任务执行
  - 可自定义 Skills 扩展
  - 中文增强版：superpowers-zh

### 3. OpenSpec（Fission AI）
- **来源：** Fission-AI/OpenSpec（开源，MIT 协议）
- **定位：** "A lightweight spec-driven framework for coding agents and CLIs"
- **核心理念：**
  - "Agree before you build" — 先对规格达成共识再写代码
  - 轻量级、迭代式、不要求严格阶段门控
  - 支持 brownfield（存量项目）而不仅仅是 greenfield
- **工作流（新版简化）：**
  1. **Propose** — `/opsx:propose "your idea"` → 创建变更文件夹
     - proposal.md — 为什么做、改什么
     - specs/ — 需求和场景
     - design.md — 技术方案
     - tasks.md — 实现清单
  2. **Apply** — `/opsx:apply` → AI 按规格实现
  3. **Archive** — `/opsx:archive` → 归档已完成的变更
- **扩展工作流：** /opsx:new, /opsx:continue, /opsx:ff, /opsx:verify, /opsx:sync
- **核心特色：**
  - **Spec Delta Review** — 理解需求变更而不需要看代码 diff
  - **持久化上下文** — 规格与代码同仓库，跨 chat session 持久化
  - 20+ AI 编码助手支持
  - 无 API key 要求，无 MCP 依赖
- **哲学：** fluid not rigid, iterative not waterfall, easy not complex
- **vs 其他工具的定位：**
  - vs Spec Kit：更轻量，无 Python 依赖，5 分钟 vs 30 分钟上手
  - vs Kiro：无 IDE 锁定，无模型锁定
  - vs Superpowers：更聚焦于规格管理本身而非完整工作流

### 4. Kiro（AWS）
- **来源：** kiro.dev（AWS 推出的 AI 编码工具）
- **定位：** "Agentic AI development from prototype to production"
- **SDD 特色：**
  - **Executable Specs** — 将规格作为 AI 编码的精确上下文
  - "Go from vibe coding to viable code"
  - Spec + Steering + Smart Context Management 构成核心
  - 支持 Claude Sonnet 4.5 等 AWS 模型
- **局限：** 锁定 AWS IDE 和 Claude 模型

---

## 二、AI 时代 SDD 工具对比

| 维度 | Spec Kit (GitHub) | Superpowers | OpenSpec | Kiro (AWS) |
|------|------------------|-------------|----------|------------|
| **来源** | GitHub 官方 | Jesse Vincent (社区) | Fission AI (社区) | AWS |
| **核心理念** | 规格可执行，阶段门控 | 技能驱动，强制流程 | 轻量规格共识 | 可执行规格 + IDE |
| **阶段门控** | 严格（5阶段） | 半严格（6步但灵活） | 无严格门控 | 适中 |
| **安装复杂度** | 较高（Python/uv） | 中等（Git clone/插件） | 低（npm） | N/A（独立IDE） |
| **上手时间** | ~30分钟 | ~15分钟 | ~5分钟 | N/A |
| **扩展性** | Extensions + Presets 生态 | 自定义 Skills | 基本无 | N/A（封闭） |
| **AI Agent 支持** | Claude Code, Codex | 14+ 工具 | 20+ 工具 | 仅 Kiro |
| **模型锁定** | 否 | 否 | 否 | 是（Claude） |
| **TDD 强制** | 否（可选） | 是（核心原则） | 否 | 否 |
| **社区活跃度** | 高（GitHub 官方背书） | 极高（99k+ stars） | 高 | N/A |
| **Stars** | 较新 | 99k+ | 快速增长 | N/A |

---

## 三、AI 编码质量保障的深层思考

### 核心矛盾：当 AI 代码贡献率接近 100%

**问题链：**
1. AI 写代码 → 谁来保证代码做的是对的事？
2. 规格文档也是 AI 生成 → 谁来保证规格是对的？
3. 人工 check 规格 → 是否吞噬 AI 的效率优势？

**这是一个递归问题：验证者本身也需要被验证。**

### 可能的解决路径

**路径 1：规格可执行化（Executable Specs）**
- 如果规格本身就是可运行的测试/断言，那么规格的正确性可以通过运行来验证
- TDD/BDD/OpenSpec 的 Spec Delta → 自动化验证规格的一致性
- 规格不再是"文档"，而是"可执行的契约"

**路径 2：分层验证（Layered Verification）**
- 第一层：规格自洽性验证（AI 检查规格是否有逻辑矛盾）
- 第二层：规格 vs 代码的一致性验证（自动化测试）
- 第三层：代码 vs 行为的验证（集成测试、端到端测试）
- 第四层：行为 vs 业务意图的验证（人工 check，但频率可以降低）

**路径 3：规格演进而非一次性定义**
- 不追求一开始就写出完美的规格
- 用 Spec Delta（增量规格）的方式逐步演进
- 每次变更都附带规格变更的 diff，人工只需 review delta
- 这降低了人工 check 的成本——不需要看完整规格，只需要看变更

**路径 4：规格生成 + 规格验证的闭环**
- AI 生成规格 → AI 自动验证规格（自洽性、一致性、完整性）
- 发现问题 → 自动修正 → 再验证
- 人工只需要在关键节点（验收标准、非功能性需求）介入
- 这类似于编译器的 lint + type check，大部分问题机器自己能发现

**路径 5：从"验证代码"到"验证行为"**
- 传统 SDD：写规格 → 写代码 → 验证代码符合规格
- AI 时代：写规格 → AI 生成代码 → 验证**行为**是否符合业务意图
- 代码本身变得不那么重要（反正 AI 生成的），**行为验证**成为关键
- 这意味着：端到端测试、属性测试、契约测试的价值在 AI 时代反而**上升**

### 关键洞察

1. **SDD 在 AI 时代的价值不是降低，而是上升了** — 因为当"写代码"变得廉价，"写对代码"的难度并没有降低。SDD 提供了定义"对"的框架。

2. **人工 check 的效率悖论有解** — 通过分层验证 + 可执行规格 + 增量演进，人工只需要在关键节点介入，而不是逐行审查。

3. **SDD 是 AI 编码的"编译器"** — 就像高级语言需要编译器来保证正确性，AI 生成的代码需要规格作为"编译目标"来保证它做的是对的事。

4. **真正的风险不在 AI 写错代码，而在 AI 忠实地实现了错误的规格** — 这才是 SDD 在 AI 时代最核心的价值：确保规格本身是对的。

---

## 四、待确认/后续

- 文章核心观点待用户确认
- 大厂实践案例是否需要补充
- 文章结构可能需要调整
