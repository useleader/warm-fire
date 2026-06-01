---
title: Bonus 2 - Agent 可观测性与评估
publish: false
status: 🟢 已完成
course_url: https://hf.co/learn/agents-course/bonus-unit2/introduction
publish: true
---

最近把 Hugging Face Agents 课程的 Bonus Unit 2 学完了，这一章讲的是 Agent 的可观测性（Observability）和评估（Evaluation）。如果说前面 Unit 1 和 Unit 2 教我"怎么造 Agent"，那这一章就是教我"怎么让 Agent 在真实世界里可靠地跑起来"。

坦白说，这部分内容比我想象中更实用。一开始我觉得"可观测性"不过是加几个日志的事，但学完之后发现，Agent 的非确定性（non-deterministic）和多步推理特性，让传统的调试和监控手段完全不够用——你面对的不是一个请求-响应，而是一个可能包含多次 LLM 调用、工具调用、内部推理的复杂闭环。没有可观测性，Agent 就是一个黑盒。

笔记仍然按我的习惯来：先列课程章节，再深入核心概念，然后是代码实践、测验、思考与疑问，最后是关键收获。

## 课程章节清单

- [x] 介绍
- [x] 什么是 Agent 可观测性与评估？
- [x] 监控与评估 Agent（Notebook 实践）
- [x] 测验

## 核心概念

### 为什么 Agent 可观测性是个"真问题"

课程开篇就点明了这个核心矛盾：**Agent 很美，但很难 debug**。

一个典型的 Agent 工作流是这样的：

1. 用户输入 query
2. LLM 生成 Thought（内部推理）
3. Agent 解析 Thought，调用工具（Tool）
4. 工具执行返回 Observation
5. Observation 拼回对话上下文，进入下一轮循环
6. 直到 LLM 判断任务完成，输出最终答案

这里面每一步都可能出问题：LLM 生成了格式错误的 Action、工具调用超时、中间步骤陷入死循环、token 消耗爆炸——而传统监控只能告诉你"这次请求花了多少钱"，完全看不到内部发生了什么。

**核心问题**：LLM 的输出是非确定性的，同样的 prompt 每次结果可能不同。这就意味着不能靠"测一次过了就完事"，必须持续观察和评估。

我发现这个问题的本质和分布式系统的可观测性困境非常像——请求链路过长、中间环节多、出问题的位置难以定位。Agent 的可观测性本质上就是"把黑盒变成玻璃盒"。

### Observability vs Monitoring：有什么区别？

课程区分了这两个概念，我觉得说得挺清楚：

| 概念 | 定义 | 类比 |
|------|------|------|
| **Monitoring（监控）** | 观察系统的外部状态，判断"系统还活着吗？" | 看仪表盘上的绿灯 |
| **Observability（可观测性）** | 通过外部信号理解系统内部状态，回答"为什么出问题了？" | 拆开引擎看内部 |

换句话说，Monitoring 告诉你有问题，Observability 告诉你问题在哪。

实际关系是：**Observability 是 Monitoring 的升级版**。Monitoring 只需要预设的指标和告警阈值，而 Observability 要求系统主动暴露内部状态，让你能在遇到未知问题时进行探查式分析。

### 三大支柱：Traces、Spans 和 Metrics

课程着重讲了 OpenTelemetry 标准下的三个核心概念：

**Traces（追踪）**

一个 Trace 代表一次完整的 Agent 任务执行的端到端记录。从用户输入 query 到 Agent 输出最终结果，整个过程是一个 Trace。如果一个 Task 需要多次 Thought-Action-Observation 循环，那所有循环都在同一个 Trace 下。

**Spans（跨度）**

Span 是 Trace 里的单个步骤。一个 Agent Trace 可能包含多个 Span：

- 第 1 轮 Thought（LLM call）
- 第 1 轮 Action（Tool call）
- 第 1 轮 Observation（工具返回结果）
- 第 2 轮 Thought（LLM call 分析 Observation）
- ...

这种层级结构让定位问题变得非常直观——一眼就能看到哪一步耗时最长、哪一步失败了、哪一步 token 消耗最大。

**Metrics（指标）**

Metrics 是可量化的测量值，聚合多个 Trace 得出统计数据。课程提到六类核心指标，我用表格整理了一下：

| 指标类别 | 衡量什么 | 为什么重要 |
|---------|---------|-----------|
| **Latency（延迟）** | 每个步骤和整个任务的耗时 | 用户的直接体验指标，也帮助定位瓶颈 |
| **Costs（成本）** | token 消耗 × 模型单价 | 没有监控，月底账单来了才知道花了多少 |
| **Request Errors（请求错误）** | API 失败、工具调用失败的次数 | 决定是否需要加 fallback 或重试 |
| **User Feedback（显式用户反馈）** | 用户的点赞/踩、评分 | 最直接的"这 Agent 行不行"信号 |
| **Accuracy（准确率）** | 任务正确完成的比率 | 核心质量指标，但需要定义"什么算完成" |
| **Automated Evaluation（自动评估）** | LLM-as-Judge 打分等 | 在大规模场景下手动评审不可行 |

