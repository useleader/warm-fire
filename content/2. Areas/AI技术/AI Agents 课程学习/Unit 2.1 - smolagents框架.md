---
title: Unit 2.1 - smolagents 框架
publish: true
status: 🟢 已完成
course_url: https://hf.co/learn/agents-course/unit2/smolagents/introduction
---

Hugging Face 出品的 smolagents 是一个轻量级 AI Agent 框架，设计哲学是"小而美"。与 LangGraph 的图编排、LlamaIndex 的数据管道风格不同，smolagents 走的是代码优先（code-first）路线——Agent 直接生成 Python 代码来执行动作，而不是 JSON。这个思路来自一篇有影响力的论文 [Executable Code Actions Elicit Better LLM Agents](https://huggingface.co/papers/2402.01030)，目前来看效果也确实更好。

本单元是 Unit 1 的延续，Alfred（韦恩庄园的管家）继续作为贯穿案例出现，这次他用 smolagents 框架来筹备一场超级英雄主题派对。通过这个案例，课程把 smolagents 的核心概念串了起来。

## 课程章节清单

- [x] smolagents 介绍
- [x] 为什么选择 smolagents？
- [x] 快速测验 1
- [x] 构建使用代码的 Agent（Code Agents）
- [x] 编写动作：代码片段 vs JSON 块（Tool Calling Agents）
- [x] 工具（Tools）
- [x] 检索 Agent（Retrieval Agents）
- [x] 快速测验 2
- [x] 多 Agent 系统（Multi-Agent Systems）
- [x] 视觉与浏览器 Agent（Vision & Browser Agents）
- [x] 最终测验
- [x] 总结

## 核心概念

### smolagents 设计哲学

smolagents 的核心理念可以总结为三点：

**1. 轻量化。** 整个框架的核心代码大约只有 1000 行（对比 LangGraph 和 LlamaIndex 的代码量要小得多）。它不是想做一个大而全的框架，而是提供一个最小可用的抽象层，让开发者能快速上手。

**2. Hugging Face 原生。** 模型集成深度绑定了 Hugging Face 生态——`InferenceClientModel` 调用 Serverless Inference API，工具可以通过 Hub 分享和加载，甚至可以把 Hugging Face Space 直接当作工具来用。如果已经使用了 HF 生态，集成成本非常低。

**3. 代码优先（Code-First）。** 这是 smolagents 最鲜明的设计选择。默认的 Agent 类型是 `CodeAgent`，它生成 Python 代码来调用工具，而不是生成 JSON 让框架去解析。后面会展开讨论这个选择背后的逻辑。

模型集成方面，smolagents 提供了多种选择：

| 模型类 | 适用场景 |
|--------|---------|
| `TransformersModel` | 本地 transformers pipeline |
| `InferenceClientModel` | HF Serverless API / 第三方推理供应商 |
| `LiteLLMModel` | 轻量级模型交互 |
| `OpenAIServerModel` | OpenAI 兼容接口 |
| `AzureOpenAIServerModel` | Azure OpenAI 部署 |

这种灵活的设计意味着你可以轻松切换底层模型，从本地模型到云端 API 都支持。

### Code Agent：代码即行动

Code Agent 是 smolagents 的主角。它不是一个普通的 Agent 实现，而是整框架的设计重心。

**为什么代码优于 JSON？**

在多步 Agent 流程中，LLM 需要生成操作步骤并调用工具。传统做法是让 LLM 输出 JSON 块，框架再去解析：

```json
{"name": "web_search", "arguments": {"query": "Gotham catering"}}
```

JSON 的问题是：
1. **表达能力有限** — 如果想把多次搜索结果合并、做条件判断、循环搜索，JSON 表达起来非常笨拙
2. **不可组合** — JSON 碎片无法复用，每次调用都是独立的
3. **对象管理困难** — 无法在步骤间传递复杂对象（如图片、数据框）

而代码天然解决这些问题。

课程的论文 [Executable Code Actions Elicit Better LLM Agents](https://huggingface.co/papers/2402.01030) 给出了四个关键论点：

- **Composability（可组合性）**：代码可以定义变量、写循环、组合函数调用，把多步操作打包成可复用的逻辑块。比如用 for 循环搜索多个关键词，这在 JSON 方案里无法优雅实现。
- **Object Management（对象管理）**：代码可以直接操作 Python 对象。Agent 生成的代码在外部 Python 解释器中执行，图片、数据框等复杂结构直接存在于内存中，可以被后续步骤访问和操作。
- **Generality（通用性）**：图灵完备的表达能力。代码可以表达任何计算上可能的任务，JSON 受限于预定义的结构。
- **LLM-Friendly**：高质量代码在 LLM 的训练数据中大量存在。模型对 Python 语法的理解通常优于对特定 JSON Schema 的理解。

**ReAct + TAO 循环**

Code Agent 的运行流程遵循 ReAct 框架，但具体实现在 smolagents 中有一套清晰的步骤分解：

1. `SystemPromptStep`：系统提示被存储
2. `TaskStep`：用户查询被记录
3. 进入 while 循环：
   - `agent.write_memory_to_messages()` 将日志转换为 LLM 可读的聊天消息
   - 消息发送给 Model 生成补全
   - 解析补全，提取代码片段
   - 执行代码
   - 结果记录到 `ActionStep`

每个 step 包含：Thought（LLM 生成推理）→ Action（执行代码）→ Observation（捕获结果）。循环直到 Agent 给出最终答案或达到最大步数。

值得注意的是，smolagents 提供了 `step_callback` 机制，可以在每个 step 结束后注入自定义逻辑（比如日志记录、调试输出）。

### Tool Calling Agent：JSON 方案的选择

Tool Calling Agent 是 smolagents 支持的第二种 Agent 类型，利用 LLM 提供商内置的 tool-calling 能力生成 JSON 结构的调用指令。

```python
# CodeAgent 生成的是 Python 代码
for query in ["catering", "music"]:
    print(web_search(query))

# ToolCallingAgent 生成的是 JSON 结构
[
    {"name": "web_search", "arguments": {"query": "catering"}},
    {"name": "web_search", "arguments": {"query": "music"}}
]
```

什么时候应该用 ToolCallingAgent？课程的判断是：对于不需要变量处理和复杂工具调用的简单系统，ToolCallingAgent 是有效的。但如果任务涉及多步推理、条件分支、数据聚合，Code Agent 的优势就会显现。

我注意到这个设计选择很有意思。其他框架（如 LangChain 的 OpenAI Functions、LlamaIndex 的 function calling）已经形成了"JSON 生成 → 框架解析"的范式，smolagents 选择了一条不同的路。论文的结论是代码更可靠，因为模型在训练数据中见过大量的 Python 代码，但对特定 JSON Schema 的熟悉度有限。

### 工具定义与使用

smolagents 中的工具（Tool）本质上是 LLM 可以调用的函数。要让 LLM 理解一个工具，需要提供四个要素：

1. **name** — 工具名称
2. **description** — 工具功能的文字描述
3. **inputs** — 输入参数的类型和描述
4. **output_type** — 输出类型

**方式一：@tool 装饰器**

推荐用于简单工具。框架通过 Python 自省自动提取函数信息：

```python
@tool
def catering_service_tool(query: str) -> str:
    """
    This tool returns the highest-rated catering service in Gotham City.

    Args:
        query: A search term for finding catering services.
    """
    services = {
        "Gotham Catering Co.": 4.9,
        "Wayne Manor Catering": 4.8,
    }
    best_service = max(services, key=services.get)
    return best_service
```

关键是：类型注解 + 清晰的 docstring + 见名知意的函数名。框架利用 Python 的 introspection 能力，在运行时读取函数的 `__name__`、`__doc__`、`__annotations__` 等属性，自动生成 LLM 能理解的工具描述。

**方式二：Tool 类继承**

适用于需要更多控制的复杂工具：

```python
class SuperheroPartyThemeTool(Tool):
    name = "superhero_party_theme_generator"
    description = "Suggests creative superhero-themed party ideas..."
    inputs = {
        "category": {
            "type": "string",
            "description": "The type of superhero party...",
        }
    }
    output_type = "string"

    def forward(self, category: str):
        themes = {"classic heroes": "Justice League Gala..."}
        return themes.get(category.lower(), "Not found")
```

`inputs` 字典的设计值得注意：它不仅指定了类型，还包含 `description`。这个描述对 LLM 理解参数含义至关重要——尤其是在参数值有枚举范围时，在 docstring 中明确列出允许值可以显著减少 LLM 的幻觉。

**默认工具箱：** smolagents 内置了一些常用工具：
- `PythonInterpreterTool` — 执行 Python 代码
- `FinalAnswerTool` — 输出最终答案
- `UserInputTool` — 获取用户输入
- `DuckDuckGoSearchTool` / `GoogleSearchTool` — 网页搜索
- `VisitWebpageTool` — 访问网页

**社区工具生态：** smolagents 支持从 Hub 加载工具、把 HF Space 当作工具使用、加载 LangChain 工具，甚至通过 MCP 协议集成外部工具。"Space 即工具"这个设计很巧妙——任何部署在 HF Space 上的 gradio 应用都可以直接成为 Agent 的工具。

### Retrieval Agents：智能检索增强

这一节讲的是 Agentic RAG（智能体驱动的检索增强生成），和传统 RAG 的区别在于：

| 维度 | 传统 RAG | Agentic RAG |
|------|---------|-------------|
| 检索次数 | 单次 | 多次，可迭代优化 |
| 查询处理 | 直接使用用户原始查询 | 可重构、分解、扩写查询 |
| 结果筛选 | 按相似度返回 Top-K | 可验证相关性、评估准确度 |
| 多源整合 | 通常单一来源 | 网页 + 本地知识库混合 |

Agentic RAG 解决了传统 RAG 的几个关键限制：
- 过度依赖与用户查询的直接语义相似性，可能忽略相关信息
- 无法根据初步结果调整搜索策略
- 对检索结果缺乏验证机制

课程中的例子使用了 `BM25Retriever`。BM25 是基于关键词匹配的稀疏检索算法，依赖搜索词与文档中具体字词的完全重合度。这和基于 embedding 的密集向量检索是两条不同的技术路线。在实际的 RAG 系统中，这两种方法经常混合使用（hybrid search）——稀疏检索保证关键词精确匹配，密集检索捕获语义相似性。

增强后的检索能力包括：

1. **Query Reformulation（查询重构）**：Agent 不是直接使用用户的原始问题去搜索，而是优化生成更匹配目标文档的搜索词
2. **Query Decomposition（查询分解）**：如果查询包含多个信息维度，分解成多个子查询
3. **Query Expansion（查询扩展）**：用不同的措辞多次搜索同一信息
4. **Reranking（重排序）**：使用 Cross-Encoder 模型对检索结果进行语义相关性重打分
5. **Multi-Step Retrieval（多步检索）**：利用初步结果指导后续搜索
6. **Result Validation（结果验证）**：在将内容纳入响应前分析相关性和准确性

课程提到了一个观点：Agent 应该根据查询类型和上下文在可用工具之间做选择，记忆系统帮助维护对话历史避免重复检索。这其实就是"meta-cognition"——Agent 不仅使用工具，还要决定什么时候用什么工具。

### Multi-Agent Systems：编排与协作

多 Agent 系统解决的问题本质上是"分治"——把复杂任务拆解给专门的子 Agent，每个 Agent 维护自己的上下文，避免单一 Agent 的上下文窗口过载和记忆干扰。

典型结构是 **Orchestrator-Manager 模式**：
- **Manager Agent（管理 Agent）**：负责任务分解、分配、结果汇总
- **Specialized Agents（专业 Agent）**：各自负责一个子任务（如 web search、code execution、image generation）

课程中的 Batmobile 案例很有代表性：Agent 需要找 Batman 的全球拍摄地、计算运输时间、在地图上可视化。如果用单一 Agent，搜索和计算的中间结果会混杂在同一个上下文里，token 开销大且容易互相干扰。分成 Web Search Agent 和 Manager Agent 后：

- Web Agent 专注搜索和抓取数据
- Manager Agent 负责协调和最终的可视化输出

Manager Agent 还有一个 `planning_interval` 参数，控制每隔多少步执行一次明确的计划步骤（planning step）。这在复杂任务中很有用——Agent 定期"停下来思考"，调整后续策略而不是盲目执行。

另一个有意思的设计是 `final_answer_checks`——在 Agent 给出最终答案前，调用一个验证函数检查输出质量。课程案例中，Manager Agent 生成了地图后，通过 GPT-4o 视觉模型检查地图是否正确。这是 Agent 自我验证（self-reflection）的具体实现。

为什么需要多 Agent 而不是一个 Agent 加更多的工具？课程给出了两个直接理由：
1. **聚焦性更好**：每个 Agent 专注于核心任务，表现更佳
2. **输入 token 减少**：分离记忆体后，每个 step 的输入 token 数减少，降低延迟和成本

这本质上是"separation of concerns"原则在 Agent 系统中的应用。

### Vision Agent & Browser Agent

视觉 Agent 将 VLM（Vision-Language Model）集成到 Agent 流程中，扩展了纯文本 Agent 的能力边界。smolagents 支持两种图片传入方式：

**方式一：初始传入。** 图片在 Agent 启动时作为 `task_images` 传入，整个执行过程都可访问。适合已知需要处理哪些图片的场景。

**方式二：动态检索。** 图片在 Agent 运行过程中动态获取，存储在 `ActionStep` 的 `observation_images` 中。在浏览器操作场景中，Agent 截取网页截图，VLM 分析截图内容，决定下一步操作——这就是 Browser Agent 的基本工作方式。

浏览器 Agent 的原理值得深入理解：Agent 借助 Selenium/Helium 控制浏览器，通过截图捕获网页状态，VLM 分析截图内容，Agent 决定下一步操作（点击、滚动、输入等）。这不是简单的 HTML 解析，而是"看"网页——能处理 JavaScript 渲染后的内容、弹窗、动态加载等复杂情况。

课程中的身份验证案例非常生动：Agent 拿到一张来访者的照片，通过 VLM 分析其服装特征，然后联网搜索对比已知角色的资料，最终判断来访者是伪装成 Wonder Woman 的 Joker。这展示了多模态信息在 Agent 流程中的流转路径：图片 → VLM 描述 → 文本检索 → 推理决策。

## 代码实践

### 构建 Code Agent

最基础的用法是创建一个带搜索工具的 Code Agent：

```python
from smolagents import CodeAgent, DuckDuckGoSearchTool, InferenceClientModel

agent = CodeAgent(
    tools=[DuckDuckGoSearchTool()],
    model=InferenceClientModel()
)

agent.run("Search for the best music recommendations for a party at the Wayne's mansion.")
```

运行时会看到执行轨迹，包括生成的 Python 代码：
```python
# Agent 生成的代码
results = web_search(query="best music for a Batman party")
print(results)
```

如果需要允许 Agent 使用额外的 Python 库（默认安全策略只允许内置模块），使用 `additional_authorized_imports`：

```python
agent = CodeAgent(
    tools=[],
    model=InferenceClientModel(),
    additional_authorized_imports=['datetime', 'pandas']
)
```

### 自定义 Tool

使用 `@tool` 装饰器：

```python
from smolagents import CodeAgent, tool, InferenceClientModel

@tool
def suggest_menu(occasion: str) -> str:
    """
    Suggests a menu based on the occasion.
    Args:
        occasion (str): The type of occasion. Allowed values: casual, formal, superhero, custom.
    """
    menus = {
        "casual": "Pizza, snacks, and drinks.",
        "formal": "3-course dinner with wine and dessert.",
        "superhero": "Buffet with high-energy and healthy food.",
    }
    return menus.get(occasion, "Custom menu for the butler.")

agent = CodeAgent(tools=[suggest_menu], model=InferenceClientModel())
agent.run("Prepare a formal menu for the party.")
```

使用 Tool 类继承：

```python
from smolagents import Tool, CodeAgent, InferenceClientModel

class PartyPlanningRetrieverTool(Tool):
    name = "party_planning_retriever"
    description = "Uses semantic search to retrieve relevant party planning ideas."
    inputs = {
        "query": {
            "type": "string",
            "description": "The query to perform related to party planning.",
        }
    }
    output_type = "string"

    def __init__(self, docs, **kwargs):
        super().__init__(**kwargs)
        self.retriever = BM25Retriever.from_documents(docs, k=5)

    def forward(self, query: str) -> str:
        docs = self.retriever.invoke(query)
        return "\n".join([doc.page_content for doc in docs])
```

### 搭建 Multi-Agent 系统

先定义子 Agent，再创建 Manager Agent：

```python
# 子 Agent：专注搜索
web_agent = CodeAgent(
    model=model,
    tools=[GoogleSearchTool(), VisitWebpageTool()],
    name="web_agent",
    description="Browses the web to find information",
    max_steps=10,
    verbosity_level=0
)

# Manager Agent：编排与可视化
manager_agent = CodeAgent(
    model=model,
    tools=[calculate_cargo_travel_time],
    managed_agents=[web_agent],
    additional_authorized_imports=["pandas", "plotly"],
    planning_interval=5,
    max_steps=15,
)
```

关键参数 `managed_agents` 把一个或多个子 Agent 注入 Manager，Manager 可以通过消息将子任务委托给子 Agent。

### 浏览器 Agent 实验

```python
# 需要安装 selenium 和 helium
from smolagents import CodeAgent, tool

@tool
def search_item_ctrl_f(text: str, nth_result: int = 1) -> str:
    """Searches for text on the current page via Ctrl+F."""
    elements = driver.find_elements(By.XPATH, f"//*[contains(text(), '{text}')]")
    elem = elements[nth_result - 1]
    driver.execute_script("arguments[0].scrollIntoView(true);", elem)
    return f"Focused on element {nth_result}"

@tool
def go_back() -> None:
    """Goes back to previous page."""
    driver.back()
```

## Quiz 记录

**测验 1 问题：** smolagents 中的两种主要 Agent 类型是什么？
**答案：** CodeAgent（生成 Python 代码）和 ToolCallingAgent（生成 JSON 结构）。

**测验 1 问题：** Code Agent 相比 JSON 方案的优势有哪些？
**答案：** Composability（可组合性）、Object Management（对象管理）、Generality（通用性）、LLM-Friendly（训练数据中有大量高质量代码）。

**测验 2 问题：** Agentic RAG 相比传统 RAG 的改进？
**答案：** 支持多次检索、查询重构、结果验证、多源整合——Agent 对检索过程有智能控制。

**最终测验问题：** 多 Agent 系统为什么能减少 token 消耗？
**答案：** 分离记忆体后，每个 Agent 只处理自己相关部分的上下文，每个 step 的输入 token 数减少。

## 思考与疑问

**1. 代码优先的选择是不是"把复杂度转移了"？**

表面上看，Code Agent 省去了解析 JSON 的步骤，但引入了代码安全执行的考量。smolagents 的解决方案是沙箱执行加导入白名单。这其实是一个 trade-off：JSON 方案安全但表达能力受限，代码方案灵活但需要严格的安全控制。论文的结论是"代码更好"是基于 LLM 生成质量的，但如果生成代码质量不高（比如小模型），JSON 的约束可能反而是优势。

**2. 多 Agent 的分治会不会引入新的协调问题？**

课程中的 Batmobile 案例很成功，但真实场景中 Task 分解本身就是难题。Manager Agent 需要准确判断何时分解、如何分解、结果怎么合并。如果分解不当，子 Agent 的结果可能难以整合。课程提到 `planning_interval` 参数来缓解这个问题，但能否有效分解任务仍然极度依赖底层模型的能力。

**3. BM25 检索器的实际局限**

课程用了 BM25 作为示例检索器，但在现代化 RAG 系统中，BM25 通常与密集检索混合使用。BM25 是基于词频-逆文档频率的稀疏方法，对同义词、概念级别的匹配无能为力。更实用的方案可能是 hybrid search 加 reranking pipeline。这个选择可能更多是教学目的——用 BM25 保持代码简洁，不引入 embedding 模型的依赖。

**4. smolagents 与 LangGraph / LlamaIndex 在理念上的根本差异**

- LangGraph：Agent 是带有状态的计算图，节点是计算，边是控制流
- LlamaIndex：Agent 是数据管道加 RAG，强调知识检索
- smolagents：Agent 是 Python 代码生成器，"代码是一切"

这三种框架的选择本质上取决于你对 Agent 本质的看法：把它当作状态机、检索器，还是一个编程执行环境？

**5. VLM 在 Agent 中的应用目前还很早期**

课程中的视觉 Agent 案例展示了可能性，但可靠性仍然存疑。VLM 分析截图、Agent 决策、执行操作的链路过长，任一步骤的错误都会累积。浏览器 Agent 尤其如此——页面结构变化、弹窗位置不同都可能导致截图分析失效。

## 关键收获

- smolagents 的核心创新是"代码优先"——Agent 生成可执行代码而非 JSON，以此获得更强的表达能力和灵活性，这个选择有论文支撑且实验证明效果更好。
- 理解 Code Agent 和 ToolCallingAgent 的适用边界：前者适合复杂多步推理，后者适合简单工具调用。不要为了"酷"而盲目选择 Code Agent。
- 工具定义的核心是四个要素（名称、描述、输入、输出），Python introspection 让这个过程几乎无感——写好类型注解和 docstring 就行。
- Agentic RAG 的关键在于 Agent 对检索过程的控制权：可以重构查询、多次检索、验证结果、整合多源信息，这些是传统 RAG 不具备的能力。
- 多 Agent 系统的价值不仅是"分工"，更是"分离上下文"——每个 Agent 维护独立的记忆体，降低 token 消耗，提升模型在子任务上的专注度。
- smolagents 是一千行出头的轻量框架，适合快速原型验证。对于需要复杂状态管理或生产级可靠性的场景，可能需要考虑 LangGraph 等更重的方案——但学完这套框架的思维模型，能帮助理解几乎所有 Agent 框架的共同抽象。
