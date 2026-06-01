---
title: Unit 3 - Agentic RAG 实战
publish: false
status: 🟢 已完成
course_url: https://hf.co/learn/agents-course/unit3/agentic-rag/introduction
publish: true
---

Unit 3 是课程第一个完整的应用案例（Use Case），不再是单纯的概念讲解，而是带着具体的场景——一个豪华晚宴（Gala）——用 Agentic RAG 把前两个单元的知识串起来。Alfred 这个管家 Agent 需要同时处理宾客信息查询、天气判断、网络搜索、Hugging Face 模型统计等多种任务，而 Agentic RAG 正是让这一切成为可能的技术 glue。

这份笔记记录了实现过程中的设计思路和关键代码片段，也梳理了传统 RAG 和 Agentic RAG 的本质差异。

## 课程章节清单

- [x] 介绍：晚宴场景设定与 Alfred 的任务要求
- [x] 什么是 Agentic RAG（检索增强生成）？
- [x] 为宾客故事创建 RAG 工具
- [x] 构建并集成 Agent 工具
- [x] 创建你的 Gala Agent
- [x] 总结

## 核心概念

### Agentic RAG 是什么？

Agentic RAG 不是简单的"在 Agent 里做 RAG"，而是**用 Agent 的推理能力来驱动检索流程**。传统的 RAG 是固定的"检索-生成"流水线，而 Agentic RAG 让模型自己决定何时检索、检索什么、是否需要多轮检索或融合多个来源。

课后课程回顾了 LLM 的固有问题：模型在训练数据上的知识可能过时或不完整。RAG 通过检索外部知识来缓解这个问题。但传统的 RAG 是**一次性的、被动的**——用户提问、检索、生成，流程就结束了。

Agentic RAG 的不同之处在于：Agent 可以在 Thought-Action-Observation 循环中**多次调用检索工具**，甚至组合检索与其他工具。比如 Alfred 被问到某位宾客的情况时，他先检索本地宾客数据库，如果信息不够，再上网搜索补充——这些都是 Agent 自主决策的。

### 传统 RAG vs Agentic RAG

| 维度 | 传统 RAG | Agentic RAG |
|------|----------|-------------|
| 检索次数 | 单次，一次性 | 多轮，可迭代 |
| 查询策略 | 用户原始查询直接检索 | Agent 可改写、分解、优化查询 |
| 信息源 | 单一知识库 | 多源融合（本地 DB + 网络 + API 等） |
| 是否使用工具 | 仅检索 | 可组合搜索、计算、API 调用等多种工具 |
| 结果验证 | 无 | Agent 可评估结果质量，决定是否需要补充检索 |
| 决策方式 | 被动（固定流程） | 主动（Agent 自主判断） |

这其中的核心差异在于**Agent 对检索流程的控制权**。传统 RAG 中检索是独立于生成步骤的预处理；Agentic RAG 中检索是 Agent 工具箱中的一个选项，Agent 可以决定是否使用、用几次、以及用完后下一步做什么。

### Agent 驱动的检索策略

课程中虽然没有直接提出"策略分类"，但从实现中可以提取出几种典型的 Agentic 检索模式：

**查询改写（Query Reformulation）**：Agent 在 Thought 阶段分析用户问题后，可以决定拆解或改写查询。比如用户说"帮我准备和 Tesla 博士的对话"，Agent 可能先查宾客数据，再搜索无线充电的最新进展——这是两个不同的检索动作，查询内容和目标完全不同。

**多步检索（Multi-step Retrieval）**：第一次检索结果不够时，Agent 可以基于 Observation 中的信息提炼新的检索词。这是最直接的"Agentic"体现——它不是一个简单的"查一次完事"，而是不断根据反馈修正。

**多源融合（Multi-source Fusion）**：Alfred 同时拥有宾客数据库（本地 BM25）、网络搜索（DuckDuckGo）、天气 API、Hub 统计四个工具。当被问到"某位 AI 研究员最近有什么新成果"时，他既查本地数据又上网搜，再结合 Hub 下载量给出综合回答。

