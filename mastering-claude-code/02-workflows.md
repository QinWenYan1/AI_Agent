# 📘 02. 工作流篇：工具链与自动化 (Workflows: Toolchain & Automation)

> 来源说明：Boris Cherny "Mastering Claude Code in 30 Minutes" 演讲 | 本篇涵盖：内置工具体系、Plan Mode 五步工作流、Hooks 事件驱动自动化、验证循环、上下文管理进阶

---

## 🧠 核心概念总览（严格按演讲顺序）

- [*知识点1: 内置工具矩阵*](#id1)
- [*知识点2: Plan Mode 优先原则*](#id2)
- [*知识点3: Explore → Plan → Confirm → Code → Commit 五步法*](#id3)
- [*知识点4: Hooks 事件体系*](#id4)
- [*知识点5: PostToolUse——Boris 最常用的 Hook*](#id5)
- [*知识点6: Stop Hook 与质量守护*](#id6)
- [*知识点7: 验证循环——"最重要的技巧"*](#id7)
- [*知识点8: 上下文管理进阶*](#id8)

---

<a id="id1"></a>
## ✅ 知识点1: 内置工具矩阵 (Built-in Tools)

**理论**
- Claude Code 自带一组内置工具（Tools），让 Claude 可以直接与环境交互。理解这些工具的能力边界是高效使用 Claude Code 的前提

| 工具 | 能力 | 典型用途 |
|------|------|----------|
| **Bash** | 执行任意 Shell 命令 | `npm test`, `git log`, `python train.py` |
| **Read** | 读取文件内容 | 查看源码、配置文件、日志 |
| **Write** | 创建/覆写文件 | 生成新文件 |
| **Edit** | 精确字符串替换 | 修改现有文件（不改动无关代码） |
| **WebFetch** | 抓取网页内容 | 查阅在线文档、API 参考 |
| **WebSearch** | Web 搜索 | 查找最新信息、解决方案 |
| **TODO** | 任务列表追踪 | 管理复杂多步骤任务 |
| **Task** | 子代理调度 | 派生子代理处理子任务 |

- Boris 强调：**让 Claude 多用 Bash 跑验证命令**（测试、lint、构建），而不是盲目信任它的输出

**命令/配置示例**
```bash
# 在 Claude Code 会话中，你无需手动调用工具——Claude 自动选择：
"帮我跑一下测试，如果失败了就修好它"
# Claude 会自动：Bash(npm test) → 读失败日志 → Edit(修复) → Bash(npm test) 验证
```

**注意点**
- ⚠️ **关键区分**：`Edit` 优于 `Write`——`Edit` 做精确字符串替换，不会意外覆盖文件中不相关的部分
- 💡 **理解技巧**：你可以用 `/allowed-tools` 精确控制每种工具的权限范围，如 `Bash(npm run *)` 只允许运行 npm 脚本
- 🔄 **知识关联**：工具权限的白名单策略是 [01-foundation.md#id2](./01-foundation.md#id2) 中 `/allowed-tools` 的核心用途

---

<a id="id2"></a>
## ✅ 知识点2: Plan Mode 优先原则 (Plan Mode First)

**理论**
- Plan Mode 是 Boris 工作流的**基石**——在 Plan Mode 下 Claude 只能读取和研究，不能写入任何文件
- **核心逻辑**：在花 token 写代码之前，先花 token 想清楚写什么。前者浪费的是时间和代码质量，后者是可控的成本
- Boris 说："我几乎每个复杂任务都从 Plan Mode 启动。打磨计划的过程决定了最终产出的质量上限"
- Plan Mode 的具体行为：Claude 会阅读相关文件、分析架构、搜索 Web，然后呈现一个详细的实施计划——但不会写一行代码

**注意点**
- 💡 **理解技巧**：Plan Mode 的 ROI 公式——花 2 分钟打磨计划 = 避免 20 分钟的重写和调试。对于超过 5 分钟的任务，Plan Mode 几乎总是净节省
- ⚠️ **常见误区**：Plan Mode 不等于"慢"。Boris 的原话是"slow is smooth, smooth is fast"——Plan Mode 让 Claude 的首次实现成功率大幅提升，反而整体更快
- 🔄 **知识关联**：Plan Mode 产出的计划直接进入 [知识点3](#id3) 五步工作流的 Confirm 阶段

---

<a id="id3"></a>
## ✅ 知识点3: Explore → Plan → Confirm → Code → Commit 五步法

**理论**
- Boris 推荐的标准工作流，每一步都有明确的职责：

```
Explore（探索） → Plan（规划） → Confirm（确认） → Code（编码） → Commit（提交）
```

| 步骤 | 模式 | 职责 | 产出 |
|------|------|------|------|
| **Explore** | Ask | 阅读代码、搜索 Web、理解上下文 | 对问题和现有代码的深入理解 |
| **Plan** | Plan | 设计实施方案，列出修改清单 | 结构化的实施计划 |
| **Confirm** | Ask/Plan | 人类审查计划，提出修改意见 | 确认的实施计划 |
| **Code** | Auto | Claude 按计划执行编码 | 代码变更 |
| **Commit** | Auto | 运行测试、lint、格式化 → 提交 | 干净的 commit |

- 关键点：**Confirm 是人类介入的唯一节点**——在 Code 之前确保方向正确，Code 之后让 Claude 自己验证和提交

**注意点**
- 💡 **理解技巧**：五个步骤中前三步（EPC）决定质量上限，后两步（CC）决定执行效率。不要在 C&C 上过度 micromanage——让 Claude 自己跑
- 🔄 **知识关联**：Code → Commit 之间的验证环节（测试、lint）是 [知识点7](#id7) 验证循环的核心战场
- ⚠️ **反模式**：跳过 Confirm 直接 Code = 赌博。Boris 说："Plan Mode 是我最常用的功能——比任何单个快捷键都多"

---

<a id="id4"></a>
## ✅ 知识点4: Hooks 事件体系 (Hooks System)

**理论**
- Hooks 是在 Claude Code **生命周期特定节点**自动触发的确定性处理程序。与 AI 推理不同，Hooks 是**确定性的、可预测的**
- 配置在 `.claude/settings.json` 中，12+ 种事件类型：

| Hook 事件 | 触发时机 | 典型用途 |
|-----------|----------|----------|
| `PreToolUse` | 工具调用**前** | 权限路由、使用统计 |
| `PostToolUse` | 工具调用**后** | 自动格式化、lint 修复 |
| `UserPromptSubmit` | 用户提交 prompt 时 | 注入额外上下文 |
| `SessionStart` | 会话启动 | 环境检查、通知 |
| `SessionEnd` | 会话结束 | 清理、通知 |
| `Stop` | Claude 响应结束时 | 质量检查、催促继续 |
| `SubagentStop` | 子代理结束时 | 校验子代理产出 |
| `PreCompact` | 上下文压缩前 | 保存关键信息 |
| `Notification` | 通知事件 | 自定义通知渠道 |
| `PermissionRequest` | 权限请求时 | 自动审批策略 |

**命令/配置示例**
```json
// .claude/settings.json 中的 Hooks 配置结构
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          { "type": "command", "command": "bun run format || true" }
        ]
      }
    ],
    "Stop": [
      {
        "hooks": [
          { "type": "prompt", "prompt": "检查你的产出是否通过了所有测试" }
        ]
      }
    ]
  }
}
```

**注意点**
- 💡 **理解技巧**：Hooks vs Skills——Hooks 是"事件驱动"（发生 X 自动做 Y），Skills 是"指令驱动"（人调用 /skill-name）。Hooks 无需人工触发
- 🔄 **知识关联**：Hooks 和 Sub-agents、Skills、Commands 共同构成 Claude Code 的"编排层"，详见 [03-advanced-patterns.md](./03-advanced-patterns.md)
- 📋 **术语提醒**：`matcher(匹配器)` — 正则表达式，决定哪些工具调用会触发 Hook；`deterministic(确定性的)` — 给定相同输入必然产生相同输出

---

<a id="id5"></a>
## ✅ 知识点5: PostToolUse——Boris 最常用的 Hook

**理论**
- `PostToolUse` 是 Boris 个人用得最多的 Hook。它在 Claude 每次 Write/Edit 文件后自动运行格式化工具
- 为什么需要：Claude 生成的代码**大约 10% 的情况格式不符合项目标准**（缩进、引号风格、换行等）。PostToolUse Hook 自动修复这些，人类无需手动指出
- 写法关键：`"command"` 后面加 `|| true` 防止格式化失败阻塞 Claude 的后续操作

**命令/配置示例**
```json
{
  "PostToolUse": [
    {
      "matcher": "Write|Edit",
      "hooks": [
        { "type": "command", "command": "bun run format || true" }
      ]
    }
  ]
}
```

**注意点**
- 💡 **理解技巧**：这个 Hook 本质上是在说"每次 Claude 写文件后，自动跑项目的格式化工具，如果失败就忽略"。是质量兜底，不是流程阻塞
- ⚠️ **关键警告**：`|| true` 很重要——没有它，格式化失败会导致 Claude 的操作被标记为失败，打断工作流
- 🔄 **知识关联**：PostToolUse + [知识点7](#id7) 验证循环 = 代码质量和格式的双重保障

---

<a id="id6"></a>
## ✅ 知识点6: Stop Hook 与质量守护 (Stop Hook)

**理论**
- `Stop` Hook 在 Claude 完成一个推理回合后触发。它可以注入额外的检查指令
- Boris 的使用方式：Stop Hook 提醒 Claude 在结束前验证自己的产出——"检查是否通过所有测试"、"确认没有遗漏的边界情况"
- 这本质上是在自动化 Boris 的"验证循环"哲学——不让 Claude 在没验证自己的工作时停下

**命令/配置示例**
```json
{
  "Stop": [
    {
      "hooks": [
        {
          "type": "prompt",
          "prompt": "Before finishing, verify your changes: run tests, check for lint errors, and confirm the original task requirements are met."
        }
      ]
    }
  ]
}
```

**注意点**
- 💡 **理解技巧**：可以把 Stop Hook 理解为"临走前的检查清单"——每次都提醒 Claude 自检，比人类事后发现再回来修更高效
- 🔄 **知识关联**：Stop Hook 是 [知识点7](#id7) 验证循环在 Hook 层面的自动化实现

---

<a id="id7"></a>
## ✅ 知识点7: 验证循环——"最重要的技巧" (Verification Loops)

**理论**
- 这是 Boris 在演讲中反复强调的**第一技巧**。他的原话：
  > *"Give Claude a way to verify its own work — if Claude has that feedback loop, it will 2-3x the quality of the final result."*
  > （给 Claude 一个验证自己产出方式——有了这个反馈循环，最终质量会提升 2-3 倍）

- **核心机制**：让 Claude 自动运行验证命令（测试、lint、构建），看到失败输出后自己修复，循环直到通过。人类只需定义"什么算通过"

- **验证手段矩阵**：

| 验证类型 | 命令/工具 | 适用场景 |
|----------|-----------|----------|
| 单元测试 | `npm test`, `pytest` | 逻辑正确性 |
| Lint | `eslint`, `ruff` | 代码风格 |
| 类型检查 | `tsc --noEmit`, `mypy` | 类型安全 |
| 构建 | `npm run build`, `make` | 编译/打包 |
| 浏览器截图 | Puppeteer, Chrome DevTools MCP | UI 视觉验证 |
| iOS 模拟器 | Xcode Simulator | 移动端验证 |

**命令/配置示例**
```bash
# 在 Claude Code 会话中的自然语言指令：
"帮我实现用户登录功能。完成后运行 npm test，如果有失败的测试就修复它，直到全部通过。然后再跑 eslint 和 tsc。"
```

**注意点**
- 💡 **理解技巧**：验证循环的精髓是"让 Claude 看到自己的错误"——模型看到测试失败输出后的修复能力远超"凭空猜测正确性"。不是"写代码 → 人类检查"，而是"写代码 → 自动验证 → 自动修复 → 再验证 → 通过"
- ⚠️ **关键前提**：项目必须有可自动化的验证手段（测试、lint、构建）。如果没有，先让 Claude 写测试，再让 Claude 写代码
- 🔄 **知识关联**：验证循环是五步工作流中 Code → Commit 之间的质量关卡，也通过 Stop Hook 实现自动化

---

<a id="id8"></a>
## ✅ 知识点8: 上下文管理进阶 (Advanced Context Management)

**理论**
- 上下文管理是 Claude Code 使用中**最容易被忽视但影响最大**的技能
- 核心原则（重申 01 篇的内容）：保持会话上下文 ≤ 40%，300-400K token 后智能开始退化
- **进阶策略**：

| 策略 | 命令 | 使用时机 |
|------|------|----------|
| 压缩上下文 | `/compact` | 上下文 > 30% 时主动执行 |
| 带提示压缩 | `/compact "keep auth logic"` | 切换焦点但保留特定上下文 |
| 清空重生 | `/clear` + 新 prompt | 完全切换任务方向 |
| 回退修正 | `Esc × 2`（Double-Esc） | Claude 跑偏时回退，不污染上下文 |
| 子代理卸载 | 派生子代理 | 大任务的子模块用独立上下文 |

- Boris 特别强调：**出错后不要在当前会话里修修补补**——用 `Esc × 2` 回退到出错前的状态，而不是把修正过程也写进上下文

**命令/配置示例**
```bash
/compact                         # 压缩上下文
/compact "focus on payment module"  # 聚焦压缩
/clear                           # 清空（会丢失当前会话所有上下文！）
```

**注意点**
- ⚠️ **关键警告**：`/clear` 是不可逆的——当前会话所有对话记忆都会丢失。只在确实需要全新开始时使用
- 💡 **理解技巧**：上下文污染就像"电话游戏"——每一轮修正都加入新的噪声。回退（`Esc × 2`）比打补丁更干净
- 🔄 **知识关联**：子代理的上下文隔离是解决上下文膨胀的根本方案，详见 [03-advanced-patterns.md](./03-advanced-patterns.md)

---

## 🔑 核心要点总结

1. **Plan Mode 是所有复杂任务的起点**——打磨计划决定了产出的质量上限，不要跳过
2. **Hooks 是"设置一次，受益终身"的投资**——PostToolUse 自动格式化和 Stop Hook 质量守护是最值得配置的两个
3. **验证循环是 Boris 的 No.1 技巧**——让 Claude 跑测试 → 看到失败 → 自己修复，质量提升 2-3 倍
4. **五步工作流**中前三步（EPC）决定质量，后两步（CC）让 Claude 自动完成，人类只在 Confirm 介入
5. **上下文是稀缺资源**——40% 红线，出错回退不修补，大任务拆成子代理

## 📌 实践速查

- **工作流速查**：Explore → Plan → Confirm → Code → Commit（人类只在 Confirm 介入）
- **Hook 配置速查**：`PostToolUse` + `Write|Edit` matcher → `"bun run format || true"` | `Stop` → prompt 验证提醒
- **验证循环速查**：`npm test` → 失败 → 自动修复 → `npm test` → 通过 → `eslint` → `tsc --noEmit`
- **上下文命令速查**：`/compact` 压缩 | `/compact "hint"` 聚焦压缩 | `/clear` 重置 | `Esc × 2` 回退
- **常见陷阱**：跳过 Plan Mode | 没有验证手段就让 Claude 写代码 | 在同一个会话塞太多不相关任务
