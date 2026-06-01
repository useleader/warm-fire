---
title: Unit 2.3 - LangGraph 框架
publish: false
status: 🟢 已完成
course_url: https://hf.co/learn/agents-course/unit2/langgraph/introduction
publish: true
---

LangGraph 是 Unit 2 的重头戏。如果说 Unit 1 讲的是 Agent 的"思想"（ReAct 循环、工具调用），那 Unit 2 讲的就是 Agent 的"骨架"——如何把多个步骤、多种能力以可控的方式组织起来。这篇笔记覆盖了 LangGraph 的设计哲学、核心构建块（State/Nodes/Edges）、条件路由机制、以及两个完整的实践案例（邮件分类和文档分析）。

## 课程章节清单

- [x] LangGraph 介绍
- [x] 什么是 LangGraph？何时使用？（vs smolagents / 手写 Python）
- [x] LangGraph 的构建模块（State / Nodes / Edges / Graph）
- [x] 构建第一个 LangGraph（Alfred 邮件分类系统）
- [x] 文档分析 Graph（Alfred 文档分析 + ReAct 模式）
- [x] 快速测验 1
- [x] 总结

## 核心概念

### LangGraph 的设计哲学：图即流程

LangGraph 最核心的想法其实非常直白：**把 Agent 的执行流程建模为一张有向图（directed graph）**。每个节点（Node）是一个处理步骤，每条边（Edge）定义了步骤之间的流转方向，而 State 则像一条传送带，把信息从一个节点带到下一个。

这和我之前接触到的其他 Agent 框架形成了鲜明对比。smolagents 和 LlamaIndex 更多是"隐式"的控制流——你定义工具、给模型一个系统提示、然后 invoke，模型内部自己决定调用什么工具、以什么顺序。LangGraph 反过来，它要求你**显式地**画出每一步，明确定义"如果 A 则去 B，否则去 C"。

```python
# LangGraph 的思路：你先画好流程图，再让 LLM 填充具体内容
# 而不是让 LLM 自己决定流程
```

这个设计选择反映了 LangChain 团队的一个判断：在 production 环境中，**可预测性比灵活性更重要**。

### Control vs Freedom 光谱

课程里用一个很清晰的框架来理解不同 Agent 框架的定位：

| 维度 | smolagents（Code Agent） | 传统 JSON Agent | LangGraph |
|------|------------------------|-----------------|-----------|
| 自由度 | 高 — 可自建工具、多步调用 | 中 — JSON 格式约束 | 低 — 图结构预定义 |
| 可预测性 | 低 — 行为难以完全预期 | 中 — JSON 格式可 parse | 高 — 执行路径明确 |
| 适用场景 | 探索性任务、原型开发 | 标准化 API 调用 | 生产级、多步骤、有状态 |
| 控制点 | 系统提示 | 格式约束 + Prompt | 边 + 条件 + State |

我自己的理解是：**框架的选择本质上是在"让模型自由发挥"和"让开发者掌控流程"之间做权衡**。smolagents 的 Code Agent 可以同时调用多个工具、动态生成代码，这对于探索性任务很爽，但一旦上了生产，你很难保证它每次都走同样的逻辑。LangGraph 则相反——你作为开发者，先设计好流程图，LLM 只在特定节点内发挥作用。

LangGraph 官方文档提到它特别适合：
- **多步推理流程**（multi-step reasoning）——每一步需要显式控制
- **需要跨步骤保持状态**的应用——对话历史、积累的上下文
- **确定性逻辑 + AI 能力的混合**——if-else + LLM 调用
- **Human-in-the-loop**——需要人工介入的工作流
- **复杂 Agent 架构**——多个组件协同工作

### 和手写 Python 相比，LangGraph 好在哪？

这是一个很自然的问题：我完全可以用 if-else + Python 函数来实现同样的流程，为什么需要 LangGraph？