**结果验证（Result Verification）**：虽然课程中没有显式实现验证步骤，但 Agent 在 Observation 阶段天然具备这一能力——如果检索结果为空或不够具体，Agent 可以重新构造查询再次尝试。

### 宾客信息检索工具（RAG Tool）

这一节的实操性很强。课程使用 Hugging Face 的 `unit3-invitees` 数据集构建了一个宾客信息检索工具，核心是 BM25 检索算法。选择 BM25 而不是 embedding-based 检索的原因很实际：

- BM25 是基于关键词匹配的统计方法，无需 embedding 模型和向量数据库
- 对于名字、关系这类结构化较强的查询，BM25 效果已经足够好
- 降低基础设施门槛，在 HF Space 上运行更轻量

然而值得注意的是，BM25 的局限性也很明显：
- 对语义相似的查询不敏感（比如"计算机先驱"可能匹配不到"Ada Lovelace"）
- 无法处理拼写错误或模糊描述
- 对于较长、语义丰富的描述文本，embedding-based 方法会有明显优势

课程中的 Tip 也提到了可以用 sentence-transformers 升级检索器，这在实际项目中值得考虑。

### 工具与检索的结合

这一部分展示了 Agent 如何**在多个工具之间决策**。Alfred 的工具箱包括：

1. **宾客检索工具**（GuestInfoRetriever）— 本地 BM25，查询宾客数据库
2. **网络搜索工具**（DuckDuckGoSearch）— 获取最新信息
3. **天气工具**（WeatherInfo）— 模拟天气查询
4. **Hub 统计工具**（HubStats）— 查询 Hugging Face 模型下载量

关键设计点在于：Agent 不被告知"什么场景用什么工具"，而是通过工具的 **name 和 description** 让 LLM 自主判断。这意味着工具描述的质量直接影响 Agent 的决策准确性。如果 `guest_info_retriever` 的描述写得不清楚，Agent 可能在不需要检索宾客时调用了它。

这也引出一个更深层的问题：Agent 决策的上限取决于 LLM 对工具描述的理解能力。如果 LLM 对某个工具的用途产生了误解，整个决策链就会出错。所以工具描述要简洁、明确、边界清晰。

### Gala Agent 的端到端设计

最终的 Alfred Agent 把上述所有工具集成到一起，用 `CodeAgent`（smolagents 框架）或 `AgentWorkflow`（LlamaIndex）/ `StateGraph`（LangGraph）驱动。

最让我印象深刻的是 **planning_interval=3** 这个参数——它让 Agent 每 3 步做一次重新规划。这意味着 Agent 不是机械地执行预设路线，而是在执行过程中不断评估进展、调整计划。这与 Unit 1 中讨论的 ReAct 循环一脉相承，只是这里从单步循环变成了带规划间隔的自治行为。

另一个值得关注的设计决策是**记忆（Memory）与 Agent 的解耦**。课程中特别指出：三种框架都没有将记忆直接内置到 Agent 中。smolagents 需要显式 `reset=False` 来保留上下文，LlamaIndex 需要引入 Context 对象，LangGraph 则依赖外部 MemorySaver。这种设计是有意的——它给了开发者更灵活的记忆管理策略，但也意味着默认情况下 Agent 是"无状态"的。

### 与前面单元的衔接

这个案例是 Unit 2 的延续和综合：

- **Unit 2.1 的 smolagents 检索 Agent** 展示了如何用框架内置的检索能力构建 Agent，但场景相对单一。Unit 3 把检索 Agent 扩展成了**多工具 Agent**，并且用真实数据集来驱动。
- **Unit 2.2 的 LlamaIndex RAG 模式** 侧重文档索引和查询管道的搭建。Unit 3 则在 LlamaIndex（和其他框架）之上叠加了 Agent 层，让索引的消费方式从"被动查询"变成了"Agent 驱动的主动检索"。

可以说 Unit 2 教的是"如何搭建 RAG 管线"，Unit 3 教的是"如何让 Agent 自主使用这条管线"——这是从工具到主权的跃迁。

## 代码实践

### 构建 RAG 工具（宾客检索）

