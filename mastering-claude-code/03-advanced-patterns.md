# 📘 03. 高级篇：规模化与扩展 (Advanced Patterns: Scaling & Extension)

> 来源说明："Mastering Claude Code in 30 Minutes" 演讲 + 社区最佳实践 | 本篇涵盖：核心快捷键、SDK/CLI 脚本化、并行会话与 Git Worktree、Sub-agents/Skills/Commands 三级复用体系、MCP 集成、工程哲学

---

## 🧠 核心概念总览（严格按演讲顺序）

- [*知识点1: 核心快捷键*](#id1)
- [*知识点2: SDK / CLI 脚本化*](#id2)
- [*知识点3: 并行会话与 Git Worktree*](#id3)
- [*知识点4: Sub-agents 子代理（扩充）*](#id4)
- [*知识点5: Skills 技能包（扩充）*](#id5)
- [*知识点6: Slash Commands 自定义命令（扩充）*](#id6)
- [*知识点7: MCP 外部工具集成（扩充）*](#id7)
- [*知识点8: 工程哲学与范式演进（扩充）*](#id8)

---

<a id="id1"></a>
## ✅ 知识点1: 核心快捷键 (Essential Keybindings)

**理论**
- 终端应用的交互方式极度精简，很多快捷键不太容易被发现。以下是最高频的几个：
- 所有快捷键可以通过 `/keybindings` 自定义，配置文件存储在 `~/.claude/keybindings.json`

| 快捷键 | 行为 | 使用场景 |
|--------|------|----------|
| `Shift+Tab` | 切换到 Auto-accept Edits 模式 | 编辑自动接受（Bash 仍需审批），适合写测试、已知方向对的场景 |
| `#` | 记忆捕获 | 把当前交互经验追加到指定 `CLAUDE.md` memory 文件 |
| `!` | Bash 模式 | 直接运行 Shell 命令，命令+输出都会进入上下文窗口 |
| `@` | 文件/文件夹引用 | 将指定文件内容拉入上下文窗口 |
| `Esc` | 安全停止 | 中断当前操作，不会破坏会话，可随时安全使用 |
| `Esc` × 2 | 回退历史 | 跳到之前的对话状态（之后可用 `claude --resume` 或 `--continue` 恢复） |
| `Ctrl+R` | 详细输出 | 展示 Claude 上下文窗口中看到的完整输出（verbose output） |

**命令/配置示例**
```bash
/vim          # 启用 Vim 模式
/keybindings  # 打开键位自定义面板
```

**注意点**
- 💡 **理解技巧**：最常用的三个快捷键——`@` 给上下文、`!` 跑命令、`Shift+Tab` 切模式。记住这三个就能覆盖 80% 的日常操作
- 💡 **理解技巧**：`Esc` 的使用场景——比如 Claude 生成了一个 20 行的编辑，19 行没问题但 1 行需要改，按 `Esc` 停止，告诉它具体改哪里，然后让它重做
- 💡 **理解技巧**：`!` Bash 模式特别适合长命令和已知操作——你在 Bash 里跑的命令和输出都会被 Claude 看到，作为下一轮的上下文
- 🔄 **知识关联**：`Shift+Tab` 进入 Auto Mode 是 [01-foundation.md](./01-foundation.md) 中 Plan Mode 的后续步骤
- 📋 **术语提醒**：`Verbose Output(详细输出)` — 展示 Claude 每一步的推理过程，类似 "show your work"

---

<a id="id2"></a>
## ✅ 知识点2: SDK / CLI 脚本化 (SDK & CLI Scripting)

**理论**
- Claude Code 支持**非交互式 CLI 调用**，可以嵌入 Shell 脚本、CI/CD 管道。本质上就是把 Claude 当成一个**超级智能的 Unix 工具**
- 核心参数：
  - `claude -p "prompt"` — 单次问答，输出到 stdout
  - `--output-format json` — JSON 格式输出，方便程序解析
  - `--allowedTools` — 限制可用工具，如 `Bash(git log:*)`
- **Unix 管道组合**：`claude -p` 的输出可以 pipe 给其他命令，也可以从其他命令 pipe 输入给 Claude
- 典型场景：CI 自动化、事件响应管道、日志分析——"我们把表面刚刚刮开，还不知道能用它做什么"

**命令/配置示例**
```bash
# 基础用法：单次问答
claude -p "explain what this repository does" --output-format json

# 限制工具：只允许 git log
claude -p "summarize the last 10 commits" --allowedTools "Bash(git log:*)"

# Unix 管道——输出 pipe 给其他命令
claude -p "review this code for bugs" --output-format json | jq '.findings[]'

# Unix 管道——从其他命令 pipe 输入
git diff HEAD~5 | claude -p "Summarize the key changes in this diff"

# 从 GCP bucket 读日志，pipe 给 Claude
gsutil cat gs://bucket/logs.txt | claude -p "Find anomalies in this log"

# CI/CD 集成——GitHub Actions 示例
- name: Claude Code Review
  run: |
    claude -p "review the PR diff. Report only critical bugs." \
      --allowedTools "Read(*)" \
      --output-format json > review.json
```

**注意点**
- 💡 **理解技巧**：`claude -p` 是非交互式的——没有 Plan Mode、没有模式切换。适合"输入 → 输出"的管道式任务，不适合交互式探索
- 💡 **理解技巧**：把 Claude 想象成一个 Unix 工具——你 pipe 数据进去，它 pipe JSON 出来。这个心智模型刚刚开始被探索，组合的可能性无穷无尽
- 🔄 **知识关联**：SDK 模式下，验证循环需要你自己在脚本中实现——`claude -p "fix"` → 跑测试 → `claude -p "fix tests"` → …
- 📋 **术语提醒**：`Non-interactive(非交互式)` — 程序直接调用，没有人在中间审查每一步

---

<a id="id3"></a>
## ✅ 知识点3: 并行会话与 Git Worktree (Parallel Sessions & Git Worktrees)

**理论**
- 高级用户常见的模式：**同时运行多个 Claude Code 会话**，每个专注一个独立任务（架构设计、功能开发、测试、文档……），互不干扰
- 运行方式：
  - 终端 Tab 1-5 各跑一个本地会话
  - SSH 远程会话 + Tmux 隧道
  - 云端 Web UI（`claude.ai/code`）同时跑 5-10 个会话
  - 开启系统通知，哪个完成了就切过去
- 使用 `git worktree` 实现分支级别的文件隔离——每个 worktree 是一个独立的 Git 工作副本，指向不同分支
- 并行会话的核心价值不是"同时写代码"（人类注意力是单线程的），而是**上下文隔离**——每个会话有独立、干净的对话历史，不会相互污染

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
- 💡 **理解技巧**：从"跑一个 Claude"到"跑 N 个并行 Claude"，是效率从量变到质变的门槛——可以同时推进多个任务而不需要频繁切换上下文
- 🔄 **知识关联**：Worktree 隔离 + Sub-agents [知识点4](#id4) 构成两级隔离——进程级（worktree）和上下文级（sub-agent）
- 📋 **术语提醒**：`Worktree(工作树)` — Git 2.5+ 支持的功能，允许同一仓库同时 checkout 多个分支到不同目录

---

<a id="id4"></a>
## ✅ 知识点4: Sub-agents 子代理（扩充） (Sub-agents)

**理论**
- Sub-agent 是 Claude Code 的**上下文隔离机制**——把一个子任务派发给独立的 Claude 实例，它有自己干净的上下文窗口，完成后只把**结论**（而非完整过程）传回主会话
- 定义在 `.claude/agents/*.md`，通过 YAML 前端元数据配置模型、工具权限、系统提示
- 子代理最多可嵌套多层级
- 典型用法：代码审查交给 `code-reviewer` 子代理，验证交给 `verify-app` 子代理

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
- ⚠️ **关键警告**：子代理只返回**结论**。如果 prompt 不够精确，结论可能缺少关键细节。把子代理的输出格式要求写清楚
- 🔄 **知识关联**：Sub-agents、Skills [知识点5](#id5)、Commands [知识点6](#id6) 构成三级复用体系——粒度从细到粗

---

<a id="id5"></a>
## ✅ 知识点5: Skills 技能包（扩充） (Skills)

**理论**
- Skills 是**可复用的打包工作流**，存储在 `.claude/skills/<name>/SKILL.md`
- 与 Sub-agents 的区别：Skills 的指令直接注入当前上下文（不新建实例），适合**过程性知识**
- 支持渐进式披露（Progressive Disclosure）：SKILL.md 主体 → `references/` 详细文档 → `scripts/` 辅助脚本，只在被调用时才加载
- 核心原则：**"如果你一天内做某件事超过一次，就把它变成 Skill 或 Command"**

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

<a id="id6"></a>
## ✅ 知识点6: Slash Commands 自定义命令（扩充） (Custom Slash Commands)

**理论**
- Slash Commands 是**最轻量级的复用单元**——一个 `.md` 文件放在 `.claude/commands/` 下，就变成了 `/command-name`
- 文件内容就是 prompt 模板，支持 `$ARGUMENTS` 占位符接收参数
- 适合重复性操作：`/commit-push-pr`、`/techdebt`、`/bump-version`
- 可以提交到 Git 作为团队共享命令（如 Anthropic 内部用 Slash Command + GitHub Action 自动为 Issue 打标签）

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

<a id="id7"></a>
## ✅ 知识点7: MCP 外部工具集成（扩充） (MCP Integration)

**理论**
- MCP（`Model Context Protocol`）允许 Claude Code 连接外部服务和数据源
- 配置在 `.mcp.json` 或 `.claude/settings.json` 的 `mcpServers` 字段
- Claude Code 启动时加载 MCP Server 的工具列表。但 MCP 的全量工具描述可能消耗 50K+ token——**新版 Tool Search 功能通过按需发现减少了 85% 的 token 消耗**
- 常用 MCP 集成：

| MCP Server | 能力 | 典型场景 |
|------------|------|----------|
| **Slack** | 读写消息 | 把 Slack 线程喂给 Claude → "fix this bug" |
| **GitHub** | Issues/PRs/Repo | 自动 triage Issues |
| **Sentry** | 错误追踪 | 读 Sentry 错误 → Claude 分析 → 修复 |
| **BigQuery** | 数据分析 | SQL 查询（有工程师 6 个月没手写 SQL 了） |
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

<a id="id8"></a>
## ✅ 知识点8: 工程哲学与范式演进（扩充） (Engineering Philosophy & Paradigm Evolution)

**理论**

**一、复利工程 (Compounding Engineering)**
> 每个小的优化会复利成巨大的生产力提升。Claude Code 本身的设计就是让每次 1% 的改进累积起来——更快的反馈、更准的自动化、更少的 friction。

**二、不要和模型对赌 (Don't Bet Against the Model)**
> 不要花大量时间写复杂的 prompt hack 来解决当前版本的某个缺陷——模型的下一次更新很可能直接消除这个缺陷。把精力花在可累积的优化上（`CLAUDE.md`、Hooks、Workflows）。

**三、Vanilla is Powerful（原版就很强）**
> Claude Code 开箱即用的体验已经很好——定制应该是**有针对性的、克制的**，而非全面覆盖。不鼓励在第一天就配置一大堆自定义设置。

**四、原型 > 文档**
> 与其写产品需求文档，不如让 Claude 快速生成 20-30 个原型版本，在迭代中找到正确答案。PR 中位数约 118 行——小步快跑。

**五、只做一次的事不值得优化**
> 只有重复的操作才值得写 Hook/Skill/Command。核心原则："如果你一天做某件事超过一次，就把它变成 Skill 或 Command。"

**六、范式进化：从 Prompt Engineering 到 Loop Engineering**

```
Prompt Engineering  →  Context Engineering  →  Loop Engineering
   (2023-2024)           (2024-2025)            (2025-2026)
   "怎样写好 prompt"     "怎样给 Claude 正确的上下文"   "怎样设计 Claude 的自循环"
```

- **Loop Engineering 的核心**：人类不再直接与 Agent 对话——人类设计一个**循环**，由循环自动 prompt Claude
- **关键特征**：
  - Claude Code 的定时唤醒、Hooks 自动触发、子代理编排 → 构建自主工作循环
  - 人类角色从"操作者"变成"设计者"——设计循环的规则、验证标准、终止条件
  - 核心理念：*"不再和 agent 对话——写一个循环替我 prompt Claude"*

**注意点**
- 💡 **理解技巧**：这些工程原则背后都有具体的工具和工流做支撑，不是空泛的"道德准则"
- 🔄 **知识关联**："原型 > 文档"对应 [01-foundation.md](./01-foundation.md) 中 Plan Mode 的快速迭代思路
- 🔄 **知识关联**：Loop Engineering 依赖本文所有前置知识——Worktrees（并行执行）、Sub-agents（任务分配）、Hooks（事件驱动）、SDK（脚本化）、MCP（外部感知）
- 📋 **术语提醒**：`Remote Control(远程控制)` — Claude Code 的移动端功能，可以在手机上查看会话状态、批准关键操作

---

## 🔑 核心要点总结

1. **五个快捷键覆盖大部分日常操作**——`@`、`!`、`Shift+Tab`、`Esc`、`Ctrl+R`
2. **`claude -p` 把 Claude 变成 Unix 管道中的一个超级工具**——pipe 进去、pipe 出来，刚刚开始探索这种组合的潜力
3. **并行会话是效率质变的门槛**——用 Git Worktree 隔离，3-5 个本地 + 5-10 个云端，上下文互不污染
4. **三级复用体系**：Sub-agents（独立上下文）> Skills（打包工作流）> Commands（快捷方式）
5. **MCP 让 Claude Code 从代码工具升级为全栈平台**——代码 + 数据 + 协作 + 监控一站式
6. **从 Prompt Engineering 到 Loop Engineering**——从"操作 AI"到"设计 AI 的自主循环"

---
