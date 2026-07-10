# 📘 01. 基础篇：环境配置与核心概念 (Foundation: Setup & Core Concepts)

> 来源说明：Boris Cherny "Mastering Claude Code in 30 Minutes" 演讲 | 本篇涵盖：Claude Code 安装配置、`CLAUDE.md` 上下文体系、代码库问答、核心快捷键与模式切换

---

## 🧠 核心概念总览（严格按演讲顺序）

- [*知识点1: Claude Code 安装*](#id1)
- [*知识点2: 初始化配置*](#id2)
- [*知识点3: `CLAUDE.md` 上下文文件体系*](#id3)
- [*知识点4: `CLAUDE.md` 编写最佳实践*](#id4)
- [*知识点5: 代码库问答*](#id5)
- [*知识点6: 核心快捷键*](#id6)
- [*知识点7: Ask → Plan → Auto 三模式切换*](#id7)
- [*知识点8: 上下文管理基础*](#id8)

---

<a id="id1"></a>
## ✅ 知识点1: Claude Code 安装 (Installation)

**理论**
- Claude Code 是一个命令行 AI 编程助手，运行在终端中，可以读写文件、执行 Bash 命令、搜索 Web，与 Git 深度集成
- 两种安装方式：`npm` 全局安装 或 `curl` 一键脚本
- 启动命令：在任意项目目录下运行 `claude`

**命令/配置示例**
```bash
# 方式一：npm 全局安装
npm install -g @anthropic-ai/claude-code

# 方式二：curl 安装脚本
curl -fsSL https://claude.ai/install.sh | bash

# 启动 Claude Code
claude
```

**注意点**
- ⚠️ **需要 Node.js 环境**：npm 方式依赖 Node.js 18+，确保已安装
- 💡 **理解技巧**：`claude` 启动后会自动识别当前目录的 Git 仓库，将整个代码库作为上下文
- 📋 **术语提醒**：`CLI(Command Line Interface)` = 命令行界面，`Agentic Coding(代理式编程)` = AI 主动执行而非被动补全

---

<a id="id2"></a>
## ✅ 知识点2: 初始化配置 (Initial Setup)

**理论**
- 首次启动后，通过一系列 `/` 斜杠命令完成环境定制
- `/init` 生成项目级 `CLAUDE.md` 骨架——这是 Boris 认为最重要的第一步
- `/allowed-tools` 精细控制 Claude 可以使用的工具权限（Git、Bash、文件系统）
- `/terminal-setup` 启用 Shift+Enter 多行输入
- `/theme` 设置终端主题（light / dark / colorblind-friendly）
- `/config` 管理通知、输出样式和模型设置
- `/install-github-app` 连接 GitHub，让 Claude 操作 Issues 和 PRs

**命令/配置示例**
```bash
# 在 Claude Code 会话中输入这些斜杠命令：
/init                    # 生成 .claude/CLAUDE.md
/allowed-tools           # 定制工具权限白名单
/terminal-setup          # 配置终端行为
/theme                   # 选择主题
/config                  # 全局配置
/install-github-app      # 连接 GitHub
```

**注意点**
- ⚠️ **权限不要跳过**：Boris 明确说 **永远不要用 `--dangerously-skip-permissions`**，应该通过 `/permissions` 预先配置白名单
- 💡 **理解技巧**：`/allowed-tools` 的核心不是限制 Claude，而是消除每次操作的确认弹窗，让工作流更流畅
- 🔄 **知识关联**：`/init` 生成的 `CLAUDE.md` 内容会在 [知识点3](#id3) 中展开

---

<a id="id3"></a>
## ✅ 知识点3: `CLAUDE.md` 上下文文件体系 (CLAUDE.md Hierarchy)

**理论**
- `CLAUDE.md` 是 Boris 眼中**"整个 Claude Code 里最高杠杆的配置文件"**——它告诉 Claude 关于你、你的团队、你的项目的一切
- Claude Code 启动时自动加载以下位置的文件，按优先级从高到低：

| 优先级 | 位置 | 作用域 | 版本控制 |
|--------|------|--------|----------|
| 1 | 企业管控策略 (Enterprise Policy) | 全公司 | 管理员配置 |
| 2 | `~/.claude/CLAUDE.md` | 用户全局 | 个人维护 |
| 3 | `<project>/.claude/CLAUDE.md` | 项目级 | ✅ 提交到 Git |
| 4 | `<project>/CLAUDE.local.md` | 本地覆盖 | ❌ 不提交 |
| 5 | `~/.claude/rules/*.md` | 用户规则集 | 支持懒加载 |

- 最近版本还支持 `paths:` 前端元数据实现**懒加载**——只在与指定路径相关的任务中加载对应规则文件

**命令/配置示例**
```markdown
# .claude/CLAUDE.md 示例骨架
## 技术栈
- TypeScript + React + Node.js
- PostgreSQL, Redis

## 编码规范
- 使用 async/await，避免回调
- 函数不超过 40 行

## 禁止模式
- 不要使用 any 类型
- 不要直接操作 DOM
```

**注意点**
- ⚠️ **关键区分**：`CLAUDE.md` ≠ Prompt。它是**持久化的项目知识**，而 Prompt 是一次性的任务描述
- 💡 **理解技巧**：可以把 `CLAUDE.md` 类比为"新员工入职文档"——你希望新人（Claude）在动手前就知道的所有上下文
- 📋 **术语提醒**：`Enterprise Policy(企业管控策略)` — 管理员远程下发，个人不可修改；`Lazy Loading(懒加载)` — 按需加载，避免一次性塞入过多 token

---

<a id="id4"></a>
## ✅ 知识点4: `CLAUDE.md` 编写最佳实践 (CLAUDE.md Best Practices)

**理论**
- **保持精简**：目标 ≤ 200 行/文件。Boris 个人偏好 ~60 行。臃肿的 `CLAUDE.md` 会导致模型 drift（行为漂移），开始忽略指令
- **持续修剪**：如果发现 Claude 开始不听话，直接删掉 `CLAUDE.md` 从头重建——"有时删掉重写比修复更快"
- **团队维护工作流**：每次 Claude 做错事 → 加一条规则到 `CLAUDE.md`。PR Review 时 `@claude` 让它自动把学到的东西更新进 `CLAUDE.md`
- **必须包含**：技术栈、编程原则、禁止模式、命名规范、架构决策、构建/测试/运行命令

**注意点**
- 💡 **理解技巧**：Boris 原话——*"Anytime we see Claude do something incorrectly we add it to the CLAUDE.md, so Claude knows not to do it next time."*（每次看到 Claude 做错了，我们就加进 CLAUDE.md，这样下次它就不会再犯）
- 🔄 **知识关联**：在会话中使用 `#`（hash/pound）可以快速捕获当前交互的记忆，自动追加到 `CLAUDE.md`——这是 [知识点6](#id6) 中的核心快捷键之一
- ⚠️ **反模式**：不要试图在 `CLAUDE.md` 里写"万能 Prompt"。越聚焦、越具体，Claude 越不会漂移

---

<a id="id5"></a>
## ✅ 知识点5: 代码库问答 (Codebase Q&A)

**理论**
- Claude Code 的最佳"入门姿势"不是直接写代码，而是**先用它理解代码库**
- 在 Anthropic 内部，新人通过 Claude Code 提问代码库问题，入职时间从**数周缩短到 2-3 天**
- Claude 会自动扫描本地文件系统和 Git 历史来回答问题
- 典型提问模式：某 class 怎么用、为什么这个函数参数这么多（查 Git blame）、"我上周做了什么"（查 Git log）、PR Review

**命令/配置示例**
```bash
# 代码库问答示例——在 Claude Code 会话中直接问：
"@src/RoutingController.py 这个类的职责是什么？它和 AuthMiddleware 怎么交互？"

"这个函数为什么有 7 个参数？从 git history 看看最初设计意图"

"我上周提交了哪些 commit？总结一下"

"Review 一下这个 PR：#123"
```

**注意点**
- 💡 **理解技巧**：代码库问答是"零风险"的入门方式——只读不写，不会有破坏性操作，适合建立对 Claude 的信任
- 🔄 **知识关联**：代码库问答依赖 [知识点3](#id3) 中 `CLAUDE.md` 提供的项目上下文——没有好的 `CLAUDE.md`，Claude 对项目的理解会很有限
- 📋 **术语提醒**：`at-mention(@引用)` — 使用 `@` 符号引用文件/文件夹，将指定内容拉入上下文窗口

---

<a id="id6"></a>
## ✅ 知识点6: 核心快捷键 (Essential Keybindings)

**理论**
- Claude Code 的交互效率高度依赖快捷键——Boris 强调这些不是"锦上添花"，而是日常高频操作
- 所有快捷键可以通过 `/keybindings` 自定义，配置文件存储在 `~/.claude/keybindings.json`

| 快捷键 | 行为 | 使用场景 |
|--------|------|----------|
| `Shift+Tab` | 切换模式（循环） | 在 Ask → Plan → Auto 之间切换 |
| `Shift+Tab` × 2 | 进入 Plan Mode | 复杂任务启动前的强制只读规划 |
| `#` | 记忆捕获 | 把当前交互追加到 `CLAUDE.md` |
| `!` | Bash 模式 | 直接运行 Shell 命令，输出回填上下文 |
| `@` | 文件/文件夹引用 | 将指定文件内容拉入上下文窗口 |
| `Esc` | 取消操作 | 中断当前 Claude 操作 |
| `Esc` × 2 | 回退历史 | 撤销到之前的对话状态（可用 `/resume` 恢复） |
| `Ctrl+R` | 详细输出 | 展示 Claude 的"思考过程"（verbose output） |
| `Enter` | 确认发送 | 单行输入 |

**命令/配置示例**
```bash
/vim          # 启用 Vim 模式
/keybindings  # 打开键位自定义面板
```

**注意点**
- 💡 **理解技巧**：最常用的三个快捷键——`@` 给上下文、`!` 跑命令、`Shift+Tab` 切模式。记住这三个就能覆盖 80% 的日常操作
- 🔄 **知识关联**：`Shift+Tab` 进入 Plan Mode 是 [02-workflows.md](./02-workflows.md) 中五步工作流的起点
- 📋 **术语提醒**：`Verbose Output(详细输出)` — 展示 Claude 每一步的推理过程，类似 "show your work"

---

<a id="id7"></a>
## ✅ 知识点7: Ask → Plan → Auto 三模式切换 (Mode Switching)

**理论**
- Claude Code 有三种工作模式，通过 `Shift+Tab` 循环切换：

| 模式 | 图标 | 行为 | 适用场景 |
|------|------|------|----------|
| **Ask** | 💬 | 只回答问题，不编辑文件 | 代码库问答、探索理解 |
| **Plan** | 📋 | 只读研究和规划，不写入 | 复杂任务的设计阶段 |
| **Auto** | ⚡ | 全自动执行，包括文件编辑 | 简单修改、已确认的实施方案 |

- **Boris 的 80% 原则**：几乎所有复杂任务都从 Plan Mode 启动，花精力打磨计划，然后让 Claude 一次性实现

**注意点**
- ⚠️ **关键用法**：Plan Mode 不是"可选项"——Boris 认为它是高质量产出的核心前提。在 Plan Mode 下 Claude 只能读不能写，你可以在它动手前充分纠正方向
- 💡 **理解技巧**：Ask 适合"探索"、Plan 适合"设计"、Auto 适合"执行"。不要把三者混用——一个任务的三个阶段对应三种模式
- 🔄 **知识关联**：Plan Mode 的具体工作流在 [02-workflows.md](./02-workflows.md) 中展开

---

<a id="id8"></a>
## ✅ 知识点8: 上下文管理基础 (Context Management Basics)

**理论**
- Claude Code 使用上下文窗口来"记住"当前会话的所有内容。但上下文越大，模型表现越差
- **40% 红线**：Boris 建议保持会话上下文使用量在 40% 以下——即使模型支持 1M token，到了 300-400K token 时智能就开始退化
- `/compact` 是核心的上下文瘦身命令——它会让 Claude 总结当前会话的关键信息，丢弃无关细节。可以附加提示（hint）引导压缩方向
- `/clear` 完全重置上下文，适合切换到全新任务
- `@` 引用文件是**精确控制上下文**的最佳手段——只把相关文件拉进来，而不是让 Claude 自己搜索

**命令/配置示例**
```bash
/compact                  # 压缩上下文，保留关键信息
/compact "focus on auth"  # 压缩时聚焦 auth 相关内容
/clear                    # 完全清空上下文（确认后）
```

**注意点**
- ⚠️ **陷阱**：不要在一个会话里塞太多不相关的任务。上下文污染后 Claude 的行为会越来越不可预测
- 💡 **理解技巧**：把上下文窗口想象成"工作记忆"——你一次只能专注几件事，Claude 也一样。子代理（[03-advanced-patterns.md](./03-advanced-patterns.md)）就是用来把大任务拆分成独立工作记忆的
- 🔄 **知识关联**：子代理的上下文隔离是解决上下文膨胀的根本方案，详见 [03-advanced-patterns.md](./03-advanced-patterns.md)

---

## 🔑 核心要点总结

1. **安装后第一件事是 `/init`**——生成项目 `CLAUDE.md`，这是所有高质量交互的基础
2. **`CLAUDE.md` 是活文档**——保持 ≤ 200 行，每次 Claude 犯错就加一条规则，定期修剪
3. **先用 Ask 模式探索代码库**——这是零风险入门方式，能快速建立对 Claude 的信任
4. **学会三个快捷键**——`@` 给上下文、`!` 跑命令、`Shift+Tab` 切模式，覆盖 80% 日常场景
5. **上下文不要超过 40%**——一旦接近红线就用 `/compact` 或新建子代理

## 📌 实践速查

- **常用命令速查**：`/init` → `/allowed-tools` → `/terminal-setup` → `/theme` → `/config`
- **快捷键速查**：`Shift+Tab` 切模式 | `#` 记记忆 | `!` 跑命令 | `@` 引文件 | `Esc` 取消
- **`CLAUDE.md` 清单**：技术栈 ✅ | 编码规范 ✅ | 禁止模式 ✅ | 命名规范 ✅ | 构建命令 ✅
- **常见陷阱**：不要用 `--dangerously-skip-permissions` | CLAUDE.md 不要超 200 行 | 不要在一个会话塞太多任务