```python
import datasets
from langchain_core.documents import Document
from smolagents import Tool
from langchain_community.retrievers import BM25Retriever

# 1. 加载数据集
guest_dataset = datasets.load_dataset("agents-course/unit3-invitees", split="train")

# 2. 转换为 Document 对象
docs = [
    Document(
        page_content="\n".join([
            f"Name: {guest['name']}",
            f"Relation: {guest['relation']}",
            f"Description: {guest['description']}",
            f"Email: {guest['email']}"
        ]),
        metadata={"name": guest["name"]}
    )
    for guest in guest_dataset
]

# 3. 自定义 RAG Tool 类
class GuestInfoRetrieverTool(Tool):
    name = "guest_info_retriever"
    description = ("Retrieves detailed information about gala guests "
                   "based on their name or relation.")
    inputs = {
        "query": {
            "type": "string",
            "description": "The name or relation of the guest."
        }
    }
    output_type = "string"

    def __init__(self, docs):
        self.is_initialized = False
        self.retriever = BM25Retriever.from_documents(docs)

    def forward(self, query: str):
        results = self.retriever.invoke(query)
        if results:
            return "\n\n".join([doc.page_content for doc in results[:3]])
        else:
            return "No matching guest information found."

# 4. 实例化
guest_info_tool = GuestInfoRetrieverTool(docs)
```

这段代码有几个设计细节值得注意：

- `name` 和 `description` 不是装饰性的——它们直接决定了 LLM 是否会在正确的场景下调用这个工具
- `inputs` 中的 type 约束帮助 LLM 生成正确的参数格式
- BM25 不需要 embedding 模型，降低了外部依赖
- `forward` 返回 top-3 结果而非全部，避免超出上下文窗口

### 集成多个工具到 Agent

```python
from smolagents import DuckDuckGoSearchTool
import random

# Web 搜索工具（框架内置）
search_tool = DuckDuckGoSearchTool()

# 自定义天气工具
class WeatherInfoTool(Tool):
    name = "weather_info"
    description = "Fetches dummy weather information for a given location."
    inputs = {
        "location": {
            "type": "string",
            "description": "The location to get weather information for."
        }
    }
    output_type = "string"

    def forward(self, location: str):
        weather_conditions = [
            {"condition": "Rainy", "temp_c": 15},
            {"condition": "Clear", "temp_c": 25},
            {"condition": "Windy", "temp_c": 20}
        ]
        data = random.choice(weather_conditions)
        return f"Weather in {location}: {data['condition']}, {data['temp_c']}°C"

weather_info_tool = WeatherInfoTool()

# Hub 统计工具
from huggingface_hub import list_models

class HubStatsTool(Tool):
    name = "hub_stats"
    description = ("Fetches the most downloaded model from a specific "
                   "author on the Hugging Face Hub.")
    inputs = {
        "author": {
            "type": "string",
            "description": "The username of the model author/organization."
        }
    }
    output_type = "string"

    def forward(self, author: str):
        try:
            models = list(list_models(author=author,
                                      sort="downloads",
                                      direction=-1,
                                      limit=1))
            if models:
                model = models[0]
                return (f"The most downloaded model by {author} is "
                        f"{model.id} with {model.downloads:,} downloads.")
            else:
                return f"No models found for author {author}."
        except Exception as e:
            return f"Error fetching models: {str(e)}"

hub_stats_tool = HubStatsTool()
```

这里课程用了一个模拟天气 API，这在实际项目中可能是临时 placeholder，但包含一个值得展开的思考：**工具的真实性/保真度问题**。如果 WeatherInfoTool 返回的是随机数据，它的存在价值是什么？在开发阶段，模拟工具可以验证 Agent 的工具调用逻辑是否正确；但在生产环境中，模拟数据反而会误导 Agent 的决策。所以工具从 mock 到 production 的切换策略是一个需要提前规划的架构决策。

### 完整的 Gala Agent