课程给出了几个关键理由：
1. **State 的显式管理** — LangGraph 的 State 是有类型的、结构化的，而不是散落在各个函数参数和全局变量中
2. **内置可视化** — 编译后的 Graph 可以调用 `get_graph().draw_mermaid_png()` 直接出图
3. **日志和追踪** — 天然支持 Langfuse 等可观测性工具
4. **Human-in-the-loop** — 内置的中断/恢复机制
5. **Checkpointing** — 每个步骤的状态都可以持久化，支持容错和恢复

> 我注意到，这些优势本质上都是在解决一个核心问题：**当 Agent 流程变复杂时，如何保持对执行状态的可观测性和可控制性**。如果你的 Agent 只有两三个步骤，手写 Python 完全够用；当步骤超过 5 个、有分支循环、需要人工介入时，LangGraph 的价值才真正体现出来。

### 构建模块一：State（状态）

State 是 LangGraph 中最核心的概念。它的定义方式非常 Pythonic——直接使用 `TypedDict`：

```python
from typing_extensions import TypedDict
from typing import Optional

class State(TypedDict):
    """LangGraph 的 State 是任意 TypedDict"""
    graph_state: str
    counter: int
    metadata: Optional[dict]
```

关键理解：**State 不是隐式的，它是你需要精心设计的**。每个字段都应该有明确的目的，要承载所有决策需要的信息。

课程中引入了更有趣的用法——通过 `Annotated` 类型加上 reducer 函数来控制状态字段的更新方式：

```python
from typing import Annotated
from langgraph.graph.message import add_messages
from langchain_core.messages import AnyMessage

class AgentState(TypedDict):
    messages: Annotated[list[AnyMessage], add_messages]  # 追加而非覆盖
    input_file: Optional[str]
```

这里 `add_messages` 是一个 reducer 函数，它的作用是：当多个节点都想更新 `messages` 字段时，不是"后一个覆盖前一个"，而是"追加到列表中"。这是一个非常重要的概念——**State 的更新策略可以在字段级别定制**，默认是 override，但你可以换成 append、merge 或其他自定义逻辑。

### 构建模块二：Nodes（节点）

Nodes 就是普通的 Python 函数。每个节点接收 State，返回 State 的增量更新：

```python
def node_1(state):
    print("--- Node 1 ---")
    # 返回字典，LangGraph 会将这些字段 merge 到当前 State
    return {"graph_state": state['graph_state'] + " I am"}
```

```python
def node_2(state):
    print("--- Node 2 ---")
    return {"graph_state": state['graph_state'] + " happy!"}
```

节点里可以包含：
- **LLM 调用** — 生成文本或决策
- **工具调用** — 与外部系统交互
- **条件逻辑** — 决定下一步
- **人工介入** — 暂停流程等待用户输入

LangGraph 预定义了 `START` 和 `END` 两个特殊节点，分别表示图的入口和终止点。

### 构建模块三：Edges（边）

边定义节点之间的连接。LangGraph 提供了两种边：

**直接边（Direct Edge）**——无条件转移，执行完节点 A 后一定去节点 B：

```python
from langgraph.graph import START, END

graph.add_edge(START, "node_1")      # 图入口 → node_1
graph.add_edge("node_1", "node_2")   # node_1 → node_2
graph.add_edge("node_2", END)        # node_2 → 图出口
```

**条件边（Conditional Edge）**——根据当前 State 动态决定去向：

```python
from typing import Literal

def decide_mood(state) -> Literal["node_2", "node_3"]:
    """根据 state 的值决定走哪条分支"""
    if "happy" in state['graph_state']:
        return "node_2"
    else:
        return "node_3"

graph.add_conditional_edges(
    "node_1",
    decide_mood,
    {
        "node_2": "node_2",    # 如果函数返回 "node_2"，就去 node_2
        "node_3": "node_3"     # 如果函数返回 "node_3"，就去 node_3
    }
)
```

这里 `decide_mood` 的返回值是一个字符串，必须和映射字典中的 key 对应。映射字典的 value 是目标节点的名称。

### 构建模块四：Graph 编译

所有组件定义好后，最后一步是编译：

