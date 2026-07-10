# 📘 01. 基础篇：快速入门与核心工作流 (Foundation: Quick Start & Core Workflow)

> 来源说明："Mastering Claude Code in 30 Minutes" 演讲 | 本篇涵盖：安装配置、代码库问答、编辑代码、Plan Mode、工具集成与验证循环

---

## 🧠 核心概念总览（严格按演讲顺序）

- [*知识点1: Claude Code 安装*](#id1)
- [*知识点2: 初始化配置*](#id2)
- [*知识点3: 代码库问答（入门第一步）*](#id3)
- [*知识点4: 编辑代码与 Plan Mode*](#id4)
- [*知识点5: 工具集成与验证循环*](#id5)
- [*知识点6: Plan Mode 五步工作流（扩充）*](#id6)
- [*知识点7: Ask → Plan → Auto 三模式切换（扩充）*](#id7)
- [*知识点8: 上下文管理基础（扩充）*](#id8)

---

<a id="id1"></a>
## ✅ 知识点1: Claude Code 安装 (Installation)

**如何安装CC...**
- Claude Code 是一种**全代理式（Agentic）AI 编程助手**——不同于传统的逐行代码补全，它面向的是**构建功能、编写整个函数/文件、修复整个 bug**
>📋 **术语提醒**：`CLI(Command Line Interface)` = 命令行界面，`Agentic Coding(代理式编程)` = AI 主动执行而非被动补全

- 兼容所有 IDE：VS Code、Xcode、JetBrains 等，无需更换开发工具。在本地终端、SSH、Tmux 等任何环境中都能运行
- 它是一个**通用工具（General Purpose）**——打开后只有一个 prompt 输入栏，你可以按自己的方式使用它，没有强制的固定工作流
    - 两种安装方式：`npm` 全局安装 或 `curl` 一键脚本
    - 启动命令：在任意项目目录下运行 `claude`

- **命令/配置示例**
    ```bash
    # 方式一：npm 全局安装
    npm install -g @anthropic-ai/claude-code

    # 方式二：curl 安装脚本
    curl -fsSL https://claude.ai/install.sh | bash

    # 启动 Claude Code
    claude
    ```

> ⚠️ **需要 Node.js 环境**：npm 方式依赖 Node.js 18+，确保已安装


---

<a id="id2"></a>
## ✅ 知识点2: 初始化配置 (Initial Setup)

**下载好之后，推荐配置指令...**
- 首次启动后，通过一系列 `/` 斜杠命令完成环境定制
- `/init` 生成项目级 `CLAUDE.md` 骨架——这是最重要的第一步
- `/terminal-setup` 启用 Shift+Enter 多行输入，告别反斜杠换行
- `/theme` 设置终端主题（light / dark / colorblind-friendly）
- `/config` 管理通知、输出样式和模型设置
- `/allowed-tools` 精细控制 Claude 可以使用的工具权限（Git、Bash、文件系统）
> 💡 **理解技巧**：`/allowed-tools` 的核心不是限制 Claude，而是**消除每次操作的确认弹窗**——对常用命令预先授权，让工作流更流畅
- `/install-github-app` 连接 GitHub，安装后可在任何 Issue 或 PR 中 `@claude` 来触发 Claude Code
- macOS 用户可开启系统设置 → 辅助功能 → 听写，按两次听写键即可**语音输入 prompt**，像和同事交流一样自然
> ⚠️ **权限不要跳过**：永远不要用 `--dangerously-skip-permissions`，应该通过 `/permissions` 预先配置白名单

- **命令/配置示例**
    ```bash
    # 在 Claude Code 会话中输入这些斜杠命令：
    /init                    # 生成项目 CLAUDE.md
    /terminal-setup          # 启用 Shift+Enter 多行输入
    /theme                   # 选择主题
    /config                  # 全局配置
    /allowed-tools           # 定制工具权限白名单
    /install-github-app      # 连接 GitHub
    ```
> 🔄 **知识关联**：`/init` 生成的 `CLAUDE.md` 是后续所有高质量交互的基础，详见 [02-workflows.md](./02-workflows.md)

---

<a id="id3"></a>
## ✅ 知识点3: 代码库问答——入门第一步 (Codebase Q&A)

**那么接下来就是了解codebase...**
- Claude Code 的最佳"入门姿势"不是直接写代码，而是**先用它理解代码库**
- 这是推荐给所有新用户和新团队的**第一件事**——代码库问答不需要编辑任何文件，零风险，能快速建立对 Claude 能力的直觉
    - 在 Anthropic 内部，**新人通过 Claude Code 提问代码库问题**，入职时间从**2-3 周缩短到 2-3 天**
    - 无需索引、无需上传代码——代码完全本地化，不做远程数据库，不用于模型训练，打开即用

- **命令/配置示例**
    ```bash
    # 代码库问答示例——在 Claude Code 会话中直接问：
    "@src/RoutingController.py 这个类的职责是什么？它是怎么被实例化的？"

    "这个函数为什么有 15 个参数？参数名为什么这么奇怪？查一下 git history"

    "我上周提交了哪些 commit？总结一下"

    "@GitHub issue #123 这个 Issue 的上下文是什么？"
    ```
- 让 Claude 查 Git 历史时，不需要详细指示每一步——它会自己看 commit 是谁引入的、关联的 Issue、当时的上下文，然后给出总结
- `at-mention(@引用)` — 使用 `@` 符号引用文件/文件夹/Issue，将指定内容拉入上下文窗口

> 💡 **理解技巧**：Claude 不只是做文本搜索——它会**深挖一层**。例如问"这个 class 怎么用"，它不会只搜类名，而是找实例化示例、调用模式，给出的答案接近 Wiki 文档的质量
> 🔄 **知识关联**：代码库问答的效果依赖于项目 `CLAUDE.md` 提供的上下文——详见 [02-workflows.md](./02-workflows.md)


---

<a id="id4"></a>
## ✅ 知识点4: 编辑代码与 Plan Mode (Editing Code & Plan Mode)

**了解完了codebase，接下来就可以开发了...**

- **关键原则**：**在让 Claude 写代码之前，先让它做计划。最容易拿到理想结果的做法是让它先思考**
- 实现 Plan Mode **不需要任何特殊工具或模式切换**——只需要在 prompt 中说一句："**Before you write code, make a plan.**"
- Claude Code 被赋予了一小组工具：编辑文件、运行 Bash 命令、搜索文件。它会**自己编排这些工具的顺序**——先探索代码 → 头脑风暴 → 最后执行编辑。你不需要指定每步用哪个工具

- 高频咒语 "**commit, push, PR**"：Claude 会自动生成 commit message、创建分支、推送、创建 PR——它会自己看 Git log 推断 commit 格式

**命令/配置示例**
```bash
# Plan Mode——最简单的方式
"Before you write code, make a plan. Run it by me for approval."

# 端到端工作流
"Implement user authentication. Before you write code, make a plan for my approval.
Once done, run the tests, and if they pass, commit, push, PR."

# 纠偏
# 如果 Claude 跑偏了方向，按 Esc 停止，告诉它哪里需要调整，让它重做
```

**注意点**
- ⚠️ **反模式**：直接让 Claude 实现一个 3000 行的巨大功能——有时一次就对了，但更常见的情况是**做出来的东西完全不是你想要的**
- 💡 **理解技巧**：Plan Mode 的价值在于**在投入 token 写代码之前，先用少量 token 对齐方向**。花 2 分钟打磨计划 = 避免 20 分钟的重写
- 💡 **理解技巧**：Claude 能理解 "commit, push, PR" 这条指令不是因为有专门的系统提示——纯粹是因为**模型本身足够聪明**，知道怎么操作 Git 和 GitHub
- 🔄 **知识关联**：Plan Mode 的具体工作流展开见 [知识点6](#id6)；模式切换方式见 [知识点7](#id7)

---

<a id="id5"></a>
## ✅ 知识点5: 工具集成与验证循环 (Tool Integration & Verification Loop)

**理论**
- 熟悉基本编辑后，下一步是**让 Claude 接入你团队已有的工具**，这是 Claude Code 真正发挥威力的地方
- 两类工具：
  - **Bash CLI 工具**：告诉 Claude 你的 CLI 名称，它可以用 `--help` 自学用法
  - **MCP 工具**：通过 MCP（Model Context Protocol）接入外部服务，Claude 可以代表你使用这些工具
- **验证循环是提升质量的终极技巧**：给 Claude 一个能检查自己产出的方式——单元测试、Puppeteer 截图、iOS 模拟器——然后让它自己迭代
- 例如：给 Claude 一个 UI mock，"build this web UI" → 第一版还行 → 让它用 Puppeteer 截图比对 → 自动迭代 2-3 轮 → **几乎完美**
- 无论什么领域，只要给 Claude 一个**看到自己结果**的方式，它就会自驱动改进

**命令/配置示例**
```bash
# 告诉 Claude 你的 CLI 工具
"Use our CLI 'barley' to deploy this. Run 'barley --help' first to learn how to use it."

# 验证循环——三种工作流模式
# 1. 探索 + 计划 + 确认（适合新功能）
"Explore the auth module, make a plan, and run it by me before coding."

# 2. 自动验证迭代（适合 UI / 测试）
"Build this UI from the mock. Use Puppeteer to screenshot and compare. Iterate 3 times."

# 3. 测试驱动修复（适合 bug 修复）
"Fix this bug. Run 'npm test' after each attempt. Keep going until all tests pass."
```

**注意点**
- 💡 **理解技巧**："给 Claude 一个验证自己产出的方式"是这次演讲中反复强调的**第一技巧**——有了反馈循环，最终质量提升 2-3 倍
- 💡 **理解技巧**：Claude 第一遍通常做得"还不错"（pretty good），但让它看到结果后迭代 2-3 轮，就能做到"几乎完美"（almost perfect）
- ⚠️ **前提条件**：项目必须有可自动化的验证手段（测试、lint、构建、截图）。如果没有，**先让 Claude 写测试，再让 Claude 写代码**
- 🔄 **知识关联**：MCP 工具的详细配置见 [03-advanced-patterns.md](./03-advanced-patterns.md)；验证循环的更深入讨论见 [02-workflows.md](./02-workflows.md)
- 📋 **术语提醒**：`MCP(Model Context Protocol)` — Anthropic 开源的标准协议，让 LLM 与外部工具/数据源安全通信

---

<a id="id6"></a>
## ✅ 知识点6: Plan Mode 五步工作流（扩充） (Five-Step Plan Mode Workflow)

**理论**
- Plan Mode 的核心逻辑：**在花 token 写代码之前，先花 token 想清楚写什么**。前者浪费的是时间和代码质量，后者是可控的成本
- 推荐的五步工作流：

| 步骤 | 模式 | 职责 | 产出 |
|------|------|------|------|
| **Explore** | Ask | 阅读代码、搜索 Web、理解上下文 | 对问题和现有代码的深入理解 |
| **Plan** | Plan | 设计实施方案，列出修改清单 | 结构化的实施计划 |
| **Confirm** | Ask/Plan | 人类审查计划，提出修改意见 | 确认的实施计划 |
| **Code** | Auto | Claude 按计划执行编码 | 代码变更 |
| **Commit** | Auto | 运行测试、lint、格式化 → 提交 | 干净的 commit |

- 关键点：**Confirm 是人类介入的唯一节点**——在 Code 之前确保方向正确，Code 之后让 Claude 自己验证和提交

**注意点**
- 💡 **理解技巧**：五个步骤中前三步（EPC）决定质量上限，后两步（CC）决定执行效率。不要在 C&C 上过度 micromanage
- 💡 **理解技巧**：Plan Mode 的 ROI——花 2 分钟打磨计划 = 避免 20 分钟的重写和调试。对于超过 5 分钟的任务，Plan Mode 几乎总是净节省
- ⚠️ **反模式**：跳过 Confirm 直接 Code = 赌博。Plan Mode 是高质量产出的核心前提
- 🔄 **知识关联**：Code → Commit 之间的验证环节是 [知识点5](#id5) 验证循环的核心战场

---

<a id="id7"></a>
## ✅ 知识点7: Ask → Plan → Auto 三模式切换（扩充） (Mode Switching)

**理论**
- Claude Code 有三种工作模式，通过 `Shift+Tab` 循环切换：

| 模式 | 图标 | 行为 | 适用场景 |
|------|------|------|----------|
| **Ask** | 💬 | 只回答问题，不编辑文件 | 代码库问答、探索理解 |
| **Plan** | 📋 | 只读研究和规划，不写入 | 复杂任务的设计阶段 |
| **Auto** | ⚡ | 全自动执行（编辑自动接受，Bash 仍需审批） | 简单修改、已确认的实施方案 |

- Auto Mode 下编辑操作**自动接受**（无需逐条确认），Bash 命令仍需审批。如果 Claude 跑偏了，随时可以要求它撤销
- 使用场景：写单元测试时让 Claude 迭代 → 切到 Auto Mode，不用每条编辑都点确认

**注意点**
- ⚠️ **关键用法**：Plan Mode 不是"可选项"——它是高质量产出的核心前提。在 Plan Mode 下 Claude 只能读不能写，可以在它动手前充分纠正方向
- 💡 **理解技巧**：Ask 适合"探索"、Plan 适合"设计"、Auto 适合"执行"。一个任务的三个阶段对应三种模式
- 💡 **理解技巧**：最简单的 Plan Mode 用法不需要切换模式——直接在 prompt 里说"Before you write code, make a plan"即可
- 🔄 **知识关联**：Plan Mode 的具体工作流见 [知识点6](#id6)

---

<a id="id8"></a>
## ✅ 知识点8: 上下文管理基础（扩充） (Context Management Basics)

**理论**
- Claude Code 使用上下文窗口来"记住"当前会话的所有内容。但上下文越大，模型表现越差
- **40% 红线**：保持会话上下文使用量在 40% 以下——即使模型支持 1M token，到了 300-400K token 时智能就开始退化
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
2. **入门第一步是代码库问答，不是编辑代码**——零风险、建立信任、学会 prompt
3. **写代码前先做计划**——不需要特殊工具，一句 "Before you write code, make a plan" 即可
4. **给 Claude 一个验证自己产出的方式**——测试、截图、lint 都行，让它看到自己的结果，质量提升 2-3 倍
5. **上下文不要超过 40%**——一旦接近红线就用 `/compact` 或新建子代理

## 📌 实践速查

- **安装速查**：`npm install -g @anthropic-ai/claude-code` → `claude`
- **初始配置速查**：`/init` → `/terminal-setup` → `/theme` → `/allowed-tools`
- **入门流程速查**：代码库问答（Ask 模式）→ 编辑代码 → Plan Mode → 验证循环 → commit, push, PR
- **Plan Mode 速查**：说 "Before you write code, make a plan. Run it by me for approval." 即可
- **验证循环速查**：`npm test` → 失败 → 自动修复 → `npm test` → 通过
- **上下文命令速查**：`/compact` 压缩 | `/compact "hint"` 聚焦压缩 | `/clear` 重置
