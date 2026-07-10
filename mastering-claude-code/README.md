# Mastering Claude Code in 30 Minutes

> **课程定位**：来自 "Mastering Claude Code in 30 Minutes" 演讲，系统展示从安装配置到高级工程实践的全链路知识。本笔记将其浓缩为三篇结构化文档，严格按演讲顺序组织，并附加社区最佳实践作为扩充。

---

## 📋 笔记导航与重难点

### [01. 基础篇：快速入门与核心工作流](./01-foundation.md)
- **核心**：
  - Claude Code 安装（`npm`/`curl`）与初始化配置（`/init`、`/terminal-setup`、`/allowed-tools`）
  - 代码库问答——入门第一步，零风险建立信任，入职从数周缩短到 2-3 天
  - 编辑代码与 Plan Mode——"写代码前先做计划"，不需要特殊工具
  - 工具集成与验证循环——给 Claude 一个验证自己产出的方式，质量提升 2-3 倍
- **难点**：
  - Plan Mode 的核心思维：花 token 想清楚再写，而非直接写 3000 行
  - 验证循环的设计思路——让 Claude 看到自己的错误并自动迭代修复
- **扩充**：Plan Mode 五步工作流、Ask/Plan/Auto 三模式切换、上下文管理基础

### [02. 工作流篇：上下文配置与自动化](./02-workflows.md)
- **核心**：
  - `CLAUDE.md` 上下文文件体系——最高杠杆的配置文件，自动加载、嵌套按需
  - `CLAUDE.md` 编写与维护——保持简洁 ≤200 行，每次犯错加一条规则
  - 配置层级体系——项目 → 全局 → 企业策略，权限既能自动审批也能强制屏蔽
  - 配置一次团队共享——提交到 Git，形成网络效应
- **难点**：
  - 配置层级的作用域理解与优先级覆盖
  - 团队共享的网络效应——写一次 N 人受益
- **扩充**：内置工具矩阵、Hooks 事件体系（PostToolUse/Stop）、验证循环进阶、上下文管理进阶

### [03. 高级篇：规模化与扩展](./03-advanced-patterns.md)
- **核心**：
  - 核心快捷键——终端环境下不易发现的高频操作
  - `claude -p` SDK/CLI 脚本化——把 Claude 当成 Unix 管道中的超级工具
  - 并行会话与 Git Worktree——3-5 个本地 + 5-10 个云端会话同时运行
- **难点**：
  - 并行会话的上下文隔离与 Worktree 机制
  - Claude-p 的非交互式特性与管道组合思维
- **扩充**：Sub-agents/Skills/Slash Commands 三级复用体系、MCP 集成、工程哲学与 Loop Engineering

---

## 🔮 延伸学习

**下一步推荐**：
- 深入 [Claude Agent SDK](https://docs.anthropic.com/en/docs/agents-and-tools/agent-sdk) —— 构建自定义 Agent 的编程接口
- 实战 [Claude Code Best Practice](https://github.com/shanraisshan/claude-code-best-practice) 仓库 —— 社区整理的配置模板
- 探索 MCP 生态 —— 将 Claude Code 从代码工具升级为全栈运维平台

---

> 🔗 **返回根目录**：[AI Agent Learning Notes](../README.md)