```python
from langgraph.graph import StateGraph

# 1. 创建图，绑定 State 类型
builder = StateGraph(State)

# 2. 添加节点
builder.add_node("node_1", node_1)
builder.add_node("node_2", node_2)
builder.add_node("node_3", node_3)

# 3. 添加边
builder.add_edge(START, "node_1")
builder.add_conditional_edges("node_1", decide_mood, {
    "node_2": "node_2",
    "node_3": "node_3"
})
builder.add_edge("node_2", END)
builder.add_edge("node_3", END)

# 4. 编译
graph = builder.compile()

# 5. 可视化
graph.get_graph().draw_mermaid_png()

# 6. 运行
result = graph.invoke({"graph_state": "Hi, this is Lance."})
# 输出: "Hi, this is Lance. I am sad!" (或 happy, 取决于随机路由)
```

编译这一步会把你的图"冻结"——检查所有边是否连接有效、是否有孤立的节点。编译后的 graph 才能被 invoke。

### 条件路由：动态决策的两种模式

条件路由本质上分为两类：

1. **确定性路由** — 基于 State 中的某个字段做 if-else 判断，不涉及 LLM。例如：根据 `is_spam` 字段决定走 spam 分支还是 legitimate 分支。
2. **LLM 驱动的路由** — 路由函数的判断逻辑内部调用 LLM，让模型决定下一步怎么走。这在更复杂的 Agent 中有应用。

值得思考的是：**条件路由的设计决定了你的 Agent 的"决策点"在哪**。如果你把决策都放在 LLM 节点内部，那 LangGraph 的优势就没那么明显了。真正发挥 LangGraph 威力的是：把你可以确定的逻辑写成硬编码的边，把需要判断的交给 LLM——两者的混合。

### State 管理与持久化

课程对 State 管理的讨论集中在几个层面：

- **消息积累**：通过 `add_messages` reducer 实现消息列表的追加而非覆盖
- **跨节点传递**：State 自动在节点间传递
- **跟踪和调试**：配合 Langfuse 等工具，可以看到每个步骤的 State 快照

LangGraph 还支持更高级的特性（虽然课程没有深入，但值得了解）：
- **Checkpointer** — 在每个步骤后保存 State 快照，支持从任意点恢复
- **Thread ID** — 区分不同对话会话的状态
- **Interrupt** — 在特定节点暂停执行，等待人工输入后再继续

### 与 smolagents 和 LlamaIndex 的对比

我自己的理解，这三个框架代表了三种不同的设计哲学：

| 框架 | 设计哲学 | 核心抽象 | 典型用户 |
|------|---------|---------|---------|
| **smolagents** | Code-First | 代码生成 + 执行 | 想要最大灵活性的研究者 |
| **LlamaIndex** | Data-First | 索引 + 检索 + 代理 | 以 RAG 为核心的应用开发者 |
| **LangGraph** | State-Machine-First | 图 + State + 边 | 需要确定性流程的生产系统 |

**smolagents** 的 Code Agent 会生成 Python 代码并由沙箱执行——这赋予了它极强的表达能力，但也让行为更不可预测。它的"控制"来源是系统提示和代码沙箱。

**LlamaIndex** 围绕数据索引展开，Agent 能力是在检索基础上的附加层。它的 Query Engine 模式在某些场景比 LangGraph 更简单直接。

**LangGraph** 的核心洞察是：**许多 AI 应用的逻辑其实可以分成"确定的部分"和"不确定的部分"**。确定的部分（流程、条件分支）用图结构来表达，不确定的部分（判断、生成）交给 LLM 在节点内部完成。这种分离让系统既可控又智能。

> 我个人的感觉是：LangGraph 的学习曲线比 smolagents 更陡，但一旦你掌握了图思维，它提供的控制粒度是其他框架很难比拟的。特别是当你的 Agent 需要处理多个步骤、多个工具、多个人工介入点时，LangGraph 的图模型显得格外优雅。

## 代码实践

### 实践一：邮件分类系统（Alfred 的基础版）

这是课程中的第一个完整示例。场景是 Alfred 管家处理布鲁斯·韦恩的邮件。

**1. 定义 State：**

