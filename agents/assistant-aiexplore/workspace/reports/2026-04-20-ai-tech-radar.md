# 🔭 AI Tech Radar — 2026-04-20

> 周期：2026-04-06 ~ 2026-04-20 | 信息源：Tavily Search + GitHub + 官方博客

---

## 一、国外大厂 AI 技术动作

### Anthropic

- **Claude Opus 4.7 更新版发布（4/16）** — 推理能力和指令遵循显著提升
- **Claude 系列传闻："Mythos" 模型** — 据报道因"过于危险"而限制发布的实验性模型，引发业界广泛讨论
- **Anthropic 企业战略加速** — B2B 赛道加大投入，Managed Agent 服务扩展，多 Agent 编排方案持续推进
- 🔗 https://www.anthropic.com

### OpenAI

- **Codex 升级为"超级应用"** — 不再只是代码生成工具，正演变为综合 AI 平台，支持后台计算机操作（Computer Use）
- **GPT-5 持续迭代** — 推理能力增强，多模态整合加速
- **融资与估值** — $122B+ 估值，持续大规模融资
- 🔗 https://openai.com

### Google DeepMind

- **Gemini 2.5 系列持续更新** — Pro 和 Flash 版本均有迭代
- **AI Agent 工具生态** — Mariner 项目（浏览器自动化 Agent）持续推进
- **Google I/O 2026 预热** — 预计将有重大 AI 发布
- 🔗 https://blog.google

### Meta AI

- **Llama 4 系列开放权重** — 多模态融合、长上下文支持
- 开源生态持续扩大，社区微调方案涌现

### xAI

- **Grok 模型迭代** — 集成到 X 平台，实时信息获取能力突出

### Mistral AI

- **Mistral Medium 3 发布** — 性价比路线，对标 GPT-4o-mini
- 欧洲 AI 独角兽地位稳固，企业客户增长

---

## 二、技术博主 / 思想领袖动态

### Simon Willison（@simonw）

- 持续高频输出 LLM 工具链评测和分析
- 关注 AI Agent 安全性和可观测性
- 🔗 https://simonwillison.net

### Andrej Karpathy

- **"Software 2.0 到 Software 3.0" 系列思考** — 从神经网络编程到 AI Agent 编程的范式转移
- 多次公开演讲探讨 AI 原生应用开发
- 🔗 https://karpathy.ai

### swyx（Latent Space）

- AI Agent 基础设施系列深度分析
- 关注"Agent 时代的 DevOps"
- 🔗 https://latent.space

---

## 三、AI 工程实践（核心关注）

### Agentic RAG — 新范式

传统 RAG（检索→生成）正在被 **Agentic RAG** 取代：
- Agent 自主规划多步查询、选择工具、迭代优化答案
- 金融研究 Agent：拉取财报 → 情感分析 → 生成投资摘要
- 医疗助手：查询临床指南 → 药物交互检查 → 患者建议
- 核心区别：**自主决策层**，而非固定的 retrieve-then-generate 流程

### MCP（Model Context Protocol）— 生态爆发

MCP 在 2026 年已从实验性协议变成**事实标准**：
- **AWS 官方支持** — 在 Amazon ECS 上部署 MCP Server 的生产级方案（4/14 发布）
- **企业治理框架成形** — GitHub/Microsoft（注册+身份）、AWS（策略深度）、GitLab（会话级审批）、Atlassian（SaaS 边界）
- **OWASP MCP Top 10 安全指南** — Microsoft 联合发布
- **开发工具全面接入** — Claude Desktop、VS Code Copilot、Cursor、Windsurf、ChatGPT 均支持
- 🔗 https://aws.amazon.com/blogs/containers/deploying-model-context-protocol-mcp-servers-on-amazon-ecs/

### Context Engineering / Memory Engineering

- Anthropic 的 Progressive Disclosure 模式（从 RAG → Grep → Skill-based 索引）成为行业参考
- 短期记忆 + 长期记忆的分层架构在 Agent 应用中广泛采用
- Memory 工程从"锦上添花"变成"必选项"

---

## 四、Java AI 框架生态

### Spring AI

- 持续快速迭代，对齐 OpenAI/Anthropic API 规范
- MCP 支持正在集成中

### LangChain4j

- 社区活跃度持续上升
- 对标 LangChain (Python) 的 Java 生态方案

### 评价

Java AI 框架整体仍处于**追赶阶段**，Python 生态领先 6-12 个月。但企业级 Java 项目需求旺盛，Spring AI 和 LangChain4j 是主要受益者。

---

## 五、GitHub 热门 AI 项目（本周/本月）