### 离线评估（Offline Evaluation）vs 在线评估（Online Evaluation）

这是课程中反复强调的核心框架。评估 Agent 不能只靠一种方式，必须两种结合。

**离线评估（Offline Evaluation）**

离线评估就是用已有的基准数据集来测试 Agent。流程很清晰：

1. 准备一个有已知正确答案的数据集（比如 GSM8K 数学题）
2. 让 Agent 在数据集上跑一遍
3. 对比 Agent 的输出和标准答案
4. 得出准确率等指标

优点是可重复、有 ground truth、可以集成到 CI/CD 管道中。缺点是测试集的覆盖率有限——在一个固定集上表现良好的 Agent，在线上可能会遇到完全没见过的场景。

课程的一个建议我记下来了：维护一个小规模的"冒烟测试集"做快速检查，加上一个大规模评估集做全面测试。

**在线评估（Online Evaluation）**

在线评估是在真实生产环境中持续监控 Agent 的表现。这不只是看成功率，还包括：

- 实时跟踪成本和延迟
- 收集用户显式反馈（👍/👎）
- 收集隐式反馈（用户重复提问、点击重试按钮——这些都是"Agent 没满足需求"的信号）
- LLM-as-Judge 自动评分
- A/B 测试（新版本与旧版本并行运行对比）

**两者的结合循环**

课程画了一个关键的迭代闭环，我觉得这是整个单元最有价值的部分之一：

```
离线评估 → 部署新版本 → 监控在线指标并收集失败案例 → 将失败案例加入离线测试集 → 迭代
```

这个循环意味着：**评估不是一次性的，而是持续的、自我增强的**。每次线上失败都是"免费的训练数据"，反馈给离线测试集让未来的评估更全面。

### OpenTelemetry 的作用

课程提到很多 Agent 框架（包括 smolagents）使用 OpenTelemetry 标准来暴露遥测数据。OpenTelemetry 提供了一个标准化的 instrumentation 层，不管后端用的是什么工具（Langfuse、Arize 还是自建），遥测数据格式是一致的。

`SmolagentsInstrumentor` 是 smolagents 框架中的关键类——一行代码就能自动捕获 Agent 的所有执行细节。

### 与 Unit 1 的连接

回过头看，Unit 1 讲的 Thought-Action-Observation 循环在 Unit 1 时还是一个理论模型，而在这一章里，**可观测性把 T-A-O 循环变成了一个可视化的事实**：

- 每个 Thought 对应一个 LLM call 的 Span
- 每个 Action 对应一个工具调用的 Span
- 每个 Observation 对应工具返回数据的 Span
- 多个循环嵌套在一起形成完整的 Trace

换句话说，可观测性就是把 Agent 的"大脑活动"录下来，让你能回放和诊断。

## 代码实践

### 环境配置

首先安装必要的库：

```bash
pip install langfuse 'smolagents[telemetry]' openinference-instrumentation-smolagents datasets 'smolagents[gradio]' gradio --upgrade
```

### Step 1：配置 Langfuse 并 Instrument Agent

```python
import os

# 从 Langfuse 项目设置页面获取密钥
os.environ["LANGFUSE_PUBLIC_KEY"] = "pk-lf-..."
os.environ["LANGFUSE_SECRET_KEY"] = "sk-lf-..."
os.environ["LANGFUSE_HOST"] = "https://cloud.langfuse.com"  # EU 区域

# 初始化 Langfuse 客户端
from langfuse import get_client
langfuse = get_client()

# 验证连接
if langfuse.auth_check():
    print("Langfuse 客户端已成功认证！")
else:
    print("认证失败，请检查凭据。")

# 使用 SmolagentsInstrumentor 自动插桩
from openinference.instrumentation.smolagents import SmolagentsInstrumentor

SmolagentsInstrumentor().instrument()
```

### Step 2：测试插桩是否生效

```python
from smolagents import InferenceClientModel, CodeAgent

agent = CodeAgent(
    tools=[],
    model=InferenceClientModel()
)

agent.run("1+1=")
```

执行后去 Langfuse Traces Dashboard 查看，应该能看到完整的 Trace 记录。这一步虽然简单，但关键是验证 instrumentation 链路是否畅通——如果这一步看不到 Trace，后面做的都是白工。

### Step 3：复杂 Agent 与 Trace 结构