```python
from typing import TypedDict, List, Dict, Any, Optional

class EmailState(TypedDict):
    email: Dict[str, Any]         # 包含 subject, sender, body
    email_category: Optional[str] # inquiry / complaint / 等
    spam_reason: Optional[str]    # 垃圾邮件原因
    is_spam: Optional[bool]       # 分类结果
    email_draft: Optional[str]    # 草稿回复
    messages: List[Dict[str, Any]] # 与 LLM 的对话记录
```

**2. 定义节点函数：**

```python
from langchain_openai import ChatOpenAI
from langchain_core.messages import HumanMessage

model = ChatOpenAI(temperature=0)

def read_email(state: EmailState):
    """读取并记录邮件（简单的预处理节点）"""
    email = state["email"]
    print(f"Processing email from {email['sender']}: {email['subject']}")
    return {}  # 不修改 State

def classify_email(state: EmailState):
    """用 LLM 判定是否为垃圾邮件"""
    email = state["email"]
    prompt = f"""Analyze this email and determine if it is spam or legitimate.

From: {email['sender']}
Subject: {email['subject']}
Body: {email['body']}

If spam, explain why. If legitimate, categorize it (inquiry, complaint, etc)."""

    response = model.invoke([HumanMessage(content=prompt)])
    response_text = response.content.lower()
    is_spam = "spam" in response_text and "not spam" not in response_text

    # 简化处理：实际项目需要更健壮的解析逻辑
    spam_reason = None
    if is_spam and "reason:" in response_text:
        spam_reason = response_text.split("reason:")[1].strip()

    email_category = None
    if not is_spam:
        categories = ["inquiry", "complaint", "thank you", "request", "information"]
        for cat in categories:
            if cat in response_text:
                email_category = cat
                break

    return {
        "is_spam": is_spam,
        "spam_reason": spam_reason,
        "email_category": email_category,
    }

def handle_spam(state: EmailState):
    print(f"Spam detected. Reason: {state['spam_reason']}")
    print("Moved to spam folder.")
    return {}

def draft_response(state: EmailState):
    email = state["email"]
    prompt = f"""Draft a polite preliminary response to this email.

From: {email['sender']}
Subject: {email['subject']}
Body: {email['body']}

Category: {state.get('email_category', 'general')}"""

    response = model.invoke([HumanMessage(content=prompt)])
    return {"email_draft": response.content}

def notify_mr_hugg(state: EmailState):
    print(f"\nSir, you have an email from {state['email']['sender']}.")
    print(f"Draft: {state['email_draft']}\n")
    return {}
```

**3. 定义路由逻辑：**

```python
def route_email(state: EmailState) -> str:
    """条件路由：根据分类结果决定去向"""
    if state["is_spam"]:
        return "spam"
    else:
        return "legitimate"
```

**4. 构建并编译图：**

```python
from langgraph.graph import StateGraph, START, END

# 创建图
builder = StateGraph(EmailState)

# 添加节点
builder.add_node("read_email", read_email)
builder.add_node("classify_email", classify_email)
builder.add_node("handle_spam", handle_spam)
builder.add_node("draft_response", draft_response)
builder.add_node("notify_mr_hugg", notify_mr_hugg)

# 连接边
builder.add_edge(START, "read_email")
builder.add_edge("read_email", "classify_email")
builder.add_conditional_edges(
    "classify_email",
    route_email,
    {"spam": "handle_spam", "legitimate": "draft_response"}
)
builder.add_edge("handle_spam", END)
builder.add_edge("draft_response", "notify_mr_hugg")
builder.add_edge("notify_mr_hugg", END)

graph = builder.compile()
```

**5. 执行：**

```python
legitimate_email = {
    "sender": "john.smith@example.com",
    "subject": "Question about your services",
    "body": "I'm interested in your consulting services. Could we schedule a call?"
}

spam_email = {
    "sender": "winner@lottery-intl.com",
    "subject": "YOU HAVE WON $5,000,000!!!",
    "body": "Send us your bank details to claim your prize!"
}

result = graph.invoke({
    "email": legitimate_email,
    "is_spam": None,
    "spam_reason": None,
    "email_category": None,
    "email_draft": None,
    "messages": []
})
```