| 项目 | Stars | 方向 | 亮点 |
|------|-------|------|------|
| hermes-agent | 85k+ | SuperAgent 平台 | $5 VPS 可运行，也支持 GPU 集群 |
| claude-mem | 56k+ | Claude 记忆工程 | 长期记忆 + 上下文管理 |
| OpenAgentTrek | 61.6k | 长时序 Agent（字节） | 沙箱/记忆/工具/skill/subagent |
| Trae Agent | 11.3k | SWE Agent（字节） | 多 LLM 支持，研究友好 |
| Qwen3.5 | 热门 | 开源 LLM（阿里） | 0.8B-397B 全尺寸覆盖 |
| Qwen-Agent | 16k | Agent 框架（阿里） | MCP + RAG + Code Interpreter |

> 🔗 https://github.com/trending?since=weekly

---

## 六、AI 开发工具动态

### Cursor
- **Cursor 3 持续更新** — 第三代 AI 编程体验
- Agent 模式成熟，自动文件编辑 + 终端操作

### Claude Code
- Anthropic 官方 CLI Agent
- 重构为 Agentic 架构，多步推理 + 工具调用

### OpenAI Codex
- 向"超级应用"转型 — 后台计算机操作、多任务编排
- 不再局限于代码生成

### Trae（字节跳动）
- AI IDE，集成豆包模型
- 国际版 + 国内版双线发展

### Windsurf
- AI 编程 IDE，Agent-first 设计
- 与 Cursor 竞争激烈

### Augment
- 企业级 AI 编程助手
- 代码库级别理解能力突出

---

## 七、行业新闻 / 资本动态

### 融资与并购
- **OpenAI** — $122B+ 估值，持续大规模融资
- **Anthropic** — B2B 赛道估值持续攀升
- **Mistral** — 欧洲最大 AI 独角兽

### 监管动态
- **EU AI Act** 进入执行阶段，企业合规压力增大
- **美国** AI 监管框架仍在讨论中
- **中国** 生成式 AI 管理办法持续完善

### 行业趋势
- AI Agent 从"玩具"到"生产工具"的拐点已至
- 企业 AI 预算从实验性转向规模化
- MCP 成为连接 AI Agent 与企业系统的标准协议

---

## 八、国内大厂 AI 动态

### 阿里巴巴

- **Qwen3.5 全系列开源** — 0.8B 到 397B MoE，201 种语言，统一视觉-语言架构
- **Qwen-Agent 框架** — MCP + RAG + Code Interpreter + Chrome 扩展
- **Qwen3-Coder / Qwen3-VL / Qwen3-omni** 多模态矩阵完善
- 🔗 https://github.com/QwenLM

### 字节跳动

- **Trae Agent 开源** — 多 LLM 支持（含豆包），研究友好的 SWE Agent
- **OpenAgentTrek** — 61.6k stars，长时序 SuperAgent harness
- **火山方舟平台** — Seedance 视频生成、Seedream 图像模型、MCP 支持、Coding Plan
- 🔗 https://github.com/bytedance

### 美团

- **LongCat 系列** — Agent Tool Use 开源 SOTA，685 亿参数 MoE
- **KuiTest** — 基于大模型的 UI 测试，异常召回率 86%
- 技术博客更新偏慢（最近 3/20），信息不透明
- 🔗 https://tech.meituan.com

### 国内趋势观察
- 三巨头都在押注 Agent 框架，与国外形成对标
- 信息不透明问题依然突出：ATA 内网、微信公众号、JS 渲染平台
- 多模态原生成为标配（Qwen3-omni、Seedance/Seedream）
- 国内强在应用落地和工程规模，弱在方法论公开

---

## 🎯 本期趋势总结

1. **MCP 协议爆发** — AWS 官方部署指南、OWASP 安全 Top 10、全 IDE 接入，从实验到生产的拐点
2. **Agentic RAG 取代传统 RAG** — 自主决策层成为标配，固定的 retrieve-then-generate 模式正在被淘汰
3. **AI 开发工具混战升级** — Codex 转型超级应用、Cursor 3、Claude Code、Windsurf、Augment 全面竞争
4. **Anthropic 安全叙事** — Mythos 模型"太危险"的叙事引发关注，与 OpenAI 的激进路线形成鲜明对比
5. **国内 Agent 竞赛** — 阿里 Qwen-Agent、字节 Trae/OpenAgentTrek、美团 LongCat，三巨头全面押注
6. **Context/Memory 工程主流化** — 从"锦上添花"到"必选项"，Agent 记忆架构成为工程标配

---

*下次执行建议：关注 Google I/O 2026 发布内容，以及 MCP 治理框架的企业落地案例。*