```python
from smolagents import CodeAgent, DuckDuckGoSearchTool, InferenceClientModel

search_tool = DuckDuckGoSearchTool()
agent = CodeAgent(tools=[search_tool], model=InferenceClientModel())

agent.run("How many Rubik's Cubes could you fit inside the Notre Dame Cathedral?")
```

这个例子会展示一个更丰富的 Trace 树，包含多次工具调用和 LLM 调用的嵌套 Span。

### Step 4：添加自定义属性丰富 Trace

```python
from smolagents import CodeAgent, DuckDuckGoSearchTool, InferenceClientModel

search_tool = DuckDuckGoSearchTool()
agent = CodeAgent(tools=[search_tool], model=InferenceClientModel())

with langfuse.start_as_current_span(name="Smolagent-Trace") as span:
    response = agent.run("What is the capital of Germany?")
    
    # 附加业务属性——这些对后续分析非常有用
    span.update_trace(
        input="What is the capital of Germany?",
        output=response,
        user_id="smolagent-user-123",
        session_id="smolagent-session-123456789",
        tags=["city-question", "testing-agents"],
        metadata={"email": "user@langfuse.com"},
    )

langfuse.flush()
```

这里 `user_id`、`session_id`、`tags` 这些字段的价值在于：当线上出现问题时，可以按用户、按会话、按标签快速筛选和定位。

### Step 5：通过 Gradio 收集用户反馈

```python
import gradio as gr
from smolagents import CodeAgent, InferenceClientModel
from langfuse import get_client

langfuse = get_client()
model = InferenceClientModel()
agent = CodeAgent(tools=[], model=model, add_base_tools=True)

trace_id = None

def respond(prompt, history):
    with langfuse.start_as_current_span(name="Smolagent-Trace"):
        output = agent.run(prompt)
        global trace_id
        trace_id = langfuse.get_current_trace_id()
        history.append({"role": "assistant", "content": str(output)})
        return history

def handle_like(data: gr.LikeData):
    if data.liked:
        langfuse.create_score(value=1, name="user-feedback", trace_id=trace_id)
    else:
        langfuse.create_score(value=0, name="user-feedback", trace_id=trace_id)

with gr.Blocks() as demo:
    chatbot = gr.Chatbot(label="Chat", type="messages")
    prompt_box = gr.Textbox(placeholder="Type your message...", label="Your message")
    prompt_box.submit(fn=respond, inputs=[prompt_box, chatbot], outputs=chatbot)
    chatbot.like(handle_like, None, None)

demo.launch()
```

这个例子展示了如何把可观测性嵌入到用户界面中。关键的思路：**用户反馈不仅仅是 UX 数据，它直接关联到具体的 Trace**，这样当用户点"踩"的时候，你可以直接点开那个 Trace 看当时 Agent 到底做了什么。

### Step 6：离线评估——使用 GSM8K 数据集

```python
import pandas as pd
from datasets import load_dataset
from langfuse import get_client

langfuse = get_client()

# 加载 GSM8K 数据集
dataset = load_dataset("openai/gsm8k", 'main', split='train')
df = pd.DataFrame(dataset)

# 在 Langfuse 中创建数据集实体
langfuse_dataset_name = "gsm8k_dataset_huggingface"
langfuse.create_dataset(
    name=langfuse_dataset_name,
    description="GSM8K benchmark dataset uploaded from Huggingface",
    metadata={"date": "2025-03-10", "type": "benchmark"}
)

# 向数据集添加前 10 个条目
for idx, row in df.iterrows():
    langfuse.create_dataset_item(
        dataset_name=langfuse_dataset_name,
        input={"text": row["question"]},
        expected_output={"text": row["answer"]},
        metadata={"source_index": idx}
    )
    if idx >= 9:
        break

# 在数据集上运行 Agent
from smolagents import CodeAgent, InferenceClientModel

model = InferenceClientModel()
agent = CodeAgent(tools=[], model=model, add_base_tools=True)

def run_smolagent(question):
    with langfuse.start_as_current_generation(name="qna-llm-call") as generation:
        result = agent.run(question)
        generation.update_trace(input=question, output=result)
        return result

dataset = langfuse.get_dataset(name=langfuse_dataset_name)

for item in dataset.items:
    with item.run(
        run_name="smolagent-notebook-run-01",
        run_metadata={"model_provider": "Hugging Face", "temperature_setting": 0.7},
        run_description="Evaluation run for GSM8K dataset"
    ) as root_span:
        generated_answer = run_smolagent(question=item.input["text"])
```

这段代码值得注意的点：`dataset.items` 和 `item.run()` 是 Langfuse 提供的数据集评估 API。它的设计意图是——数据集中的每个 item 的执行都是一个独立的 Trace，并且这些 Trace 都被关联到同一个数据集运行（run）上，方便后续按运行批次对比性能。

## Quiz 记录

以下是 Bonus Unit 2 的测验题和我的答案记录：