这个例子的关键启发是：**整个流程的"骨架"完全由开发者定义，LLM 只在 `classify_email` 和 `draft_response` 两个节点内部发挥作用**。路由逻辑 `route_email` 是完全确定性的（基于 `is_spam` 字段判断），不涉及 LLM。

### 实践二：文档分析 Agent（Alfred 的高级版）

第二个示例引入了 **ReAct 模式**（Reason-Act-Observe），这是 Agent 框架中最经典的设计模式。

**核心区别**：第一个示例的图是**线性的**（虽然有分支但从不回头），而文档分析 Agent 的图包含**循环**——Agent 不停调用工具，直到不需要为止。

```python
from langgraph.graph.message import add_messages
from langgraph.prebuilt import ToolNode, tools_condition
from langchain_core.messages import AnyMessage, SystemMessage, HumanMessage
from typing import Annotated, Optional
import base64

class AgentState(TypedDict):
    input_file: Optional[str]
    messages: Annotated[list[AnyMessage], add_messages]

# 工具定义：从图片提取文字
def extract_text(img_path: str) -> str:
    """Extract text from an image using GPT-4o vision."""
    with open(img_path, "rb") as f:
        img_b64 = base64.b64encode(f.read()).decode("utf-8")
    message = [HumanMessage(content=[
        {"type": "text", "text": "Extract all text from this image. Return only the text."},
        {"type": "image_url", "image_url": {"url": f"data:image/png;base64,{img_b64}"}}
    ])]
    vision_llm = ChatOpenAI(model="gpt-4o")
    return vision_llm.invoke(message).content

# 工具定义：简单计算
def divide(a: int, b: int) -> float:
    """Divide a by b."""
    return a / b

tools = [divide, extract_text]
llm_with_tools = ChatOpenAI(model="gpt-4o").bind_tools(tools, parallel_tool_calls=False)

# 助理节点
def assistant(state: AgentState):
    sys_msg = SystemMessage(
        content="You are Alfred, serving Mr. Wayne. "
                "You can analyze documents and run computations."
    )
    return {
        "messages": [llm_with_tools.invoke([sys_msg] + state["messages"])],
        "input_file": state["input_file"]
    }

# 构建图（包含循环！）
builder = StateGraph(AgentState)
builder.add_node("assistant", assistant)
builder.add_node("tools", ToolNode(tools))

builder.add_edge(START, "assistant")
builder.add_conditional_edges(
    "assistant",
    tools_condition,  # 如果 LLM 调用了工具 → tools 节点；否则 → END
)
builder.add_edge("tools", "assistant")  # 工具执行完回到 assistant，形成循环

graph = builder.compile()
```

这里的 `tools_condition` 是 LangGraph 预置的条件边函数，它的逻辑很简单：检查 LLM 的最新输出是否包含 tool_calls。如果有，路由到 `tools` 节点；如果没有，路由到 `END`。

循环的逻辑：
1. `assistant` 节点输出 → 可能包含 tool_calls
2. `tools_condition` 检查 → 有工具调用则去 `tools`，否则结束
3. `tools` 节点执行工具 → 结果追加到 messages
4. 回到 `assistant`（第 1 步），但 messages 里已经包含工具执行结果
5. LLM 看到工具结果，可能再调用下一个工具，也可能直接回答

> 这正是 ReAct 循环在 LangGraph 中的实现。与 smolagents 不同，这里循环的退出条件不是"模型决定停止"，而是通过 `tools_condition` 检测"模型没有调用工具"来判断。两者效果类似，但实现机制不同：smolagents 在代码解释器内部循环，LangGraph 在图层面实现循环。

执行示例——文档分析：

```python
messages = [HumanMessage(content="What should I buy for the dinner menu?")]
result = graph.invoke({
    "messages": messages,
    "input_file": "Batman_training_and_meals.png"
})
```

