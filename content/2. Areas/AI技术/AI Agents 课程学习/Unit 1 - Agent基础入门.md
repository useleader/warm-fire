---
title: Unit 1 - Agent基础入门
publish: true
course_url: https://hf.co/learn/agents-course/unit1/introduction
---

最近开始系统学习 Hugging Face 官方出品的 [AI Agents 课程](https://huggingface.co/learn/agents-course/)，这是 Unit 1 的学习笔记，覆盖了 Agent 最核心的基础概念：什么是 Agent、LLM 如何驱动 Agent、工具（Tools）的机制、ReAct 循环，以及不同类型 Agent 的对比。笔记保留了我学习过程中的疑问和思考，作为整理思路的方式。

## 课程章节清单

- [ ] 介绍
- [ ] 什么是 Agent？
- [ ] 快速测验 1
- [ ] 什么是 LLM？
- [ ] 消息与特殊 Token
- [ ] 什么是工具（Tools）？
- [ ] 快速测验 2
- [ ] 理解 AI Agent：Thought-Action-Observation 循环
- [ ] 思考（Thought）、内部推理与 ReAct 方法
- [ ] 行动（Actions）：让 Agent 与环境交互
- [ ] 观察（Observe）：整合反馈以反思与适应
- [ ] Dummy Agent Library
- [ ] 使用 smolagents 创建第一个 Agent
- [ ] Unit 1 最终测验
- [ ] 总结

## 核心概念

### Agent 是什么

Agent 是能够自主执行任务、与环境交互的 AI 系统。与单纯的 LLM 不同，Agent 具备使用工具（Tools）、观察反馈（Observation）、调整策略的能力。核心区别在于：LLM 只能生成文本，而 Agent 能够**行动**。

### LLM 基础：模型家族、注意力与训练

**模型家族树**

基于 Transformer 架构的模型可大致分为 Encoder-Only、Decoder-Only 和 Encoder-Decoder 三类。Encoder-Only 模型（如 BERT）适合分类等理解任务，只需输出概率分布，不需要逐 token 生成。

课程中提及的模型包括 DeepSeek、GPT-4、Llama 3（Meta）、Gemma（Google）、Mistral 等。不过课程内容肯定有滞后性，更值得关注的是近年涌现的国产模型：Qwen、Minimax、GLM、豆包（Doubao）等，在多模态领域表现突出。

**注意力跨度（Attention Span）**

LLM 在生成当前 token 时，需要回看前文以理解上下文。Attention Span 指的就是模型能有效回溯并提取信息的最大 token 距离范围，即我们常说的上下文长度。

**预训练、自监督与微调**

预训练本质上是无监督学习，模型在海量文本上通过自监督方式学习语言的深层规律。自监督不需要人工标注，模型从数据本身的结构中学习——掩码语言建模（MLM）和自回归语言建模都是自监督的具体方式。

完成预训练后，模型在监督学习目标上进行微调（SFT），即用明确的输入输出对训练。此外还有 RLHF、DPO、PPO 等强化学习方法进一步对齐。

一个值得思考的问题：预训练后的模型只能随意输出，没有经过系统性约束，表现其实很差。真正好用的模型需要 SFT + RLHF 等多阶段对齐。

**不受限制的大模型**

仅靠 Prompt 只能绕过浅层防御。真正”不受限”的模型需要在微调阶段改造——使用正交化技术或”纯净”数据集进行反向 DPO 微调，在不损失智能效果的前提下解除安全对齐。

### 消息格式与 Chat Template

**特殊 Token 与 EOS**

每个 LLM 都有特定的特殊 token，用于标记序列、消息或响应的开始与结束。最重要的一个是 EOS（End of Sequence），标志着生成终止。

**Chat Template 与上下文管理**

LLM 通过 Chat Template 构建生成内容。模板中嵌入特殊 token 来分隔角色（system/user/assistant）。用户输入的特殊字符如果过滤不彻底，可能被模型当作控制 token 解析——这是 prompt injection 的常见攻击面。

对话历史通过 Chat Template 保存，维持多轮对话的连贯性。当对话过长时，Agent 框架会在底层执行压缩：有的自动删减最早对话，有的将其总结为摘要。

**系统消息（System Messages）**

系统提示作为持久指令定义模型行为，放在对话开头传输（因此这部分通常有缓存）。在 Agent 场景中，系统消息还携带可用工具的描述、行动格式指令和思考分段指南。

**一个有意思的细节：为什么是 “You are” 而不是 “I am”？**

LLM 预训练和微调时，数据结构通常划分为 User 和 Assistant 两个角色。System 是第三方视角向 Assistant 下达指令，”You are” 是 System 在对模型说话，避免模型混淆指令与自身生成内容。这个细节让我意识到 Prompt 设计的每一处都有训练数据结构的”历史包袱”。

**基础模型 vs 指令模型**

基础模型只完成预训练，做的事仅仅是预测下一个 token，没有明确的指令遵循能力。指令模型经过专门的指令微调，能理解和遵循对话格式。想让基础模型表现得像指令模型，需要用模型能理解的方式格式化 prompt。

### 工具（Tools）

Agent 的关键能力在于执行 Action，而 Action 需要通过 Tool 来实现。Tool 本质上是赋予 LLM 的函数，每个工具都有明确的目标。

**常用工具类型**

- **Web Search** — 联网搜索
- **图像生成** — 调用外部画图工具（而非模型本身具备多模态能力）
- **信息检索（RAG）** — 外接知识库，检索增强生成
- **API 接口** — 与外部服务交互，类似于 MCP Server 的前身

能力的演化路径很有意思：先有这些基础能力，再演化出更多样的 MCP Server，进而发展出 Skill 等更高层抽象。

**工具的工作原理**

Agent 负责解析 LLM 的输出，识别工具调用需求，执行工具调用，再将结果返回给 LLM。在 ReAct 循环中，Action 和 Observation 由 Agent 执行，Reasoning 由 LLM 完成。

工具调用的输出是一种独立的消息类型，调用步骤通常对用户不可见。工具描述通过 system prompt 以结构化方式（JSON 或编程语言）传递给 LLM，包含：工具名称、功能描述、输入参数和输出类型。本质上相当于黑盒使用——LLM 不需要知道工具内部实现。

**Python 自省与工具定义**

Python 自省（Introspection）能让代码在运行时读取自身内部结构。Agent 框架利用这一特性，自动将 Python 函数转化为给 LLM 看的工具说明。只需满足三个条件：

1. 使用类型注解（Type Hints）
2. 编写文档字符串（`”””...”””`，能被自省读取，而 `#` 注释不能）
3. 采用见名知意的函数命名

配合 `@tool` 装饰器，最终形式如下：

```python
@tool
def fun_name(a: int, b: int) -> int:
    “””Adds two numbers.”””
    return a + b

print(fun_name.to_string())
```

`to_string()` 用于呈现工具的展示信息（名称、描述、参数、输出），供 LLM 理解，并非工具本身的一部分。

通用的 Tool 类，可在需要时重复使用：

```python
class Tool:
    """
    A class representing a reusable piece of code (Tool).
    
    Attributes:
        name (str): Name of the tool.
        description (str): A textual description of what the tool does.
        func (callable): The function this tool wraps.
        arguments (list): A list of argument.
        outputs (str or list): The return type(s) of the wrapped function.
    """
    def __init__(self, 
                 name: str, 
                 description: str, 
                 func: callable, 
                 arguments: list,
                 outputs: str):
        self.name = name
        self.description = description
        self.func = func
        self.arguments = arguments
        self.outputs = outputs

    def to_string(self) -> str:
        args_str = “, “.join([
            f”{arg_name}: {arg_type}” for arg_name, arg_type in self.arguments
        ])
        return (
            f”Tool Name: {self.name},”
            f” Description: {self.description},”
            f” Arguments: {args_str},”
            f” Outputs: {self.outputs}”
        )

    def __call__(self, *args, **kwargs):
        return self.func(*args, **kwargs)
```

**MCP（Model Context Protocol）**

MCP 是模型上下文协议，提供了统一的工具接口，规范了应用程序向 LLM 提供工具的方式。它解决了三个问题：

- 不断增加的预构建集成列表，LLM 可直接接入
- LLM 提供商与供应商之间的切换能力
- 统一标准下的数据安全最佳实践

### Thought-Action-Observation 循环与 ReAct 方法

Agent 的运行核心是一个闭环：Thought（思考）→ Action（行动）→ Observation（观察）。

**Thought（思考）**

Thought 代表 Agent 解决任务的内部推理与规划过程，利用 LLM 能力分析当前信息，制定下一步策略。Thought 负责处理 Observation 的结果，将复杂问题分解为更小、可管理的步骤。

常见的思维模式：

| 思维类型 | 示例 |
|---------|------|
| Planning（规划） | “I need to break this task into three steps: 1) gather data, 2) analyze trends, 3) generate report” |
| Analysis（分析） | “Based on the error message, the issue appears to be with the database connection parameters” |
| Decision Making（决策） | “Given the user’s budget constraints, I should recommend the mid-tier option” |
| Problem Solving（问题解决） | “To optimize this code, I should first profile it to identify bottlenecks” |
| Memory Integration（记忆整合） | “The user mentioned their preference for Python earlier, so I’ll provide examples in Python” |
| Self-Reflection（自我反思） | “My last approach didn’t work well, I should try a different strategy” |
| Goal Setting（目标设定） | “To complete this task, I need to first establish the acceptance criteria” |
| Prioritization（优先级排序） | “The security vulnerability should be addressed before adding new features” |

