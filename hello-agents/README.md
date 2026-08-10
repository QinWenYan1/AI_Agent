# Hello-Agents: 从零开始构建智能体

> **课程定位**：Datawhale 开源教程《从零开始构建智能体》，5 个 Part 共 16 章。从 Agent 和 LLM 基础出发，手写经典范式，进阶到记忆/上下文/协议/RL/评估，最后通过 3 个综合项目整合。目标：从 LLM 的使用者变成 Agent 系统的构建者。

---

## 📋 笔记导航与重难点

### [01. 智能体与语言模型基础](./01-agent-fundamentals.md)
- **核心**：Agent 的四大组件（感知/规划/行动/记忆）、Agent vs Workflow 的本质区别、LLM 的 Prompt 工程基础
- **难点**：理解 Agent 的自主性（Autonomy）——不是调用链，而是动态决策
- **扩充**：Agent 发展简史、从 GPT 到 ChatGPT 到 Agent 的演进脉络

### [02. 构建你的大语言模型智能体](./02-building-agents.md)
- **核心**：`ReAct` 范式（Reasoning + Acting）、低代码平台搭建、框架开发实践、手写 Agent 框架
- **难点**：ReAct 的 Thought → Action → Observation 循环设计；框架的抽象层级设计
- **扩充**：低代码平台（Dify/Coze）vs 手写框架的取舍

### [03. 高级知识扩展](./03-advanced-topics.md)
- **核心**：`RAG` 记忆与检索、`Context Engineering` 上下文工程、`MCP/A2A` 通信协议、`Agentic-RL` 强化学习、性能评估
- **难点**：RAG 的 Chunk 策略与检索质量调优；MCP 协议的工具发现机制；Agentic-RL 的奖励设计
- **扩充**：评估指标体系的搭建

### [04. 综合案例进阶](./04-projects.md)
- **核心**：智能旅行助手（trip-planner）、自动化深度研究（deep-research）、赛博小镇（AI-Town）三个完整项目
- **难点**：多 Agent 协作的架构设计、长链路任务的错误处理
- **扩充**：项目中的工程化经验

### [05. 毕业设计及未来展望](./05-capstone-and-outlook.md)
- **核心**：毕业设计选题指导、Agent 技术未来趋势
- **难点**：从学习者到构建者的角色转换
- **扩充**：社区生态与职业发展方向

---

## 🔮 延伸学习

- **在线阅读**: [hello-agents.datawhale.cc](https://hello-agents.datawhale.cc)
- **GitHub 仓库**: [datawhalechina/hello-agents](https://github.com/datawhalechina/hello-agents)
- **配套代码**: 仓库 `code/` 目录，每章对应一个可运行的代码示例

---

> 🔗 **返回根目录**：[AI Agent Learning Notes](../README.md)