Agent 会：调用 `extract_text` → 读取图片中的文字 → 发现训练计划和菜单 → 提取购物清单 → 返回结果。

这个例子的表现力在于——`ToolNode` 自动处理了工具调用的分派和结果返回，开发者不需要写任何工具执行的逻辑。这就是 LangGraph Prebuilt 组件的威力。

### 可观测性集成

课程还介绍了如何集成 Langfuse 进行追踪。这是 production 部署的必备能力：

```python
from langfuse.langchain import CallbackHandler

langfuse_handler = CallbackHandler()

result = graph.invoke(
    input={...},
    config={"callbacks": [langfuse_handler]}
)
```

每个步骤的输入、输出、token 消耗、延迟都会被记录到 Langfuse 仪表板中。对于调试复杂的多步 Agent，这种追踪能力几乎是必需的。

## Quiz 记录

**Q1: LangGraph 的主要用途是什么？**
LangGraph 主要用于管理集成 LLM 的应用的控制流（control flow），通过有向图结构为多步推理工作流提供显式的编排能力。

**Q2: 在 Control vs Freedom 的光谱上，LangGraph 处于什么位置？**
LangGraph 偏向 Control 一端。它通过显式的图结构（预定义的节点和边）确保执行的可预测性，适合对流程有严格要求的 production 场景，而非探索性任务。

**Q3: State 在 LangGraph 中扮演什么角色？**
State 是图的核心数据载体，由用户通过 TypedDict 定义，包含所有需要在步骤间传递的信息。State 在节点间被不断读取和更新，并影响条件边的路由决策。State 的设计质量直接影响整个应用的可靠性和可维护性。

**Q4: 什么是 Conditional Edge（条件边）？**
条件边是一个由用户定义的函数，它读取当前 State 并根据判断结果返回一个目标节点名称。LangGraph 根据这个返回值将执行流转到不同的后续节点，实现动态分支。条件边是 LangGraph 将"决策"从 LLM 内部解耦出来的关键机制。

**Q5: LangGraph 如何帮助缓解 LLM 的幻觉问题？**
通过将执行流程显式结构化，LangGraph 减少了对 LLM 自主判断下一步行为的依赖。关键决策点由确定性逻辑或开发者定义的规则控制，LLM 只在受限的节点内部发挥作用，从而降低不可控生成的风险。本质上是用结构化的控制流来约束 LLM 的输出空间。

## 思考与疑问

**1. LangGraph 和 MCP 是什么关系？**

这是个让我琢磨了一段时间的问题。两者的定位不同：MCP 是工具层面的标准协议，解决的是"LLM 如何发现和调用外部工具"的问题；LangGraph 是流程层面的编排框架，解决的是"多个步骤如何组合"的问题。它们可以共存——在 LangGraph 的节点内部调用 MCP Server 的工具。实际上，把 MCP 的工具封装成 LangGraph 的 ToolNode 是一个很自然的集成方式。

**2. 图模型 vs 线性代码：思维方式的转变**

从写 if-else 的线性思维切换到图思维，需要一定的适应。在线性代码中，你"从上往下"控制流程；在图模型中，你定义节点和边，然后由框架决定的执行引擎来调度。这种反转意味着你不再控制"何时"执行，而是控制"什么条件下"执行。我觉得这是理解 LangGraph 最关键的一步——当你不再思考"先做 A 再做 B"，而是思考"做完 A 后，在条件 X 下去 B，条件 Y 下去 C"时，你的设计就上了一个台阶。

**3. 循环的实现与 ReAct 的关系**

文档分析 Agent 中的循环实际上是 ReAct 循环的图化表达。这与 smolagents 中通过 while 循环 + 代码执行的实现方式形成对比。LangGraph 将"循环"这种控制流结构以图的形态显式表达出来，这让调试和追踪变得更加透明——你可以在 Langfuse 中看到每一次循环的完整轨迹。我注意到一个微妙的区别：smolagents 的循环在代码执行引擎内部，开发者看不到每一次迭代的中间状态；LangGraph 的循环在图层面，每一次节点跳转都可以被观测和记录。

**4. Conditional routing 的粒度选择**