一些高级 Agent 框架还引入了 Reflexion 和 Memory 等机制来增强思考能力。这让我想到人类学习中的反思总结与灵感迸发——这些是否也能在 Agent 中体现？

**ReAct 方法：Reasoning + Acting**

ReAct 是将推理与行动结合的提示技术。原理出奇的简单：在让 LLM 解码后续 token 前，添加 “Let’s think step by step” 的提示，引导模型逐步思考，生成计划而非直接输出最终方案。一些模型经过专门微调，被训练为”先思考再回答”。

需要注意的是，对于经过 function-calling 微调的 LLM，显式的思维过程是可选的——这类模型天生就能精准输出符合工具参数规范的 JSON。

**Action（行动）**

行动让 Agent 与环境交互。在许多 Agent 框架中，规则和指南直接嵌入到系统提示中，确保每个循环遵循定义的逻辑。

**Observation（观察）**

观察阶段 Agent 收集反馈、附加结果、调整策略。这种迭代式反馈整合确保 Agent 始终保持与目标的动态对齐，根据现实结果不断学习和调整。

关于循环的终止：虽然最终判断是 Thought 阶段做出的，但 Observation 阶段的策略调整才是关键——它决定了下一轮 Thought 的输入质量。实际部署中通常将 Thought 放在最后一步。