**Q1：AI Agent 的可观测性主要指的是什么？**

> 通过日志、指标和追踪（traces）来理解 Agent 内部发生了什么，从而调试和提升 Agent 性能。

可观测性的核心是"从外部信号理解内部状态"，而不是简单的日志记录。

**Q2：以下哪项不是 Agent 可观测性中常见的监控指标？**

> Agent 的代码行数。

其他选项（延迟、每次 Agent 运行的成本、用户反馈与评分）都是标准指标，但代码行数跟运行时行为无关。

**Q3：以下哪个选项最好地描述了 AI Agent 的离线评估？**

> 使用有已知正确答案的 curated 数据集来评估 Agent 的表现。

离线评估的关键特征：受控环境、有 ground truth、可重复。

**Q4：在线评估 AI Agent 的优势是什么？**

> 在真实生产环境中捕捉实时的用户交互和性能数据。

在线评估的价值在于"测到你没测到的"。离线测试集再大，也覆盖不了所有线上场景。

**Q5：OpenTelemetry 在 Agent 可观测性和评估中扮演什么角色？**

> 提供一个标准化的框架来 instrument 代码，收集 traces、metrics 和 logs。

OpenTelemetry 本身不存储数据，它是数据采集层的标准接口。选什么后端（Langfuse、Arize、Jaeger 等）是另一层的事。

## 思考与疑问

**1. Instrumentation 的"免费午餐"能持续多久？**

`SmolagentsInstrumentor().instrument()` 一行代码就搞定了所有插桩，这当然很好。但我的疑问是：这种自动化插桩能捕获的粒度有限——如果我想记录业务层面的指标（比如"用户意图分类是否正确"、"Agent 是否遵守了系统 prompt 中的约束"），仍然需要手动加自定义 Span。自动化 instrumentation 解决的是"有没有"的问题，但"好不好"还得靠人工设计。

**2. 离线评估的数据集维护成本被低估了**

课程提到要把线上失败案例不断补充到离线测试集中，这个循环在理论上完美，但在实践中意味着需要有人持续标注数据。我猜测在规模较大的团队中，这需要专门的人或工具来维护，可能还需要引入标注平台。对于个人项目或小团队，这个成本可能比想象中高。

**3. LLM-as-Judge 的可靠性问题**

课程介绍了用 LLM 去评估 LLM 的做法，但在 Unit 1 里我们已经知道 LLM 本身就有幻觉和偏见的问题。用另一个 LLM 来做 judge，它的判断真的可靠吗？课程没有深入讨论 judge LLM 的准确率如何验证，或者 judge LLM 本身是否需要评估。这让我觉得 LLM-as-Judge 更适合做"快速筛选"而不是"最终裁决"。

**4. Agent 可观测性的"Trace 爆炸"问题**

当 Agent 经历 5-10 轮 Thought-Action-Observation 循环时，一个任务可能产生几十个 Spans。如果用户量一大（比如几百个并发请求），Trace 数据的量级会非常可观。课程的动手示例跑的是单个 Agent 请求，没有讨论生产环境下的数据采样策略——是否需要采样？什么情况下要全量保存？这些在实际部署中是无法回避的问题。

**5. 从"能跑"到"可靠"的跨越**

这个 Bonus Unit 给我的最大感受是：**造一个 demo Agent 很容易，让一个 Agent 可靠地运行在生产环境中是另一回事**。可观测性和评估不是"锦上添花"的功能，而是把 Agent 从玩具变成工具的必要条件。没有可观测性的 Agent 就像没有仪表盘的飞机——也许能飞，但你不知道什么时候会掉下来。

## 关键收获

1. **Agent 可观测性不是可选项，而是生产部署的硬性要求。** Agent 的非确定性和多步执行特性决定了传统的日志和监控手段不够用，必须通过 Traces、Spans 和 Metrics 实现端到端的透明性。

2. **在线评估和离线评估必须结合，缺一不可。** 离线评估提供可控的、可重复的基准测试；在线评估捕捉真实世界的边缘情况。两者通过"失败案例回流到测试集"的循环实现持续改进。

3. **OpenTelemetry 是 Agent 可观测性的基石。** 它提供了标准化的 instrumentation 接口，使得 smolagents、LangGraph、LlamaIndex 等不同框架可以接入同一个可观测性后端，避免厂商锁定。

4. **用户反馈是最高质量的评估信号。** 无论是显式的 👍/👎 还是隐式的重复提问，真实的用户行为比任何自动化指标都更能反映 Agent 的实际表现。

5. **造 Agent 和运维 Agent 是完全不同的两件事。** 能写一个能跑的 Agent 只需要理解 LLM + Tools 的基础知识，但要把 Agent 部署到生产环境中稳定运行，需要一整套可观测性、评估、迭代的基础设施和流程。