在邮件分类系统中，路由逻辑是确定性的（基于一个 bool 字段）。但更复杂的场景下，路由逻辑本身也可以调用 LLM——让模型决定下一步。我还没有完全想清楚这个设计边界：什么情况下应该用确定性路由，什么情况下应该让 LLM 来路由？目前的理解是：如果决策标准明确（如 is_spam 的 true/false），用确定性路由；如果决策需要语义理解（如"这个邮件的情绪是积极还是消极"），用 LLM 路由。关键原则是：**能用规则解决的问题，不要交给 LLM**。

**5. 与 Unit 1 的连接**

Unit 1 详细讲解了 ReAct 循环（Thought → Action → Observation → Thought...）。LangGraph 的文档分析 Agent 实际上就是 ReAct 的一个具体实现——但这里的"循环"是通过图的边来实现的，而不是通过代码循环。这让我看到一个有趣的现象：**同样的模式可以在不同抽象层次上以不同方式实现**。smolagents 在 Python 代码层面实现了 ReAct（while 循环 + try-except），LangGraph 在图结构层面实现了 ReAct（assistant → tools → assistant 的循环边）。殊途同归，但控制粒度和可观测性完全不同。

**6. add_messages 的设计意图**

`Annotated[list[AnyMessage], add_messages]` 这个类型注解让我困惑了好一阵。为什么不是直接覆盖而是追加？原因是在多轮对话场景中，每次 LLM 调用都需要看到完整的对话历史。如果每个节点都只返回自己的那部分消息而没有累积，上下文就会丢失。`add_messages` 本质上是一个 **merge 策略**，它告诉 LangGraph：当多个节点都想更新这个字段时，把新内容追加到现有列表的末尾，而不是替换。这和 React 的 `useState` 与 `setState` 的合并行为有类似的意图——只是 LangGraph 把这个策略提升到了框架层面。

**7. 关于"图太大"的问题**

我还有一个尚未解决的问题：当图变得非常大（几十个节点）时，管理和调试是不是也会变得困难？LangGraph 的可视化能力可以部分缓解这个问题，但我觉得当图达到一定复杂度后，可能需要图的分层和模块化——把子图封装成独立的组件。LangGraph 的 Subgraph 概念可能就是这个方向的答案，需要在后续的学习中进一步探索。

## 关键收获

1. **LangGraph 的本质是一个图执行引擎**，它将 Agent 的流程建模为 StateGraph，通过 State、Nodes、Edges 三个基础抽象提供了对执行路径的精确控制。它的核心优势不是"更智能"，而是"更可控"——这是一个非常重要的区别。

2. **State 是第一等公民**，需要精心设计。TypedDict 定义的结构化 State 不仅让类型安全得到保障，更重要的是，它强迫你提前想清楚每一步需要什么数据、产生什么数据——这种前置的思考对复杂 Agent 的开发至关重要。

3. **条件边是 LangGraph 表达力的关键**，它将"决策"从 LLM 内部剥离出来，交给开发者定义的逻辑。这与其他 Agent 框架（smolagents 把所有决策交给 LLM）形成了根本性的设计差异，也是在 production 环境中降低不可控风险的核心手段。

4. **循环图的引入**让 LangGraph 能够表达 ReAct 模式——通过"assistant → tools_condition → tools → assistant"的循环边，而非 while 循环。这是对 Agent 执行循环的另一种建模视角，将控制流的可见性从"黑盒执行"提升到了"白盒追踪"。

5. **生产就绪性**是 LangGraph 区别于其他框架的重要特征：内置的可视化、Langfuse 追踪集成、State 快照与恢复、以及 human-in-the-loop 支持，这些能力让从原型到部署的跨度变得更小。

6. **LangGraph 不是银弹**。对于简单的单步 Agent，smolagents 或直接调用 API 更轻量。LangGraph 的优势在流程复杂度达到一定阈值时才显现出来——这个阈值大约在"需要 3 个以上顺序步骤"或"需要条件分支"时。选框架前，想清楚你的 Agent 到底有多复杂。