```python
from smolagents import CodeAgent, InferenceClientModel

model = InferenceClientModel()

alfred = CodeAgent(
    tools=[guest_info_tool, weather_info_tool, hub_stats_tool, search_tool],
    model=model,
    add_base_tools=True,
    planning_interval=3  # 每 3 步重新规划一次
)

# 查询示例
response = alfred.run("Tell me about 'Lady Ada Lovelace'")
print(response)

# 组合查询
response = alfred.run(
    "What's the weather like in Paris tonight? "
    "Will it be suitable for our fireworks display?"
)
print(response)

# 带上下文的对话
response1 = alfred.run("Tell me about Lady Ada Lovelace.")
response2 = alfred.run(
    "What projects is she currently working on?",
    reset=False  # 保留上下文
)
```

`planning_interval=3` 是一个启动参数。它的含义是：Agent 每执行 3 个步骤（Thought-Action-Observation 循环）后，暂停并重新评估当前进展，决定下一步计划是否需要调整。这对于长时间运行的多步任务来说是一个重要的防"跑偏"机制。

## 思考与疑问

- **Agentic RAG 和 Multi-hop RAG 有什么区别？** 从概念上看，两者的确有重叠——都是多步检索。但 Agentic RAG 强调 Agent 的自主决策权，不仅限于检索策略的调整，还包括是否调用非检索类工具（天气、搜索等），而 Multi-hop RAG 通常只关注检索路径的优化。Agentic RAG 是"检索 + 行动"的综合体，Multi-hop RAG 是"检索 + 检索"的链式深化。

- **工具的 name/description 质量 = Agent 能力上限的瓶颈？** 课程中使用工具描述来引导 Agent 决策，这让我想到 Unit 1 中讨论的 prompt engineering——这里同样存在"Garbage In, Garbage Out"的问题。不同模型对工具描述的理解能力差异很大，强模型（GPT-4、Claude）可以从模糊描述中推断出正确用途，而弱模型可能需要极其精确的措辞。

- **BM25 的选择是务实的，但值得探讨 trade-off。** 课程选择 BM25 而非 embedding-based 检索，降低了入门门槛。但在实际项目中，如果数据集规模增长或查询变得更加语义化（比如"介绍一下那位对计算机有开创性贡献的数学家"而不是直接搜"Ada Lovelace"），BM25 的效果会显著下降。一个现实的做法是：小规模、结构化数据用 BM25；大规模、非结构化文本用 embedding。

- **Agent 的无状态设计是有意为之还是妥协？** 课程指出三种框架都未默认启用记忆。我认为这既是有意设计（让开发者灵活控制记忆策略和成本），也是现实妥协（记忆管理会大幅增加系统复杂度，包括上下文窗口管理、记忆压缩、遗忘策略等）。但"默认无状态"意味着在多轮交互场景中需要额外开发工作。

- **这个案例能直接映射到实际业务场景。** Alfred + Gala 虽然是虚构的，但"一个 Agent 同时管理多个知识源 + 实时工具"的架构非常真实。比如客服场景：Agent 需要查客户信息（内部 CRM）、查实时订单状态（API）、查最新政策（知识库）——这和 Alfred 的宾客库 + 天气 + 搜索的结构几乎一致。

## 关键收获

- Agentic RAG 的核心不是检索技术本身，而是**Agent 对检索流程的自主控制权**——它让 RAG 从"被动的信息补充"变成了"主动的信息探索"
- 工具描述的清晰度直接影响 Agent 的决策质量，写工具时花时间打磨 name 和 description 不是"非功能性需求"，而是**核心功能**
- BM25 在结构化数据检索场景中仍然是一种务实有效的选择，但需要意识到它的语义局限性
- `planning_interval` 这样的机制展示了 Agent 如何通过**周期性重新规划**来避免在错误路径上越走越远，这是长时间运行 Agent 的关键设计模式
- 多工具 Agent 的核心挑战不是"怎么实现每个工具"，而是**工具间的调度逻辑**——什么时候用哪个、结果如何融合、冲突如何解决
- Agentic RAG 的架构模式（本地知识库 + 实时搜索 + 专用 API）可以直接映射到真实业务场景，课程中的 Gala 案例虽小但极具代表性
