# 📘 03. 高级篇：规模化与工程哲学 (Advanced Patterns: Scaling & Engineering Philosophy)

> 来源说明：Boris Cherny "Mastering Claude Code in 30 Minutes" 演讲 + 社区最佳实践 | 本篇涵盖：Git Worktree 并行多会话、Sub-agents/Skills/Commands 三级复用体系、SDK 脚本化、MCP 集成、工程哲学

---

## 🧠 核心概念总览（严格按演讲顺序）

- [*知识点1: Git Worktree 并行多会话*](#id1)
- [*知识点2: Sub-agents 子代理*](#id2)
- [*知识点3: Skills 技能包*](#id3)
- [*知识点4: Slash Commands 自定义命令*](#id4)
- [*知识点5: Claude Code SDK / CLI 脚本化*](#id5)
- [*知识点6: MCP 外部工具集成*](#id6)
- [*知识点7: Boris 的工程哲学*](#id7)
- [*知识点8: 从 Prompt Engineering 到 Loop Engineering*](#id8)

---

<a id="id1"></a>
## ✅ 知识点1: Git Worktree 并行多会话 (Parallel Sessions with Git Worktrees)

**理论**
- Boris 的日常状态：**3-5 个本地终端会话 + 5-10 个云端会话同时运行**
- 每个会话专注一个独立任务（架构设计、功能开发、测试、文档……），互不干扰
- 使用 `git worktree` 实现分支级别的文件隔离——每个 worktree 是一个独立的 Git 工作副本，指向不同分支
- 终端 Tab 1-5 各跑一个会话，开启系统通知，哪个完成了就切过去

**命令/配置示例**
```bash
# 创建一个新的 worktree（在独立分支上工作）
git worktree add .claude/worktrees/feature-auth feature-auth

# 查看所有 worktree
git worktree list

# 删除 worktree（工作完成后）
git worktree remove .claude/worktrees/feature-auth

# 终端多 Tab 布局示例：
# Tab 1: claude → 架构设计
# Tab 2: claude → 功能实现
# Tab 3: claude → 测试编写
# Tab 4: claude → Code Review
# Tab 5: claude → 文档更新
```

**注意点**
- ⚠️ **前提条件**：每个 worktree 需要干净的工作区（无未提交变更）。如果需要切换分支但当前有未提交工作，先 `git stash`
- 💡 **理解技巧**：并行会话的核心价值不是"同时写代码"（人类注意力是单线程的），而是**上下文隔离**——每个 Tab 有独立、干净的对话历史，不会相互污染
- 🔄 **知识关联**：Worktree 隔离 + Sub-agents [知识点2](#id2) 构成两级隔离——进程级（worktree）和上下文级（sub-agent）
- 📋 **术语提醒**：`Worktree(工作树)` — Git 2.5+ 支持的功能，允许同一仓库同时 checkout 多个分支到不同目录

---

<a id="id2"></a>
## ✅ 知识点2: Sub-agents 子代理 (Sub-agents)

**理论**
- Sub-agent 是 Claude Code 的**上下文隔离机制**——把一个子任务派发给独立的 Claude 实例，它有自己干净的上下文窗口，完成后只把**结论**（而非完整过程）传回主会话
- 定义在 `.claude/agents/*.md`，通过 YAML 前端元数据配置模型、工具权限、系统提示
- 截至 v2.1.172+，子代理最多可嵌套 **5 层**
- Boris 的用法：代码审查交给 `code-reviewer` 子代理，验证交给 `verify-app` 子代理

**命令/配置示例**
```markdown
<!-- .claude/agents/code-reviewer.md -->
---
name: code-reviewer
description: 审查代码变更，查找 bug 和简化机会
tools: Read, Bash, WebSearch
model: opus
---

你是一个 Code Reviewer。仔细检查 diff，报告：
1. 潜在的 bug
2. 可简化的逻辑
3. 性能问题
```

**注意点**
- 💡 **理解技巧**：子代理的使用时机 = "这个子任务如果放在主会话里，会塞爆上下文"。原则：独立子问题 → 独立子代理
- ⚠️ **关键警告**：子代理只返回**结论**。如果你的子代理 prompt 不够精确，结论可能缺少关键细节。把子代理的输出格式要求写清楚
- 🔄 **知识关联**：Sub-agents、Skills [知识点3](#id3)、Commands [知识点4](#id4) 构成三级复用体系——粒度从细到粗

---

<a id="id3"></a>
## ✅ 知识点3: Skills 技能包 (Skills)

**理论**
- Skills 是**可复用的打包工作流**，存储在 `.claude/skills/<name>/SKILL.md`
- 与 Sub-agents 的区别：Skills 的指令直接注入当前上下文（不新建实例），适合**过程性知识**
- 支持渐进式披露（Progressive Disclosure）：SKILL.md 主体 → `references/` 详细文档 → `scripts/` 辅助脚本，只在被调用时才加载
- Boris 的原则：**"如果你一天内做某件事超过一次，就把它变成 Skill 或 Command"**

**命令/配置示例**
```markdown
<!-- .claude/skills/commit-push-pr/SKILL.md -->
---
name: commit-push-pr
description: 提交代码、推送并创建 PR
---

## 流程
1. 运行 `git diff --stat` 检查变更
2. 运行 `npm test` 确保测试通过
3. 生成 conventional commit message
4. `git commit` + `git push`
5. 用 `gh pr create` 创建 PR，标题和描述自动生成
```

**注意点**
- 💡 **理解技巧**：Skill vs Command——两者本质相同（都是 Markdown 文件），但 Skill 可以有子目录（`references/`、`scripts/`）支持渐进式加载，Command 更轻量
- 🔄 **知识关联**：Skills 的三层结构（主指令 → 详细参考 → 脚本）是"渐进式披露"设计模式的实践
- 📋 **术语提醒**：`Progressive Disclosure(渐进式披露)` — 先加载概要，需要时再加载详细内容，节省上下文 token

---

<a id="id4"></a>
## ✅ 知识点4: Slash Commands 自定义命令 (Custom Slash Commands)

**理论**
- Slash Commands 是**最轻量级的复用单元**——一个 `.md` 文件放在 `.claude/commands/` 下，就变成了 `/command-name`
- 文件内容就是 prompt 模板，支持 `$ARGUMENTS` 占位符接收参数
- 适合重复性操作：`/commit-push-pr`、`/techdebt`、`/bump-version`
- Boris 的团队把几乎所有日常操作都变成了 slash commands

**命令/配置示例**
```markdown
<!-- .claude/commands/release.md -->
按照以下步骤发布新版本：
1. 运行 npm test && npm run build
2. 从 CHANGELOG.md 生成 release notes
3. npm version $ARGUMENTS
4. git push origin --tags
5. 创建 GitHub Release

版本号：$ARGUMENTS
```

在会话中使用：`/release patch` → `$ARGUMENTS` = `patch`

**注意点**
- 💡 **理解技巧**：三级复用体系的选型决策——

| 场景 | 用哪个 |
|------|--------|
| 独立子任务，需要干净上下文 | Sub-agent |
| 过程性工作流，可复用步骤 | Skill |
| 简单 prompt 快捷方式 | Slash Command |

- 🔄 **知识关联**：Commands 是最简单的复用方式——一行 Markdown = 一个命令，不需要学习 YAML 配置

---

<a id="id5"></a>
## ✅ 知识点5: Claude Code SDK / CLI 脚本化 (Scripting & SDK)

**理论**
- Claude Code 支持**非交互式 CLI 调用**，可以嵌入 Shell 脚本、CI/CD 管道
- 核心参数：
  - `claude -p "prompt"` — 单次问答，输出到 stdout
  - `--output-format json` — JSON 格式输出，方便程序解析
  - `--allowedTools` — 限制可用工具，如 `Bash(git log:*)`
- **Unix 管道组合**：`claude -p` 的输出可以 pipe 给其他命令，像传统 Unix 工具一样编排

**命令/配置示例**
```bash
# 基础用法：单次问答
claude -p "explain what this repository does" --output-format json

# 限制工具：只允许 git log
claude -p "summarize the last 10 commits" --allowedTools "Bash(git log:*)"

# Unix 管道组合
claude -p "review this code for bugs" --output-format json | jq '.findings[]'

# CI/CD 集成——GitHub Actions 示例
- name: Claude Code Review
  run: |
    claude -p "review the PR diff. Report only critical bugs." \
      --allowedTools "Read(*)" \
      --output-format json > review.json
```

**注意点**
- 💡 **理解技巧**：`claude -p` 是非交互式的——没有 Plan Mode、没有模式切换。适合"输入 → 输出"的管道式任务，不适合交互式探索
- 🔄 **知识关联**：SDK 模式下，验证循环（[02-workflows.md#id7](./02-workflows.md#id7)）需要你自己在脚本中实现——`claude -p "fix"` → 跑测试 → `claude -p "fix tests"` → …
- 📋 **术语提醒**：`Non-interactive(非交互式)` — 程序直接调用，没有人在中间审查每一步

---

<a id="id6"></a>
## ✅ 知识点6: MCP 外部工具集成 (MCP Integration)

**理论**
- MCP（`Model Context Protocol`）允许 Claude Code 连接外部服务和数据源
- 配置在 `.mcp.json` 或 `.claude/settings.json` 的 `mcpServers` 字段
- Claude Code 启动时加载 MCP Server 的工具列表。但 MCP 的全量工具描述可能消耗 50K+ token——**新版的 Tool Search 功能通过按需发现减少了 85% 的 token 消耗**
- Boris 团队常用的 MCP 集成：

| MCP Server | 能力 | 典型场景 |
|------------|------|----------|
| **Slack** | 读写消息 | 把 Slack 线程喂给 Claude → "fix this bug" |
| **GitHub** | Issues/PRs/Repo | 自动 triage Issues |
| **Sentry** | 错误追踪 | 读 Sentry 错误 → Claude 分析 → 修复 |
| **BigQuery** | 数据分析 | SQL 查询（Boris 说有位同事 6 个月没手写 SQL 了） |
| **Puppeteer** | 浏览器操控 | 截图验证 UI |

**命令/配置示例**
```json
// .mcp.json
{
  "mcpServers": {
    "slack": {
      "type": "http",
      "url": "https://slack.mcp.anthropic.com/mcp"
    },
    "github": {
      "type": "http",
      "url": "https://github.mcp.anthropic.com/mcp"
    }
  }
}
```

**注意点**
- ⚠️ **Token 消耗陷阱**：MCP Server 的工具描述会占用上下文 token——不要同时挂载太多不用的 MCP Server。开启 Tool Search 减少浪费
- 💡 **理解技巧**：MCP 让 Claude Code 从一个"代码工具"升级为"全栈运维平台"——代码 + 数据 + 协作 + 监控，全在一个终端窗口里
- 📋 **术语提醒**：`MCP(Model Context Protocol)` — Anthropic 开源的标准协议，让 LLM 与外部工具/数据源安全通信

---

<a id="id7"></a>
## ✅ 知识点7: Boris 的工程哲学 (Boris's Engineering Philosophy)

**理论**
- Boris 在演讲和后续对谈中反复传达几个核心信念，它们构成了 Claude Code 的设计哲学：

**1. 复利工程 (Compounding Engineering)**
> 每个小的优化会复利成巨大的生产力提升。Claude Code 本身的设计就是让每次 1% 的改进累积起来——更快的 Plan Mode、更准的 Hook、更少的权限弹窗。

**2. 不要和模型对赌 (Don't Bet Against the Model)**
> 不要花大量时间写复杂的 prompt hack 来解决当前版本的某个缺陷——模型的下一次更新很可能直接消除这个缺陷。把精力花在可累积的优化上（`CLAUDE.md`、Hooks、Workflows）

**3. Vanilla is powerful（原版就很强）**
> Claude Code 开箱即用的体验已经很好——定制应该是**有针对性的、克制的**，而非全面覆盖。Boris 不鼓励在第一天就配置一大堆自定义设置

**4. 原型 > PRD**
> 与其写产品需求文档，不如让 Claude 快速生成 20-30 个原型版本，在迭代中找到正确答案。Boris 团队的 PR 中位数是 **118 行**——小步快跑

**5. 只做一次的事不值得优化**
> 只有重复的操作才值得写 Hook/Skill/Command。Boris："如果你一天做某件事超过一次，就把它变成 Skill 或 Command"

**注意点**
- 💡 **理解技巧**：这些哲学不是"道德准则"，而是 Boris 团队在真金白银的工程实践中总结的效率法则。每一条背后都有具体的工具和工作流做支撑
- 🔄 **知识关联**："原型 > PRD"对应 [02-workflows.md#id3](./02-workflows.md#id3) 中五步工作流的前三步——快速迭代计划，而非写长篇文档

---

<a id="id8"></a>
## ✅ 知识点8: 从 Prompt Engineering 到 Loop Engineering (Evolution of AI Engineering)

**理论**
- Boris 在 2026 年的对谈中总结了 AI 工程范式的三次进化：

```
Prompt Engineering  →  Context Engineering  →  Loop Engineering
   (2023-2024)           (2024-2025)            (2025-2026)
   "怎样写好 prompt"     "怎样给 Claude 正确的上下文"   "怎样设计 Claude 的自循环"
```

- **Loop Engineering 的核心**：人类不再直接与 Agent 对话——人类设计一个**循环**，由循环自动 prompt Claude。Boris 说他现在**大约一半的工程工作是在手机上通过 Remote Control 完成的**

- **关键特征**：
  - Claude Code 的 `ScheduleWakeup`（定时唤醒）、Hooks 自动触发、子代理编排 → 构建自主工作循环
  - 人类角色从"操作者"变成"设计者"——设计循环的规则、验证标准、终止条件
  - Boris 原话：*"I no longer talk to agents — I write a loop that prompts Claude for me"*（我不再和 agent 对话——我写一个循环替我 prompt Claude）

**注意点**
- 💡 **理解技巧**：Loop Engineering 是验证循环（[02-workflows.md#id7](./02-workflows.md#id7)）的终极形态——不是"写完代码验证一次"，而是"持续运行、持续验证、持续改进"
- 🔄 **知识关联**：Loop Engineering 依赖本文所有前置知识——Worktrees（并行执行）、Sub-agents（任务分配）、Hooks（事件驱动）、SDK（脚本化）、MCP（外部感知）
- 📋 **术语提醒**：`Remote Control(远程控制)` — Claude Code 的移动端功能，可以在手机上查看会话状态、批准关键操作

---

## 🔑 核心要点总结

1. **Git Worktree 实现上下文级的并行**——3-5 个本地会话各司其职，通过系统通知协调
2. **三级复用体系**：Sub-agents（独立上下文）> Skills（打包工作流）> Commands（快捷方式）
3. **MCP 让 Claude Code 从代码工具升级为全栈平台**——代码 + 数据 + 协作 + 监控一站式
4. **Boris 哲学的核心是"累积式优化"**——不要和模型对赌、Vanilla 就很强、小步快跑
5. **Loop Engineering 是下一站**——从"操作 AI"到"设计 AI 的自主循环"

## 📌 实践速查

- **Worktree 速查**：`git worktree add .claude/worktrees/<name> <branch>` | `git worktree list` | `git worktree remove <path>`
- **复用决策速查**：独立子任务 → Sub-agent | 可复用工作流 → Skill | 快捷指令 → Slash Command
- **SDK 速查**：`claude -p "prompt" --output-format json --allowedTools "Bash(git log:*)"`
- **MCP 配置位置**：`.mcp.json` 或 `.claude/settings.json` → `mcpServers` 字段
- **Loop Engineering 本质**：不和人对话，和循环对话——你设计规则，循环驱动 Claude
