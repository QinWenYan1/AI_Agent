# 📘 第四章 智能体经典范式构建 (Building Classic Agent Paradigms)

> 来源说明：Hello-Agents 教程 第四章 | 本章涵盖：环境准备与 `HelloAgentsLLM` 客户端封装、ReAct 范式（工作流程/工具实现/完整编码/特点局限与调试）、Plan-and-Solve 范式（两阶段原理/规划器/执行器与状态管理）、Reflection 范式（执行-反思-优化循环/记忆模块/多角色提示词/成本收益分析）、三范式对比选型

---

## 🧠 核心概念总览（严格按教程顺序）

**ReAct 部分（4.1–4.2）**

- [*知识点1: 环境准备与基础工具定义*](#id1)
- [*知识点2: ReAct 的工作流程——"思考-行动-观察"循环*](#id2)
- [*知识点3: 工具的定义与实现——三要素与 ToolExecutor*](#id3)
- [*知识点4: ReAct 智能体实现（上）——系统提示词与核心循环*](#id4)
- [*知识点5: ReAct 智能体实现（下）——输出解析、工具执行与观测整合*](#id5)
- [*知识点6: ReAct 运行实例、特点局限与调试技巧*](#id6)

**Plan-and-Solve 部分（4.3）**

- [*知识点7: Plan-and-Solve 的工作原理——先规划，后执行*](#id7)
- [*知识点8: 规划阶段——Planner 与结构化计划生成*](#id8)
- [*知识点9: 执行器与状态管理——Executor 与 PlanAndSolveAgent*](#id9)
- [*知识点10: Plan-and-Solve 运行实例与分析*](#id10)

**Reflection 部分（4.4–4.5）**

- [*知识点11: Reflection 的核心思想——事后自我校正循环*](#id11)
- [*知识点12: 案例设定与记忆模块 Memory*](#id12)
- [*知识点13: Reflection 智能体的编码实现——三种角色提示词*](#id13)
- [*知识点14: Reflection 运行实例与分析*](#id14)
- [*知识点15: Reflection 成本收益分析与本章小结*](#id15)

---

<a id="id1"></a>
## ✅ 知识点1: 环境准备与基础工具定义

**为什么要"重复造轮子"：只有亲手处理解析、重试、死循环这些工程挑战，才能从框架的"使用者"变成智能体应用的"创造者"**

- 上一章我们把大语言模型这个"大脑"摸透了，本章把它变成真正的智能体
- 现代智能体的核心能力在于**将 LLM 的推理能力与外部世界联通**：自主理解用户意图、拆解复杂任务，通过调用代码解释器、搜索引擎、API 等一系列"工具"获取信息、执行操作
- 但它同样有**能力边界**：大模型本身的"幻觉"、复杂任务中可能陷入推理循环、对工具的错误使用。

**章首引子——为什么有了 LangChain、LlamaIndex 还要"重复造轮子"？**

1. **理解机制**：直接使用高度抽象的工具，不利于了解背后的设计机制如何运行、有何好处
2. **暴露工程挑战**：框架替我们处理了模型输出格式的解析、工具调用失败的重试、防止智能体陷入死循环等问题——亲手处理这些问题，是培养系统设计能力的最直接方式
3. **从"使用者"到"创造者"**：掌握设计原理后，当标准组件无法满足复杂需求时，才有深度定制乃至从零构建新智能体的能力

**本章从零实现三种最具代表性的经典范式**：
1. **ReAct (Reasoning and Acting)**（边想边做、动态调整）
2. **Plan-and-Solve**（三思而后行，先计划后执行）
3. **Reflection**（自我批判与修正）

 **安装依赖库：**
- 实战部分主要使用 **Python 3.10 或更高版本**
- `openai` 库用于与大语言模型交互；`python-dotenv` 库用于安全地管理 API 密钥
    ```bash
    pip install openai python-dotenv
    ```

**配置 API 密钥**——模型服务信息统一配置在环境变量中，保证代码通用性：

1. 在项目根目录创建 `.env` 文件
2. 添加以下内容，可指向 OpenAI 官方服务，或**任何兼容 OpenAI 接口的本地/第三方服务**
3. 不会获取可参考原作仓库 `Extra-Chapter/Extra07-环境配置.md`
    ```bash
    # .env file
    LLM_API_KEY="YOUR-API-KEY"
    LLM_MODEL_ID="YOUR-MODEL"
    LLM_BASE_URL="YOUR-URL"
    ```

4. 代码将从此文件自动加载配置。

**封装基础 LLM 调用函数**：
- 定义专属 LLM 客户端类，封装所有与模型服务交互的细节，让主逻辑专注于智能体构建：
    ```python
    class HelloAgentsLLM:
        """为本书 "Hello Agents" 定制的LLM客户端。
        它用于调用任何兼容OpenAI接口的服务，并默认使用流式响应。"""

        def __init__(self, model: str = None, apiKey: str = None, baseUrl: str = None, timeout: int = None): ...
        def think(self, messages: List[Dict[str, str]], temperature: float = 0) -> str: ...
    ```

**实现要点：**

- `__init__`：**传入参数优先**，未提供则从环境变量读取（`LLM_MODEL_ID`/`LLM_API_KEY`/`LLM_BASE_URL`）；`timeout` 默认取 `LLM_TIMEOUT` 环境变量、兜底 60 秒；三者缺一则 `raise ValueError("模型ID、API密钥和服务地址必须被提供或在.env文件中定义。")`
- `think(messages, temperature=0)`：调用 `client.chat.completions.create(..., stream=True)`，**流式处理响应**——逐 `chunk` 取出 `delta.content` 实时打印并收集，结束后 `"".join(collected_content)` 返回完整文本；异常时打印错误并返回 `None`

> ⚠️ **关键约束**：`temperature=0` 是默认值——智能体场景要求输出确定性，便于解析（与采样参数的关系见 [ch3-llm-fundamentals.md#id4](../part1-fundamentals/ch3-llm-fundamentals.md#id4)）
>
> 💡 **理解技巧**：`HelloAgentsLLM` 与第一章的 `OpenAICompatibleClient` 是同类封装（见 [ch1#id8](../part1-fundamentals/ch1-introduction-to-agents.md#id8)），差别在于本章默认流式输出，让读者能实时看到模型的"思考"过程
>
> 📋 **术语**：`流式响应(Streaming Response)`——模型边生成边返回，而非等待全部生成完毕

---

<a id="id2"></a>
## ✅ 知识点2: ReAct 的工作流程——"思考-行动-观察"循环

**思考与行动是相辅相成的：推理使行动更具目的性，行动为推理提供事实依据**

- ReAct 由 Shunyu Yao 于 2022 年提出（ICLR 2023），核心思想是模仿人类解决问题的方式，将**推理 (Reasoning)** 与**行动 (Acting)** 显式结合，形成"思考-行动-观察"循环。

**ReAct 诞生前的两类主流路线：**

- **"纯思考"型**：如`思维链 (Chain-of-Thought)`——能引导模型进行复杂逻辑推理，但**无法与外部世界交互**，容易产生事实幻觉
- **"纯行动"型**：模型直接输出要执行的动作，但**缺乏规划和纠错能力**

**ReAct 的巧妙之处在于认识到思考与行动是相辅相成的思考指导行动，行动的结果又反过来修正思考。通过特殊的提示工程，引导模型每一步输出遵循固定轨迹**：

- **Thought (思考)**：智能体的"内心独白"——分析当前情况、分解任务、制定下一步计划，或反思上一步结果
- **Action (行动)**：决定采取的具体动作，通常是调用外部工具，如 `Search['华为最新款手机']`
- **Observation (观察)**：执行 `Action` 后从外部工具返回的结果，如搜索结果摘要或 API 返回值

**智能体不断重复Thought → Action → Observation循环，将新观察追加到历史记录，形成不断增长的上下文，直到在 `Thought` 中认为已找到最终答案并输出**
- 这形成强大的协同效应：**推理使得行动更具目的性，而行动则为推理提供了事实依据。**

- **形式化表达**：在每个时间步 $t$，智能体的策略（大语言模型 $\pi$）根据初始问题 $q$ 和之前所有"行动-观察"历史轨迹 $((a_1,o_1),\dots,(a_{t-1},o_{t-1}))$ 生成当前思考 $th_t$ 和行动 $a_t$：
    $$\left(th_t,a_t\right)=\pi\left(q,(a_1,o_1),\ldots,(a_{t-1},o_{t-1})\right)$$

- 环境中的工具 $T$ 执行行动 $a_t$ 并返回新观察 $o_t$：
    $$o_t = T(a_t)$$

- 循环不断进行，将新的 $(a_t,o_t)$ 对追加到历史中，直到模型在思考 $th_t$ 中判断任务已完成。

**特别适用的三类场景：**

1. **需要外部知识的任务**：查询实时信息（天气、新闻、股价）、搜索专业领域知识
2. **需要精确计算的任务**：把数学问题交给计算器工具，避免 LLM 的计算错误
3. **需要与 API 交互的任务**：操作数据库、调用某个服务的 API

**本节目标任务**：
- 构建一个具备**使用外部工具**能力的 ReAct 智能体
- 回答 LLM 仅凭自身知识库无法直接回答的问题"华为最新的手机是哪一款？它的主要卖点是什么？"这需要智能体理解自己需要上网搜索、调用工具并总结答案。

> ⚠️ **关键区分**：ReAct 与纯 CoT 的本质差别在于 Observation 来自**真实外部世界**而非模型续写——这是它能回答时效性问题的根本原因
>
> 🔄 **知识关联**：第一章手写旅行助手的主循环就是 ReAct 的雏形（[ch1#id9](../part1-fundamentals/ch1-introduction-to-agents.md#id9)），`Thought:`/`Action:` 交互协议见 [ch1#id7](../part1-fundamentals/ch1-introduction-to-agents.md#id7)；思维链详见 [ch3#id6](../part1-fundamentals/ch3-llm-fundamentals.md#id6)
>
> 📋 **参考文献**：[1] Yao S, et al. *ReAct: Synergizing Reasoning and Acting in Language Models*. ICLR 2023

---

<a id="id3"></a>
## ✅ 知识点3: 工具的定义与实现——三要素与 ToolExecutor

**工具是智能体与外部世界交互的"手和脚"，而工具描述是整个机制中最关键的部分**

如果说大语言模型是智能体的大脑，那么**工具 (Tools)** 就是其与外部世界交互的"手和脚"。针对"华为最新手机"问题，需要为智能体提供网页搜索工具——原作选用 **SerpApi**，它通过 API 提供结构化的 Google 搜索结果，能直接返回"答案摘要框"或精确的知识图谱信息。

```bash
pip install google-search-results
```

并前往 SerpApi 官网注册免费账户，将密钥加入 `.env`：`SERPAPI_API_KEY="YOUR_SERPAPI_API_KEY"`。

**（1）工具的三核心要素：**

1. **名称 (Name)**：简洁、唯一的标识符，供智能体在 `Action` 中调用，如 `Search`
2. **描述 (Description)**：一段清晰的自然语言描述，说明工具用途。**这是整个机制中最关键的部分**——LLM 依赖这段描述来判断何时使用哪个工具
3. **执行逻辑 (Execution Logic)**：真正执行任务的函数或方法

**搜索工具核心逻辑**（签名+要点）：

```python
from serpapi import SerpApiClient

def search(query: str) -> str:
    """基于SerpApi的实战网页搜索引擎工具。
    智能解析搜索结果，优先返回直接答案或知识图谱信息。"""
```

- 请求参数：`engine="google"`、`gl="cn"`（国家代码）、`hl="zh-cn"`（语言代码）；未配置 `SERPAPI_API_KEY` 时直接返回错误提示串
- **智能解析的优先级**（为 LLM 提供质量更高的信息输入）：
  1. `answer_box_list`（答案摘要框列表）→ 直接拼接返回
  2. `answer_box.answer`（答案摘要框）→ 返回精确答案
  3. `knowledge_graph.description`（知识图谱）→ 返回描述
  4. 以上都没有 → 返回**前三个有机结果**（`organic_results[:3]`）的标题+摘要
- 全失败返回"对不起，没有找到关于 '{query}' 的信息。"；异常返回"搜索时发生错误： {e}"

**（2）通用工具执行器**——当智能体需要多种工具（搜索、计算、查数据库……）时，用统一的管理器注册和调度：

```python
class ToolExecutor:
    """一个工具执行器，负责管理和执行工具。"""

    def registerTool(self, name: str, description: str, func: callable): ...  # 注册工具；重名时警告并覆盖
    def getTool(self, name: str) -> callable: ...        # 按名称取执行函数，不存在返回 None
    def getAvailableTools(self) -> str: ...              # 返回 "- name: description" 多行格式化清单
```

内部维护 `self.tools: Dict[str, Dict[str, Any]]`，每个条目存 `{"description": ..., "func": ...}`。

**（3）注册与测试**——把 `search` 注册进执行器并模拟一次 `Action` 调用：

```python
toolExecutor = ToolExecutor()
search_description = "一个网页搜索引擎。当你需要回答关于时事、事实以及在你的知识库中找不到的信息时，应使用此工具。"
toolExecutor.registerTool("Search", search_description, search)
# 模拟 Action: Search['英伟达最新的GPU型号是什么']
```

运行输出（节选）——返回了 RTX 50 系列等前三条有机结果摘要：

```
工具 'Search' 已注册。
--- 可用的工具 ---
- Search: 一个网页搜索引擎。当你需要回答关于时事、事实以及在你的知识库中找不到的信息时，应使用此工具。
--- 执行 Action: Search['英伟达最新的GPU型号是什么'] ---
👀 观察: [1] GeForce RTX 50 系列显卡
GeForce RTX™ 50 系列GPU 搭载NVIDIA Blackwell 架构……
```

> ⚠️ **关键认知**：工具描述是写给 **LLM** 看的"使用说明书"，不是写给人类程序员看的注释——描述的质量直接决定智能体能否在正确的时机选择正确的工具
>
> 💡 **理解技巧**：`getAvailableTools()` 的输出会被拼进提示词（见知识点4 的 `{tools}`），这就是 LLM "知道"自己有哪些工具的唯一途径
>
> 💡 **理解技巧**：智能解析的四级优先级体现了一个工程原则——**尽量把最精确的答案直接喂给 LLM**，减少它从噪音中提炼信息的负担

---

<a id="id4"></a>
## ✅ 知识点4: ReAct 智能体实现（上）——系统提示词与核心循环

**提示词是整个 ReAct 机制的基石，它为大语言模型提供了行动的操作指令**

现在把 LLM 客户端和工具执行器组装成完整的 `ReActAgent` 类。原作将实现拆成几个关键部分讲解。

**（1）系统提示词设计**——模板动态插入可用工具、用户问题与中间步骤的交互历史：

```python
# ReAct 提示词模板
REACT_PROMPT_TEMPLATE = """
请注意，你是一个有能力调用外部工具的智能助手。

可用工具如下:
{tools}

请严格按照以下格式进行回应:

Thought: 你的思考过程，用于分析问题、拆解任务和规划下一步行动。
Action: 你决定采取的行动，必须是以下格式之一:
- `{{tool_name}}[{{tool_input}}]`:调用一个可用工具。
- `Finish[最终答案]`:当你认为已经获得最终答案时。
- 当你收集到足够的信息，能够回答用户的最终问题时，你必须在Action:字段后使用 Finish[最终答案] 来输出最终答案。

现在，请开始解决以下问题:
Question: {question}
History: {history}
"""
```

模板定义了智能体与 LLM 交互的规范，四要素：

- **角色定义**："你是一个有能力调用外部工具的智能助手"，设定 LLM 的角色
- **工具清单 (`{tools}`)**：告知 LLM 它有哪些可用的"手脚"
- **格式规约 (`Thought`/`Action`)**：**最重要的部分**——强制 LLM 输出具有结构性，使代码能精确解析其意图
- **动态上下文 (`{question}`/`{history}`)**：注入用户原始问题和不断累积的交互历史，让 LLM 基于完整上下文决策

> ⚠️ **模板细节**：`{{tool_name}}` 双花括号是 Python `str.format` 的转义——渲染后变成字面量 `{tool_name}` 给模型看，而不是被当作占位符替换掉

**（2）核心循环的实现**——`run` 方法是入口，`while` 循环构成 ReAct 主体，`max_steps` 是**防止无限循环耗尽资源的安全阀**：

```python
class ReActAgent:
    def __init__(self, llm_client: HelloAgentsLLM, tool_executor: ToolExecutor, max_steps: int = 5):
        self.llm_client = llm_client
        self.tool_executor = tool_executor
        self.max_steps = max_steps
        self.history = []

    def run(self, question: str):
        """
        运行ReAct智能体来回答一个问题。
        """
        self.history = [] # 每次运行时重置历史记录
        current_step = 0

        while current_step < self.max_steps:
            current_step += 1
            print(f"--- 第 {current_step} 步 ---")

            # 1. 格式化提示词
            tools_desc = self.tool_executor.getAvailableTools()
            history_str = "\n".join(self.history)
            prompt = REACT_PROMPT_TEMPLATE.format(
                tools=tools_desc,
                question=question,
                history=history_str
            )

            # 2. 调用LLM进行思考
            messages = [{"role": "user", "content": prompt}]
            response_text = self.llm_client.think(messages=messages)

            if not response_text:
                print("错误:LLM未能返回有效响应。")
                break

            # ... (后续的解析、执行、整合步骤)
```

每一步的骨架固定为四拍：**格式化提示词 → 调用 LLM → 执行动作 → 整合结果**，直到任务完成或达到最大步数。

> 💡 **理解技巧**：注意每轮循环都用 `self.history` 重新拼接**完整**提示词再调用 LLM——LLM 本身无状态，"记忆"完全靠把所有历史塞进上下文来实现
>
> 🔄 **知识关联**：这正是第一章 `AGENT_SYSTEM_PROMPT` + 主循环（[ch1#id9](../part1-fundamentals/ch1-introduction-to-agents.md#id9)）的正规化版本：ch1 用正则截断单对 Thought-Action，本章升级为完整的解析-执行-回填流水线

---

<a id="id5"></a>
## ✅ 知识点5: ReAct 智能体实现（下）——输出解析、工具执行与观测整合

**LLM 返回的是纯文本，精确解析与结果回填是形成闭环的关键**

**（3）输出解析器**——LLM 返回纯文本，需用正则表达式精确提取 `Thought` 和 `Action`（这两个方法是 `ReActAgent` 类的一部分）：

```python
    def _parse_output(self, text: str):
        """解析LLM的输出，提取Thought和Action。
        """
        # Thought: 匹配到 Action: 或文本末尾
        thought_match = re.search(r"Thought:\s*(.*?)(?=\nAction:|$)", text, re.DOTALL)
        # Action: 匹配到文本末尾
        action_match = re.search(r"Action:\s*(.*?)$", text, re.DOTALL)
        thought = thought_match.group(1).strip() if thought_match else None
        action = action_match.group(1).strip() if action_match else None
        return thought, action

    def _parse_action(self, action_text: str):
        """解析Action字符串，提取工具名称和输入。
        """
        match = re.match(r"(\w+)\[(.*)\]", action_text, re.DOTALL)
        if match:
            return match.group(1), match.group(2)
        return None, None
```

- `_parse_output`：从完整响应中分离出 `Thought` 和 `Action` 两个主要部分
- `_parse_action`：进一步解析 `Action` 字符串，如从 `Search[华为最新手机]` 提取工具名 `Search` 和输入 `华为最新手机`

**（4）工具调用与执行**——`Action` 的执行中心（位于 `run` 的 `while` 循环内）。先检查是否为 `Finish` 指令，是则结束；否则通过 `tool_executor` 取工具函数执行：

```python
            # 3. 解析LLM的输出
            thought, action = self._parse_output(response_text)

            if thought:
                print(f"思考: {thought}")

            if not action:
                print("警告:未能解析出有效的Action，流程终止。")
                break

            # 4. 执行Action
            if action.startswith("Finish"):
                # 如果是Finish指令，提取最终答案并结束
                final_answer = re.match(r"Finish\[(.*)\]", action).group(1)
                print(f"🎉 最终答案: {final_answer}")
                return final_answer

            tool_name, tool_input = self._parse_action(action)
            if not tool_name or not tool_input:
                # ... 处理无效Action格式 ...
                continue

            print(f"🎬 行动: {tool_name}[{tool_input}]")

            tool_function = self.tool_executor.getTool(tool_name)
            if not tool_function:
                observation = f"错误:未找到名为 '{tool_name}' 的工具。"
            else:
                observation = tool_function(tool_input) # 调用真实工具
```

**（5）观测结果的整合**——**形成闭环的关键**：把 `Action` 和执行后的 `Observation` 添加回历史记录，为下一轮循环提供新上下文：

```python
            print(f"👀 观察: {observation}")

            # 将本轮的Action和Observation添加到历史记录中
            self.history.append(f"Action: {action}")
            self.history.append(f"Observation: {observation}")

        # 循环结束
        print("已达到最大步数，流程终止。")
        return None
```

通过把 `Observation` 追加到 `self.history`，智能体下一轮生成提示词时就能"看到"上一步行动的结果，据此进行新一轮思考和规划。

> ⚠️ **解析顺序**：必须先判断 `Finish` 再走工具解析——`Finish[答案]` 也能被 `(\w+)\[(.*)\]` 匹配，顺序错了会把最终答案当成工具调用
>
> ⚠️ **容错设计**：工具不存在时不中断流程，而是把错误信息作为 `observation` 喂回给 LLM——让它下一步**自我纠正**，这是框架级错误处理的雏形
>
> 💡 **理解技巧**：`re.DOTALL` 让 `.` 匹配换行符——`Thought` 和工具输入都可能跨行，缺了这个标志正则会提前截断

---

<a id="id6"></a>
## ✅ 知识点6: ReAct 运行实例、特点局限与调试技巧

**透明可解释、动态纠错是 ReAct 的闪光点；强依赖 LLM、串行低效、提示词脆弱是它的代价**

**（6）运行实例**——完整代码见原作配套 `code` 文件夹。一次真实运行记录（两步完成任务）：

```
--- 第 1 步 ---
Thought: 要回答这个问题，我需要查找华为最新发布的手机型号及其主要特点。
这些信息可能在我的现有知识库之外，因此需要使用搜索引擎来获取最新数据。
Action: Search[华为最新手机型号及主要卖点]
👀 观察: [1] 华为手机- 华为官网 智能手机 ; Mate 系列. 非凡旗舰 · HUAWEI Mate XTs ……
[2] 2025年华为手机哪一款性价比高？……
[3] 2025年华为新款手机哪个性价比高？……

--- 第 2 步 ---
Thought: 根据搜索结果，华为最新发布的旗舰机型包括Mate 70和Pura 80 Pro+。
为了确定最新型号及其主要卖点，我将重点放在这些信息上……
Action: Finish[根据最新信息，华为的最新手机可能是HUAWEI Pura 80 Pro+或HUAWEI Mate 70。
其中，HUAWEI Mate 70的主要卖点包括顶级的拍照配置，全焦段覆盖，适合专业摄影，
做工出色，并且具有良好的户外抗摔性能。而HUAWEI Pura 80 Pro+则强调了先锋影像技术。]
🎉 最终答案: ……
```

智能体清晰展示了思考链条：**先意识到知识不足需要搜索 → 再根据搜索结果推理总结 → 两步内得出最终答案**。由于模型知识和互联网信息不断更新，你的运行结果可能不同——截至原作编写的 2025 年 9 月 8 日，Mate 70 与 Pura 80 Pro+ 确实是华为当时最新旗舰。这正展示了 ReAct 处理**时效性问题**的强大能力。

**（1）ReAct 的三大特点：**

1. **高可解释性**：最大优点之一是透明——通过 `Thought` 链清晰看到每步"心路历程"（为什么选这个工具、下一步打算做什么），对理解、信任和调试至关重要
2. **动态规划与纠错能力**："走一步，看一步"——根据每步 `Observation` 动态调整后续 `Thought` 和 `Action`；上一步搜索结果不理想，下一步可修正搜索词重试
3. **工具协同能力**：天然结合 LLM 的推理能力与工具的执行能力——LLM 运筹帷幄（规划推理），工具解决具体问题（搜索计算），突破单一 LLM 在知识时效性、计算准确性上的固有局限

**（2）ReAct 的四大固有局限：**

1. **对 LLM 自身能力的强依赖**：若 LLM 的逻辑推理、指令遵循或格式化输出能力不足，容易在 `Thought` 环节错误规划，或在 `Action` 环节生成不合格式指令，导致流程中断
2. **执行效率问题**：循序渐进，完成任务需多次调用 LLM；每次调用伴随网络延迟与计算成本，复杂任务的总耗时和费用较高
3. **提示词的脆弱性**：机制稳定运行建立在精心设计的提示词模板之上，任何微小变动甚至用词差异都可能影响行为；并非所有模型都能持续稳定遵循预设格式
4. **可能陷入局部最优**：步进式决策缺乏全局长远规划，可能因眼前的 `Observation` 选择看似正确但长远非最优的路径，甚至"原地打转"

**（3）五条调试技巧**（行为不符预期时入手）：

- **检查完整的提示词**：每次调用 LLM 前打印格式化好的完整提示词——追溯决策源头的最直接方式
- **分析原始输出**：解析失败时务必打印 LLM 原始返回文本，判断是模型没遵循格式还是解析逻辑有误
- **验证工具的输入与输出**：检查 `tool_input` 是否符合工具函数期望的格式，`observation` 是否是智能体能理解的格式
- **调整提示词中的示例 (Few-shot Prompting)**：加入一两个完整的 "Thought-Action-Observation" 成功案例引导模型
- **尝试不同的模型或参数**：换更强模型，或调整 `temperature`（通常设为 0 保证输出确定性）

> 💡 **理解技巧**：调试的第一性原理是"**先看 LLM 实际收到了什么、实际返回了什么**"——90% 的问题出在提示词与解析之间的格式缝隙里
>
> 🔄 **知识关联**：Few-shot 提示见 [ch3#id5](../part1-fundamentals/ch3-llm-fundamentals.md#id5)；"原地打转"正是 `max_steps` 安全阀存在的原因（知识点4）
>
> 📋 **术语**：`局部最优(Local Optimum)`——步进贪心决策的通病，对比 Plan-and-Solve 的全局规划（知识点7）

---

<a id="id7"></a>
## ✅ 知识点7: Plan-and-Solve 的工作原理——先规划，后执行

**ReAct 像经验丰富的侦探边查边想，Plan-and-Solve 则像建筑师先绘制完整蓝图再严格施工**

Plan-and-Solve Prompting 由 Lei Wang 在 2023 年提出（arXiv:2305.04091），核心动机是解决思维链处理**多步骤、复杂问题时容易"偏离轨道"**的问题。任务处理明确分为两个阶段：**先规划 (Plan)，后执行 (Solve)**。

**一个精妙的比喻**：ReAct 像经验丰富的侦探，根据现场蛛丝马迹（Observation）一步步推理、随时调整调查方向；Plan-and-Solve 则像建筑师，动工前必须先绘制完整蓝图（Plan），然后严格按蓝图施工（Solve）。**现在许多大模型工具的 Agent 模式都融入了这种设计模式。**

**两阶段解耦**（对应原作图 4.2）：

1. **规划阶段 (Planning Phase)**：接收完整问题后，第一个任务**不是直接解决或调用工具**，而是将问题分解、制定清晰的分步行动计划——这个计划本身就是一次 LLM 调用的产物
2. **执行阶段 (Solving Phase)**：获得完整计划后，**严格按照计划步骤逐一执行**；每步可能是独立的 LLM 调用，或对上一步结果的加工，直到所有步骤完成得出答案

"先谋后动"策略使智能体处理需要长远规划的复杂任务时保持更高的**目标一致性**，避免在中间步骤迷失方向。

**形式化表达**：规划模型 $\pi_{\text{plan}}$ 根据原始问题 $q$ 生成包含 $n$ 个步骤的计划：

$$P = \pi_{\text{plan}}(q)$$

执行模型 $\pi_{\text{solve}}$ 逐步完成计划——第 $i$ 步的解决方案 $s_i$ 同时依赖原始问题 $q$、完整计划 $P$ 和之前所有步骤的执行结果 $(s_1, \dots, s_{i-1})$：

$$s_i = \pi_{\text{solve}}(q, P, (s_1, \dots, s_{i-1}))$$

最终答案就是最后一步的执行结果 $s_n$。

**尤其适用的任务**（结构性强、可被清晰分解）：

- **多步数学应用题**：先列计算步骤，再逐一求解
- **需要整合多个信息源的报告撰写**：先规划报告结构（引言、数据来源A、数据来源B、总结），再逐一填充
- **代码生成任务**：先构思函数、类和模块的结构，再逐一实现

> ⚠️ **关键区分**：与 ReAct 把思考和行动**融合在每一步**不同，Plan-and-Solve 把两阶段**解耦**——计划一旦生成就是静态的，执行阶段不再重新规划（这既是稳定性来源，也是灵活性短板）
>
> 🔄 **知识关联**：与 ch1 分类维度中"规划式智能体 (Deliberative Agents)"的思想一脉相承（[ch1#id4](../part1-fundamentals/ch1-introduction-to-agents.md#id4)）
>
> 📋 **参考文献**：[2] Wang L, et al. *Plan-and-Solve Prompting: Improving Zero-Shot Chain-of-Thought Reasoning by Large Language Models*. arXiv:2305.04091, 2023

---

<a id="id8"></a>
## ✅ 知识点8: 规划阶段——Planner 与结构化计划生成

**强制模型输出 Python 列表格式的计划，让解析比自然语言更稳定、更可靠**

为凸显 Plan-and-Solve 在**结构化推理任务**上的优势，本节**不使用工具**，纯通过提示词设计完成推理任务——这类任务答案无法单次查询或计算得出，必须先分解为逻辑连贯的子步骤再按顺序求解，恰好发挥"先规划，后执行"的核心能力。

**目标问题**："一个水果店周一卖出了15个苹果。周二卖出的苹果数量是周一的两倍。周三卖出的数量比周二少了5个。请问这三天总共卖出了多少个苹果？"

问题不算困难，但包含清晰的逻辑链条可供参考——实际遇到逻辑难题时，可参考这个设计模式设计自己的 Agent。智能体需要：规划阶段分解为三个计算步骤（周二销量、周三销量、总销量）→ 执行阶段严格按步执行，每步结果作为下一步输入。

**规划器提示词**——明确告诉模型角色和任务，并给出输出格式范例：

````python
PLANNER_PROMPT_TEMPLATE = """
你是一个顶级的AI规划专家。你的任务是将用户提出的复杂问题分解成一个由多个简单步骤组成的行动计划。
请确保计划中的每个步骤都是一个独立的、可执行的子任务，并且严格按照逻辑顺序排列。
你的输出必须是一个Python列表，其中每个元素都是一个描述子任务的字符串。

问题: {question}

请严格按照以下格式输出你的计划,```python与```作为前后缀是必要的:
```python
["步骤1", "步骤2", "步骤3", ...]
```
"""
````

提示词的三点设计确保输出质量与稳定性：

- **角色设定**："顶级的AI规划专家"，激发模型专业能力
- **任务描述**：清晰定义"分解问题"的目标
- **格式约束**：强制输出 Python 列表格式字符串——**极大地简化后续代码解析，比解析自然语言更稳定、更可靠**

**`Planner` 类**——封装提示词逻辑的规划器：

```python
class Planner:
    def __init__(self, llm_client):
        self.llm_client = llm_client

    def plan(self, question: str) -> list[str]:
        """
        根据用户问题生成一个行动计划。
        """
        prompt = PLANNER_PROMPT_TEMPLATE.format(question=question)

        # 为了生成计划，我们构建一个简单的消息列表
        messages = [{"role": "user", "content": prompt}]

        print("--- 正在生成计划 ---")
        # 使用流式输出来获取完整的计划
        response_text = self.llm_client.think(messages=messages) or ""

        print(f"✅ 计划已生成:\n{response_text}")

        # 解析LLM输出的列表字符串
        try:
            # 找到```python和```之间的内容
            plan_str = response_text.split("```python")[1].split("```")[0].strip()
            # 使用ast.literal_eval来安全地执行字符串，将其转换为Python列表
            plan = ast.literal_eval(plan_str)
            return plan if isinstance(plan, list) else []
        except (ValueError, SyntaxError, IndexError) as e:
            print(f"❌ 解析计划时出错: {e}")
            print(f"原始响应: {response_text}")
            return []
        except Exception as e:
            print(f"❌ 解析计划时发生未知错误: {e}")
            return []
```

> 💡 **理解技巧**：`ast.literal_eval` 而非 `eval`——只解析字面量（列表/字典/数字等），**不会执行任意代码**，是处理 LLM 输出的安全选择
>
> ⚠️ **工程细节**：提示词要求用 ` ```python ` 围栏包裹列表，就是为了给 `split("```python")` 提供**可靠的切分锚点**——输出格式设计要同时服务模型理解和代码解析两端
>
> ⚠️ **兜底策略**：解析失败返回空列表 `[]` 而不是抛异常，由上层 `PlanAndSolveAgent` 检查并优雅终止（见知识点9）

---

<a id="id9"></a>
## ✅ 知识点9: 执行器与状态管理——Executor 与 PlanAndSolveAgent

**执行器的核心职责是状态管理：记录每一步结果并作为上下文提供给后续步骤**

规划器生成蓝图后，执行器 (`Executor`) 逐一完成计划。它除调用 LLM 解决子问题外，还承担至关重要的角色：**状态管理**——记录每步执行结果，作为上下文提供给后续步骤，确保信息在整个任务链条中顺畅流动。

**执行提示词的四个关键信息**（与规划器目标不同：不是分解问题，而是**在已有上下文基础上专注解决当前这一步**）：

- **原始问题**：确保模型始终了解最终目标
- **完整计划**：让模型了解当前步骤在整个任务中的位置
- **历史步骤与结果**：提供至今已完成的工作，作为当前步骤的直接输入
- **当前步骤**：明确指示现在需要解决哪个具体任务

```python
EXECUTOR_PROMPT_TEMPLATE = """
你是一位顶级的AI执行专家。你的任务是严格按照给定的计划，一步步地解决问题。
你将收到原始问题、完整的计划、以及到目前为止已经完成的步骤和结果。
请你专注于解决“当前步骤”，并仅输出该步骤的最终答案，不要输出任何额外的解释或对话。

# 原始问题:
{question}

# 完整计划:
{plan}

# 历史步骤与结果:
{history}

# 当前步骤:
{current_step}

请仅输出针对“当前步骤”的回答:
"""
```

**`Executor` 类**——循环遍历计划、调用 LLM、维护历史记录（状态）：

```python
class Executor:
    def __init__(self, llm_client):
        self.llm_client = llm_client

    def execute(self, question: str, plan: list[str]) -> str:
        """
        根据计划，逐步执行并解决问题。
        """
        history = "" # 用于存储历史步骤和结果的字符串

        print("\n--- 正在执行计划 ---")

        for i, step in enumerate(plan):
            print(f"\n-> 正在执行步骤 {i+1}/{len(plan)}: {step}")

            prompt = EXECUTOR_PROMPT_TEMPLATE.format(
                question=question,
                plan=plan,
                history=history if history else "无", # 如果是第一步，则历史为空
                current_step=step
            )

            messages = [{"role": "user", "content": prompt}]

            response_text = self.llm_client.think(messages=messages) or ""

            # 更新历史记录，为下一步做准备
            history += f"步骤 {i+1}: {step}\n结果: {response_text}\n\n"

            print(f"✅ 步骤 {i+1} 已完成，结果: {response_text}")

        # 循环结束后，最后一步的响应就是最终答案
        final_answer = response_text
        return final_answer
```

**整合：`PlanAndSolveAgent`**——职责清晰：接收 LLM 客户端、初始化内部规划器和执行器、提供 `run` 方法启动整个流程：

```python
class PlanAndSolveAgent:
    def __init__(self, llm_client):
        """
        初始化智能体，同时创建规划器和执行器实例。
        """
        self.llm_client = llm_client
        self.planner = Planner(self.llm_client)
        self.executor = Executor(self.llm_client)

    def run(self, question: str):
        """
        运行智能体的完整流程:先规划，后执行。
        """
        print(f"\n--- 开始处理问题 ---\n问题: {question}")

        # 1. 调用规划器生成计划
        plan = self.planner.plan(question)

        # 检查计划是否成功生成
        if not plan:
            print("\n--- 任务终止 --- \n无法生成有效的行动计划。")
            return

        # 2. 调用执行器执行计划
        final_answer = self.executor.execute(question, plan)

        print(f"\n--- 任务完成 ---\n最终答案: {final_answer}")
```

这个设计体现了**"组合优于继承"**的原则：`PlanAndSolveAgent` 本身不含复杂逻辑，而是作为一个**协调者 (Orchestrator)**，清晰调用内部组件完成任务。

> 💡 **理解技巧**：`history` 字符串逐步累积（`步骤 i: ...\n结果: ...`），就是 P&S 的"状态"——对比 ReAct 的 `self.history` 列表，本质都是用提示词模拟记忆，只是这里不存在 Observation（没有外部工具）
>
> 📋 **术语**：`协调者(Orchestrator)`——不亲自干活、只调度组件的控制层，是复杂 Agent 系统的常见角色

---

<a id="id10"></a>
## ✅ 知识点10: Plan-and-Solve 运行实例与分析

**四步执行，历史结果逐步传递，最终得出正确答案 70**

完整代码见原作配套 `code` 文件夹。运行记录：

````bash
--- 开始处理问题 ---
问题: 一个水果店周一卖出了15个苹果。周二卖出的苹果数量是周一的两倍。周三卖出的数量比周二少了5个。请问这三天总共卖出了多少个苹果？
--- 正在生成计划 ---
```python
["计算周一卖出的苹果数量： 15个", "计算周二卖出的苹果数量： 周一数量 × 2 = 15 × 2 = 30个", "计算周三卖出的苹果数量： 周二数量 - 5 = 30 - 5 = 25个", "计算三天总销量： 周一 + 周二 + 周三 = 15 + 30 + 25 = 70个"]
```

--- 正在执行计划 ---
-> 正在执行步骤 1/4: 计算周一卖出的苹果数量: 15个      → 结果: 15
-> 正在执行步骤 2/4: 计算周二卖出的苹果数量: 15 × 2     → 结果: 30
-> 正在执行步骤 3/4: 计算周三卖出的苹果数量: 30 - 5     → 结果: 25
-> 正在执行步骤 4/4: 计算三天总销量: 15 + 30 + 25       → 结果: 70

--- 任务完成 ---
最终答案: 70
````

**三点流程分析：**

1. **规划阶段**：`Planner` 成功把复杂应用题分解成含四个逻辑步骤的 **Python 列表**——结构化计划为后续执行奠定基础
2. **执行阶段**：`Executor` 严格按计划逐步执行；每步都把历史结果作为上下文，**确保信息正确传递**（步骤2 正确使用步骤1 的结果"15个"，步骤3 正确使用步骤2 的"30个"）
3. **结果**：整个过程逻辑清晰、步骤明确，准确得出正确答案"70个"

> 💡 **理解技巧**：注意运行日志里每步都重新调用一次 LLM——P&S 的"执行"不是代码算出来的，而是模型在历史上下文约束下一步步推出来的
>
> ⚠️ **对比 ReAct**：全程**无 Observation、无中途调整**——计划生成后不再改变。本任务逻辑路径确定，这是优势；若中途某步失败，静态计划就成了短板（原作习题 4 专门讨论"动态重规划"机制）

---

<a id="id11"></a>
## ✅ 知识点11: Reflection 的核心思想——事后自我校正循环

**像人类校对初稿、验算数学题一样，让智能体审视自己的工作并迭代优化**

在 ReAct 和 Plan-and-Solve 中，智能体一旦完成任务，工作流程便告结束——但生成的初始答案（无论行动轨迹还是最终结果）都可能存在谬误或有待改进。Reflection 机制的核心思想，是引入一种**事后（post-hoc）的自我校正循环**：像人类一样审视自己的工作、发现不足、迭代优化。

灵感来源于人类学习过程：完成初稿后校对、解出数学题后验算。这一思想在多个研究中体现，如 Shinn, Noah 在 2023 年提出的 **Reflexion 框架**（NeurIPS 2023）。核心工作流程是简洁的三步循环：**执行 → 反思 → 优化**。

1. **执行 (Execution)**：用熟悉的方法（如 ReAct 或 Plan-and-Solve）尝试完成任务，生成初步解决方案或行动轨迹——可看作"初稿"
2. **反思 (Reflection)**：调用一个独立的（或带特殊提示词的）LLM 实例扮演**"评审员"**，审视"初稿"并从多个维度评估：
   - **事实性错误**：是否存在与常识或已知事实相悖的内容？
   - **逻辑漏洞**：推理过程是否存在不连贯或矛盾之处？
   - **效率问题**：是否有更直接、更简洁的路径？
   - **遗漏信息**：是否忽略了问题的某些关键约束或方面？

   根据评估生成结构化的**反馈 (Feedback)**，指出具体问题和改进建议
3. **优化 (Refinement)**：把"初稿"和"反馈"作为新上下文，再次调用 LLM，要求根据反馈修正初稿，生成更完善的"修订稿"

如原作图 4.3 所示，循环可重复多次，直到反思阶段不再发现新问题，或达到预设迭代次数上限。

**形式化表达**：设 $O_i$ 是第 $i$ 次迭代的输出（$O_0$ 为初始输出），反思模型 $\pi_{\text{reflect}}$ 生成针对 $O_i$ 的反馈 $F_i$：

$$F_i = \pi_{\text{reflect}}(\text{Task}, O_i)$$

优化模型 $\pi_{\text{refine}}$ 结合原始任务、上一版输出和反馈，生成新版输出 $O_{i+1}$：

$$O_{i+1} = \pi_{\text{refine}}(\text{Task}, O_i, F_i)$$

**与前两种范式相比，Reflection 的三大价值：**

- 提供**内部纠错回路**：不再完全依赖外部工具的反馈（ReAct 的 Observation），能修正**更高层次的逻辑和策略错误**
- 把一次性任务执行转变为**持续优化过程**，显著提升复杂任务的最终成功率和答案质量
- 构建临时的**"短期记忆"**：整个"执行-反思-优化"轨迹形成宝贵的经验记录——智能体不仅知道最终答案，还记得自己如何从不完美的初稿迭代到最终版本；这个记忆系统还可以是**多模态的**（反思修正文本以外的输出，如代码、图像），为构建更强大的多模态智能体奠定基础

> ⚠️ **关键区分**：ReAct 纠错靠**外部** Observation（世界反馈），Reflection 纠错靠**内部**评审员（自我批判）——前者修正"行动"，后者能修正"思路"
>
> 🔄 **知识关联**：ch2 讲 LLM 智能体架构时，规划模块就包含"反思 (Reflection)"与"自我批判 (Self-criticism)"子模块（[ch2#id13](../part1-fundamentals/ch2-history-of-agents.md#id13)），本章是它的完整实现
>
> 📋 **参考文献**：[3] Shinn N, et al. *Reflexion: Language Agents with Verbal Reinforcement Learning*. NeurIPS 2023

---

<a id="id12"></a>
## ✅ 知识点12: 案例设定与记忆模块 Memory

**迭代的前提是记住之前的尝试与反馈——短期记忆模块是 Reflection 范式的必需品**

实战中需引入**记忆管理机制**——Reflection 对应着信息的存储和提取；如果上下文足够长，让"评审员"直接获取所有信息进行反思，往往会传入大量冗余信息。本步实践完成**代码生成与迭代优化**。

**目标任务**："编写一个Python函数，找出1到n之间所有的素数 (prime numbers)。"

这个任务是检验 Reflection 的绝佳场景，三点理由：

1. **存在明确的优化路径**：LLM 初次生成的代码很可能是简单但效率低下的递归实现
2. **反思点清晰**：可通过反思发现"时间复杂度过高"或"存在重复计算"的问题
3. **优化方向明确**：可根据反馈优化为更高效的迭代版本或使用备忘录模式的版本

**记忆模块设计**——Reflection 的核心在于迭代，迭代的前提是**能记住之前的尝试和获得的反馈**。"短期记忆"模块是实现该范式的必需品，负责存储每一次"执行-反思"循环的完整轨迹：

```python
from typing import List, Dict, Any, Optional

class Memory:
    """
    一个简单的短期记忆模块，用于存储智能体的行动与反思轨迹。
    """

    def __init__(self):
        """
        初始化一个空列表来存储所有记录。
        """
        self.records: List[Dict[str, Any]] = []

    def add_record(self, record_type: str, content: str):
        """
        向记忆中添加一条新记录。

        参数:
        - record_type (str): 记录的类型 ('execution' 或 'reflection')。
        - content (str): 记录的具体内容 (例如，生成的代码或反思的反馈)。
        """
        record = {"type": record_type, "content": content}
        self.records.append(record)
        print(f"📝 记忆已更新，新增一条 '{record_type}' 记录。")

    def get_trajectory(self) -> str:
        """
        将所有记忆记录格式化为一个连贯的字符串文本，用于构建提示词。
        """
        trajectory_parts = []
        for record in self.records:
            if record['type'] == 'execution':
                trajectory_parts.append(f"--- 上一轮尝试 (代码) ---\n{record['content']}")
            elif record['type'] == 'reflection':
                trajectory_parts.append(f"--- 评审员反馈 ---\n{record['content']}")

        return "\n\n".join(trajectory_parts)

    def get_last_execution(self) -> Optional[str]:
        """
        获取最近一次的执行结果 (例如，最新生成的代码)。
        如果不存在，则返回 None。
        """
        for record in reversed(self.records):
            if record['type'] == 'execution':
                return record['content']
        return None
```

`Memory` 类设计简洁，主体四点：

- 用列表 `records` **按顺序**存储每一次行动和反思
- `add_record`：添加新条目（`'execution'` 或 `'reflection'` 两种类型）
- `get_trajectory`：**核心方法**——把记忆轨迹"序列化"成一段文本，可直接插入后续提示词，为反思和优化提供完整上下文
- `get_last_execution`：方便获取最新"初稿"以供反思

> 💡 **理解技巧**：`Memory` 与 ReAct 的 `self.history` 列表本质相同——都是**用追加式记录模拟记忆**；区别在于这里记录带类型标签，支持按类型检索（`get_last_execution` 只取代码不取反馈）
>
> 🔄 **知识关联**：这是 ch2 中 LLM 智能体"记忆模块 (Memory)"（[ch2#id13](../part1-fundamentals/ch2-history-of-agents.md#id13)）的最小可用实现；更完整的记忆与检索机制属于后续章节内容

---

<a id="id13"></a>
## ✅ 知识点13: Reflection 智能体的编码实现——三种角色提示词

**Reflection 需要多个不同角色的提示词协同工作，其中反思提示词是整个机制的灵魂**

有了 `Memory` 模块作基础，现在构建 `ReflectionAgent` 核心逻辑。工作流程围绕"执行-反思-优化"循环展开，**通过精心设计的提示词引导 LLM 扮演不同角色**——这是与前两种范式最大的不同。

**（1）提示词设计——三个角色模板：**

**① 初始执行提示词 (Execution Prompt)**：首次尝试解决问题，内容直接，只要求完成任务：

```text
INITIAL_PROMPT_TEMPLATE = """
你是一位资深的Python程序员。请根据以下要求，编写一个Python函数。
你的代码必须包含完整的函数签名、文档字符串，并遵循PEP 8编码规范。

要求: {task}

请直接输出代码，不要包含任何额外的解释。
"""
```

**② 反思提示词 (Reflection Prompt)**——**Reflection 机制的灵魂**。指示模型扮演"代码评审员"，对上一轮代码进行批判性分析，提供具体、可操作的反馈：

````text
REFLECT_PROMPT_TEMPLATE = """
你是一位极其严格的代码评审专家和资深算法工程师，对代码的性能有极致的要求。
你的任务是审查以下Python代码，并专注于找出其在算法效率上的主要瓶颈。

# 原始任务:
{task}

# 待审查的代码:
```python
{code}
```

请分析该代码的时间复杂度，并思考是否存在一种算法上更优的解决方案来显著提升性能。
如果存在，请清晰地指出当前算法的不足，并提出具体的、可行的改进算法建议（例如，使用筛法替代试除法）。
如果代码在算法层面已经达到最优，才能回答"无需改进"。

请直接输出你的反馈，不要包含任何额外的解释。
"""
````

**③ 优化提示词 (Refinement Prompt)**：收到反馈后，引导模型根据反馈修正和优化原有代码：

```text
REFINE_PROMPT_TEMPLATE = """
你是一位资深的Python程序员。你正在根据一位代码评审专家的反馈来优化你的代码。

# 原始任务:
{task}

# 你上一轮尝试的代码:
{last_code_attempt}
评审员的反馈：
{feedback}

请根据评审员的反馈，生成一个优化后的新版本代码。
你的代码必须包含完整的函数签名、文档字符串，并遵循PEP 8编码规范。
请直接输出优化后的代码，不要包含任何额外的解释。
"""
```

**（2）智能体封装**——整合提示词逻辑与 `Memory` 模块：

```python
class ReflectionAgent:
    def __init__(self, llm_client, max_iterations=3):
        self.llm_client = llm_client
        self.memory = Memory()
        self.max_iterations = max_iterations

    def run(self, task: str):
        print(f"\n--- 开始处理任务 ---\n任务: {task}")

        # --- 1. 初始执行 ---
        print("\n--- 正在进行初始尝试 ---")
        initial_prompt = INITIAL_PROMPT_TEMPLATE.format(task=task)
        initial_code = self._get_llm_response(initial_prompt)
        self.memory.add_record("execution", initial_code)

        # --- 2. 迭代循环:反思与优化 ---
        for i in range(self.max_iterations):
            print(f"\n--- 第 {i+1}/{self.max_iterations} 轮迭代 ---")

            # a. 反思
            print("\n-> 正在进行反思...")
            last_code = self.memory.get_last_execution()
            reflect_prompt = REFLECT_PROMPT_TEMPLATE.format(task=task, code=last_code)
            feedback = self._get_llm_response(reflect_prompt)
            self.memory.add_record("reflection", feedback)

            # b. 检查是否需要停止
            if "无需改进" in feedback:
                print("\n✅ 反思认为代码已无需改进，任务完成。")
                break

            # c. 优化
            print("\n-> 正在进行优化...")
            refine_prompt = REFINE_PROMPT_TEMPLATE.format(
                task=task,
                last_code_attempt=last_code,
                feedback=feedback
            )
            refined_code = self._get_llm_response(refine_prompt)
            self.memory.add_record("execution", refined_code)

        final_code = self.memory.get_last_execution()
        print(f"\n--- 任务完成 ---\n最终生成的代码:\n```python\n{final_code}\n```")
        return final_code

    def _get_llm_response(self, prompt: str) -> str:
        """一个辅助方法，用于调用LLM并获取完整的流式响应。"""
        messages = [{"role": "user", "content": prompt}]
        response_text = self.llm_client.think(messages=messages) or ""
        return response_text
```

> ⚠️ **终止条件的脆弱性**：`"无需改进" in feedback` 是字符串匹配——如果评审员说"代码基本无需大改"就漏检了。更鲁棒的方案（如结构化输出判断）是原作习题 5 的延伸方向
>
> 💡 **理解技巧**：反思提示词里"**极其严格**""对性能有极致的要求""**才能**回答无需改进"这些措辞不是修辞——它们直接决定评审员是敷衍点赞还是认真挑刺（运行实例见知识点14）
>
> 💡 **理解技巧**：每轮迭代是两次 LLM 调用（反思 + 优化），`max_iterations=3` 意味着最多 1 + 2×3 = 7 次调用——这就是知识点15 成本分析的具体来源

---

<a id="id14"></a>
## ✅ 知识点14: Reflection 运行实例与分析

**从试除法到埃拉托斯特尼筛法：一次有效的批判驱动算法复杂度从 O(n·√n) 降到 O(n log log n)**

完整代码见原作配套 `code` 文件夹。运行实例（任务：找出 1 到 n 之间所有的素数）：

**运行轨迹（节选）：**

- **初始尝试**：模型生成一版功能正确但低效的实施代码（试除法思路）
- **第 1 轮反思**：评审员精准指出瓶颈——

> 当前代码的时间复杂度为 O(n * sqrt(n))。虽然对于较小的 n 值，这种实现是可以接受的，但当 n 非常大时，性能会显著下降。主要瓶颈在于每个数都需要进行试除法检查，这导致了较高的时间开销。
>
> 建议使用埃拉托斯特尼筛法（Sieve of Eratosthenes），该算法的时间复杂度为 O(n log(log n))，能够显著提高查找素数的效率。

- **第 1 轮优化**：根据反馈生成筛法实现
- **第 2 轮反思**：面对已高效的筛法，展现出更深层次的知识——

> 当前代码使用了 Eratosthenes 筛法，时间复杂度为 O(n log log n)，空间复杂度为 O(n)。此算法在寻找 1 到 n 之间的所有素数时已经非常高效，通常情况下无需进一步优化。但在某些特定场景下，可以考虑以下改进：
>
> 1. **分段筛法（Segmented Sieve）**：适用于 n 非常大但内存有限的情况。将区间分成多个小段，每段分别用筛法处理，减少内存使用。
> 2. **奇数筛法（Odd Number Sieve）**：除了 2 以外，所有素数都是奇数。可以在初始化 `is_prime` 数组时只标记奇数，这样可以将空间复杂度降低一半，同时减少一些不必要的计算。
>
> 然而，这些改进对于大多数应用场景来说并不是必需的……因此，在一般情况下，**无需改进**。

- **终止**："无需改进"触发终止条件，任务完成

**最终生成的代码**（完整的筛法实现）：

```python
def find_primes(n):
    """
    Finds all prime numbers between 1 and n using the Sieve of Eratosthenes algorithm.

    :param n: The upper limit of the range to find prime numbers.
    :return: A list of all prime numbers between 1 and n.
    """
    if n < 2:
        return []

    is_prime = [True] * (n + 1)
    is_prime[0] = is_prime[1] = False

    p = 2
    while p * p <= n:
        if is_prime[p]:
            for i in range(p * p, n + 1, p):
                is_prime[i] = False
        p += 1

    primes = [num for num in range(2, n + 1) if is_prime[num]]
    return primes
```

**三点深度分析：**

1. **有效的"批判"是优化的前提**：第一轮反思中，由于使用了"极其严格"且"专注于算法效率"的提示词，智能体**没有满足于功能正确的初版代码**，而是精准指出 $O(n\sqrt{n})$ 的时间复杂度瓶颈，并提出算法层面的改进建议——埃氏筛
2. **迭代式改进**：接收明确反馈后，优化阶段成功实现更高效的筛法，复杂度降至 $O(n\log\log n)$，完成第一次有意义的自我迭代
3. **收敛与终止**：第二轮反思中，智能体面对已高效的筛法不仅肯定其效率，甚至提及分段筛法等更高级方向，但最终做出"一般情况下无需改进"的**正确判断**——触发终止条件，优化过程收敛

这个案例充分证明：设计良好的 Reflection 机制，价值不仅在于修复错误，更在于**驱动解决方案在质量和效率上实现阶梯式的提升**——这是构建复杂、高质量智能体的关键技术之一。

> 💡 **理解技巧**：注意第 2 轮反思的"克制"同样重要——知道何时**停止**优化（收益递减）比无休止改进更接近真实工程判断
>
> 📋 **术语**：`埃拉托斯特尼筛法(Sieve of Eratosthenes)`——从 2 开始，把每个素数的倍数全部标记为合数，$O(n\log\log n)$；对比试除法 $O(n\sqrt{n})$

---

<a id="id15"></a>
## ✅ 知识点15: Reflection 成本收益分析与本章小结

**Reflection 是典型的"以成本换质量"策略，选型取决于任务的核心需求**

Reflection 提升任务解决质量的能力并非没有代价，实际应用中需要权衡收益与成本。

**（1）三大主要成本：**

1. **模型调用开销增加**：最直接的成本——每轮迭代至少额外调用两次 LLM（一次反思、一次优化），多轮迭代则 API 调用成本和计算资源成倍增加
2. **任务延迟显著提高**：Reflection 是**串行过程**，每轮优化必须等上一轮反思完成，总耗时显著延长，不适合实时性要求高的场景
3. **提示工程复杂度上升**：成功很大程度上依赖高质量、有针对性的提示词——为"执行""反思""优化"等不同阶段设计调试有效提示词，需要投入更多开发精力

**（2）两大核心收益：**

1. **解决方案质量的跃迁**：最大收益——把"合格"的初始方案迭代优化成"优秀"的最终方案。从功能正确到性能高效、从逻辑粗糙到逻辑严谨的提升，在很多关键任务中至关重要
2. **鲁棒性与可靠性增强**：通过内部自我纠错循环，发现并修复初始方案中的逻辑漏洞、事实性错误或边界情况处理不当，大大提高最终结果的可靠性

**定位：典型的"以成本换质量"策略。** 适合**对最终结果的质量、准确性和可靠性有极高要求，且对实时性要求相对宽松**的场景：

- 生成关键的业务代码或技术报告
- 科学研究中的复杂逻辑推演
- 需要深度分析和规划的决策支持系统

反之，若需要快速响应、或"大致正确"的答案已足够，更轻量的 ReAct 或 Plan-and-Solve 是更具性价比的选择。

**本章小结——三范式核心回顾：**

| | ReAct | Plan-and-Solve | Reflection |
|---|---|---|---|
| **策略** | 边想边做，步进循环 | 先谋后动，两阶段解耦 | 事后校正，迭代循环 |
| **核心组件** | T-A-O 循环 + ToolExecutor | Planner + Executor | Memory + 三角色提示词 |
| **核心优势** | **环境适应性**、**动态纠错能力** | **结构性**、**稳定性** | **显著提升解决方案质量** |
| **主要代价** | 串行多次调用、提示词脆弱、可能局部最优 | 计划静态、无中途纠错 | 调用成本成倍、延迟高、提示工程复杂 |
| **适用场景** | 探索性、需外部工具输入的任务（首选） | 逻辑路径确定、内部推理密集的任务 | 对准确性和可靠性要求极高的场景 |

> 📋 **表格说明**：原作的表 4.1（不同 Agent Loop 的选择策略）以图片呈现，此处按 4.2.4 / 4.3 / 4.4.5 / 4.5 正文描述整理重构

下一章预告：已掌握构建单个智能体的核心技术，下一章将探索**不同低代码平台**的使用方式以及轻代码构建 Agent 的方案。

> 💡 **理解技巧**：三范式不是互斥选择而是**可组合的积木**——Reflection 的执行阶段"使用我们熟悉的方法（如 ReAct 或 Plan-and-Solve）"，意味着完全可以用 ReAct 产出初稿、再用 Reflection 迭代打磨

---

## 🔑 核心要点总结

1. **三范式是"思考与行动如何组织"的三种答案**：ReAct 边想边做（T-A-O 步进循环）、Plan-and-Solve 先谋后动（规划/执行两阶段解耦）、Reflection 事后校正（执行-反思-优化迭代循环）
2. **ReAct 的闭环三件套**：格式规约提示词（Thought/Action/Finish）+ 正则解析器 + history 回填；工具三要素中**描述最关键**（LLM 靠它选工具）；`max_steps` 是防死循环安全阀
3. **Plan-and-Solve 的关键工程决策**：计划强制输出为 **Python 列表**（`ast.literal_eval` 安全解析，比自然语言解析稳定）；Executor 的本质是**状态管理**（history 字符串逐步传递结果）；`PlanAndSolveAgent` 体现"组合优于继承"
4. **Reflection 的三角色提示词协同**：执行者（写初稿）→ 评审员（"极其严格"地挑刺，四维度评估）→ 优化者（按反馈修订）；`Memory` 短期记忆存储轨迹；**"无需改进"触发终止**
5. **工程共性**：提示词的格式规约是解析稳定的前提；所有循环都需要最大步数兜底；解析失败要返回兜底值而非崩溃
6. **选型逻辑**：探索性/需外部工具 → ReAct；逻辑确定/推理密集 → Plan-and-Solve；质量极重要且不计成本 → Reflection（"以成本换质量"）

## 📌 考试速记版

**三范式一句话对比：**

- ReAct = **侦探**：看线索（Observation）边查边想
- Plan-and-Solve = **建筑师**：先画蓝图（Plan）再施工（Solve）
- Reflection = **作家改稿**：初稿 → 评审 → 修订，直到"无需改进"

**关键数字与枚举：**

- ReAct：3 特点（可解释/动态纠错/工具协同）、4 局限（依赖LLM/串行低效/提示词脆弱/局部最优）、5 调试技巧
- Reflection：3 成本（调用开销/延迟/提示工程）、2 收益（质量跃迁/鲁棒性）、评审员 4 评估维度（事实/逻辑/效率/遗漏）
- 工具 3 要素：名称/描述/执行逻辑；`search` 智能解析 4 级优先级（answer_box → 知识图谱 → 前三条摘要）

**关键公式：**

- ReAct：$(th_t,a_t)=\pi(q,(a_1,o_1),\ldots,(a_{t-1},o_{t-1}))$，$o_t=T(a_t)$
- P&S：$P=\pi_{\text{plan}}(q)$，$s_i=\pi_{\text{solve}}(q,P,(s_1,\dots,s_{i-1}))$
- Reflection：$F_i=\pi_{\text{reflect}}(\text{Task},O_i)$，$O_{i+1}=\pi_{\text{refine}}(\text{Task},O_i,F_i)$

**关键正则/解析：**

- `Thought:\s*(.*?)(?=\nAction:|$)` 和 `Action:\s*(.*?)$`（都需 `re.DOTALL`）
- 工具调用：`(\w+)\[(.*)\]`；**Finish 判断必须先于工具解析**
- 计划解析：`split("```python")` + `ast.literal_eval`（不用 `eval`）

**易混淆对比：**

- ReAct 纠错靠**外部** Observation vs Reflection 纠错靠**内部**评审员
- P&S 计划**静态**（稳定但不灵活）vs ReAct 每步**动态**调整（灵活但可能局部最优）
- ReAct/P&S 的"记忆"是 history 字符串 vs Reflection 的 `Memory` 带类型标签（execution/reflection）

**记忆口诀**：边想边做 ReAct，先谋后动 P&S，事后挑刺 Reflection；描述定工具，列表定计划，记忆定反思。