### Code Agent vs JSON Agent

Agent 执行动作的方式分为三种类型：

**JSON Agent** — 动作以 JSON 格式指定，结构化强但表达能力有限。

**Code Agent** — Agent 生成可执行的代码块（通常 Python），由外部解释器执行。相比 JSON Agent 有几个优势：
1. **表达力更强** — 代码比 JSON 更灵活
2. **模块化和可重用** — 生成的代码可包含可复用的函数和模块
3. **更易调试** — 编程语法明确定义，错误更易检测和纠正
4. **直接集成** — 可直接与外部库和 API 交互

**Function-Calling Agent** — JSON Agent 的子类别，经微调后为每个动作生成精确的函数调用消息。这类模型在训练时看过大量 API 调用样例，不需要额外规则就能精准输出符合参数规范的 JSON，准确率高，且 ReAct 的文本推理过程也可以省略。

**Stop and Parse 方法**

所有 Agent 类型都依赖这套方法确保输出结构化与可预测：

1. **结构化格式生成** — Agent 以预定义的清晰格式（JSON 或代码）输出预期动作
2. **停止进一步生成** — 动作完成后立即停止生成新 token，防止额外或错误的输出
3. **解析输出** — 外部解析器读取格式化的动作，确定调用哪个工具，提取所需参数

LLM 只处理文本，用它描述要采取的动作和工具参数——实际的执行和解析由 Agent 框架完成。

关于安全性：Stop and Parse 方法虽然保证了格式规范，但如果参数验证不严格，恶意构造的输入仍可能通过 prompt injection 导致意外行为。参数错误会作为 Observation 的一部分返回给模型，形成反馈闭环。

## 代码实践

### Dummy Agent Library 实验

课程首先引入了 Serverless API 的概念——无需安装和部署，直接在云端进行模型推理。

实验过程中的几个观察：

- Agent 需要"停在观察阶段"才会真正执行函数
- 停在观察后，输出的内容是函数的原始输出，而非经过 LLM 理解的内容
- 新的 prompt 直接将函数输出拼接到对话中——Observation 本质上就是把输出拼起来重新进入 Thought

这让我更清楚地理解了：Observation 不是一个复杂的智能过程，而是将工具执行结果注入回对话上下文的机制。

### 第一个 smolagents Agent

（待补充实践细节）

## 思考与疑问

- 人类学习中的反思总结与灵感迸发，在 Agent 中是否已经有对应的机制？（部分已被 Reflexion、Memory 等覆盖）
- 最终循环终止的判断在 Thought 阶段还是 Observation 阶段？实际部署中通常 Thought 在最后一步
- "You are" vs "I am" 的选择源于训练数据中的角色结构——Prompt 设计的每一处都有历史包袱
- 关于不受限大模型：如何在微调阶段抹除安全对齐而不损失模型智能效果？正交化技术和反向 DPO 是目前的方向
- Agent 的能力演化路径：基础工具 → MCP Server → Skill，这一抽象层次提升值得持续关注
- 有没有一种方式让 LLM 自己去探索和搜索可用工具，而不是被动接收工具列表？
