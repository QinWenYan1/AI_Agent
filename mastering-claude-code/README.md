# Mastering Claude Code in 30 Minutes

> **课程定位**：Boris Cherny（Claude Code 创始人）在 30 分钟演讲中系统展示了从安装配置到高级工程实践的全链路知识。本笔记将其浓缩为三篇结构化文档，覆盖 Claude Code CLI 的完整使用图谱。

---

## 📋 笔记导航与重难点

### [01. 基础篇：环境配置与核心概念](./01-foundation.md)
- **核心**：
  - Claude Code 安装（`npm`/`curl`）与初始化配置（`/init`、`/allowed-tools`）
  - `CLAUDE.md` 上下文文件体系——Boris 眼中"最高杠杆的配置文件"
  - 代码库问答——从数周入职缩短到 2-3 天
  - 核心快捷键与 Ask → Plan → Auto 三模式切换
- **难点**：
  - `CLAUDE.md` 七层优先级的作用域理解
  - 上下文窗口的 40% 红线与 `/compact` 策略

### [02. 工作流篇：工具链与自动化](./02-workflows.md)
- **核心**：
  - 内置工具矩阵（Bash、File、WebFetch、WebSearch、TODO）
  - Plan Mode 优先原则与 Explore → Plan → Confirm → Code → Commit 五步法
  - Hooks 事件体系——12 种事件驱动的自动化
  - 验证循环——Boris 口中"2-3 倍质量提升"的第一技巧
- **难点**：
  - Hooks 的 `matcher` 正则与事件触发时机
  - 验证循环的设计思路——让 Claude 自己验证自己的产出

### [03. 高级篇：规模化与工程哲学](./03-advanced-patterns.md)
- **核心**：
  - Git Worktree 实现 3-5 个本地会话并行
  - Sub-agents / Skills / Slash Commands 三级复用体系
  - `claude -p` 脚本化与 `--output-format json` 管道组合
  - MCP 协议集成 Slack、Sentry、GitHub 等外部工具
- **难点**：
  - 并行会话的上下文隔离与 TMUX 协调
  - Prompt Engineering → Context Engineering → Loop Engineering 的思维范式升级

---

## 🔮 延伸学习

**下一步推荐**：
- 深入 [Claude Agent SDK](https://docs.anthropic.com/en/docs/agents-and-tools/agent-sdk) —— 构建自定义 Agent 的编程接口
- 阅读 [Boris Cherny 在 Lenny's Podcast 的对谈](https://www.lennyspodcast.com/) —— "What happens after coding is solved"
- 实战 [Claude Code Best Practice](https://github.com/shanraisshan/claude-code-best-practice) 仓库 —— 社区整理的配置模板

---

> 🔗 **返回根目录**：[AI Agent Learning Notes](../README.md)
