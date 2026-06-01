---
title: Bonus 1 - Function-Calling 微调
publish: false
status: 🟢 已完成
course_url: https://hf.co/learn/agents-course/bonus-unit1/introduction
publish: true
---

这是 Hugging Face Agents 课程的第一个 Bonus Unit，主题是微调 LLM 以实现 Function Calling。如果说 Unit 1 教的是"怎么用现成的 Agent 框架"，那这个 Bonus Unit 教的就是"怎么让模型本身学会调用工具"。思路完全不同——前者靠 Prompt，后者靠训练。

本笔记包含我对 Function Calling 原理、微调流程、LoRA 机制的理解，以及一些个人思考。

## 课程章节清单

- [x] 介绍
- [x] 什么是 Function Calling？
- [x] 微调模型实现 Function Calling
- [x] 总结

## 核心概念

### What is Function Calling

Function Calling 是 LLM 主动与环境交互的一种能力，最早由 GPT-4 引入（2023 年中），后来被大量开源模型复现。它不是通过 Prompt 告诉模型"你有这些工具可以用"，而是在训练阶段就让模型**学会**何时以及如何调用工具。

**这跟 Unit 1 的 Agent 有什么区别？**

这是理解这个 Bonus Unit 最关键的问题。对比一下两种思路：

| 维度 | ReAct Agent（Unit 1 方式） | Function Calling（微调方式） |
|------|---------------------------|---------------------------|
| 工具使用机制 | Prompt 中列出工具描述，模型靠泛化能力理解 | 模型在训练中见过大量 API 调用样例，内化了工具调用格式 |
| 输出格式 | 靠 System Prompt + Stop-and-Parse 保证格式 | 模型原生生成结构化函数调用消息 |
| 推理过程 | 依赖 Thought-Action-Observation 文本循环 | 可使用特殊 token 隔离推理与调用，也可省略显式推理 |
| 准确率 | 对复杂工具集可能输出不合规的 JSON | 通常更高，参数格式一致性更好 |
| Token 消耗 | 每轮都需要输出 Thought 文本，token 开销大 | 省略推理文本，token 更少 |
| 适用范围 | 几乎所有模型都能用，无需额外训练 | 需要经过微调的模型 |

简单来说：ReAct 是**通过提示工程让模型表现出工具使用能力**，Function Calling 是**通过训练让模型内化工具使用能力**。前者灵活但脆弱，后者稳定但需要训练成本。

**一个值得注意的历史细节：** GPT-4 的 Function Calling 并不是通过传统意义上的微调实现的，而是通过 RLHF 阶段的偏好对齐让模型学会了函数调用格式。后来开源社区（尤其是 Nous Research、Hermes 系列）才真正用 SFT 把这件事复现出来。所以"微调"只是实现 Function Calling 的路径之一，不是唯一路径。

### Function Calling 的运作机制

**对话结构的演化**

在传统对话中，消息只有 user 和 assistant 两个角色交替出现。Function Calling 引入了两个新角色：

1. **Action（动作）** —— 模型决定调用某个工具时生成的函数调用消息
2. **Observation（观察）** —— 工具执行后返回的结果消息

以 Mistral API 为例，一个典型的 Function Calling 对话长这样：

```python
conversation = [
    {
        "role": "user",
        "content": "What's the status of my transaction T1001?"
    },
    {
        "role": "assistant",
        "content": "",
        "function_call": {
            "name": "retrieve_payment_status",
            "arguments": "{\"transaction_id\": \"T1001\"}"
        }
    },
    {
        "role": "tool",
        "name": "retrieve_payment_status",
        "content": "{\"status\": \"Paid\"}"
    },
    {
        "role": "assistant",
        "content": "Your transaction T1001 has been successfully paid."
    }
]
```

注意上例中 function_call 是 assistant 消息里的一个字段，而不是独立角色。但在实际微调过程中，我们需要在 token 级别区分这些角色，这就引出了 **Special Tokens（特殊标记）**。

**Special Tokens 的设计**

为了让模型区分"推理"、"调用"和"观察结果"，需要引入一组新的特殊 token：

- `[AVAILABLE_TOOLS]` / `[/AVAILABLE_TOOLS]` —— 标记可用工具列表的起止
- `[TOOL_CALLS]` —— 表示模型正在发起工具调用
- `[TOOL_RESULTS]` / `[/TOOL_RESULTS]` —— 标记工具执行结果的起止

课程和 notebook 中使用的是另一套标记（ChatML 风格的变体）：

- `<tools>` / `</tools>` —— 工具定义包围
- `<think>` / `</think>` —— 模型内部推理过程包围
- `<tool_call>` / `</tool_call>` —— 函数调用内容包围
- `<tool_response>` / `</tool_response>` —— 工具响应包围
- `<pad>` —— 填充 token
- `<eos>` —— 序列结束 token

这套标记的设计是有讲究的：`<think>` 和 `<tool_call>` 是平级的，模型可以在一次生成中交替输出多个 `think -> tool_call -> tool_response` 循环——这和 ReAct 循环在逻辑上完全对应，只是 token 化了。

### 微调流程与数据准备

**从预训练到 Function Calling 的三阶段**

一个完整的模型训练流程包含三步：

1. **预训练（Pre-training）** —— 在海量文本上做自监督学习，输出基础模型。如 `google/gemma-2-2b`，只知道预测下一个 token，没有指令遵循能力。
2. **指令微调（Instruction Fine-tuning）** —— 用对话数据微调，让模型学会遵循指令。如 `google/gemma-2-2b-it`。
3. **偏好对齐（Preference Alignment）** —— 用 RLHF/DPO 等方法对齐人类偏好。

本教程直接在 `google/gemma-2-2b-it`（第二步的模型）上做 Function Calling 微调，而不是从基础模型开始。理由很直观：基础模型连"对话格式"都没学会，从头训需要同时学习指令遵循、对话格式和函数调用三件事，难度陡增。在已指令微调的模型上用 Adapter 做微调，需要学的内容量最小化。

**数据集**

使用的数据集是 [Jofthomas/hermes-function-calling-thinking-V1](https://huggingface.co/datasets/Jofthomas/hermes-function-calling-thinking-V1)，它是 Nous Research Hermes 系列数据集的一个变体。数据格式是标准的 messages 格式（类似 OpenAI 的 message 结构），包含 system / user / assistant / tool 多种角色。

**数据预处理的关键步骤**

数据集中的 system message 被合并到了第一条 user message 中，而不是保留为独立角色。这是因为 Gemma 2 的 chat template 不支持 system 角色。预处理函数把 system 内容和一段固定的 thinking prompt 拼接到 user 消息前面：

> "Also, before making a call to a function take the time to plan the function to take. Make that thinking process between <think>{your thoughts}</think>"

这个设计很重要：它告诉模型在调用函数之前先"think"，而且 think 的内容要放在 `<think>` 标签内。这意味着模型在生成时先输出推理过程，再输出具体的函数调用——本质上就是 ReAct 的 Thought 和 Action，只是被训练成了模型的内部能力。

### LoRA（Low-Rank Adaptation）

**LoRA 是什么？为什么适合这个场景？**

LoRA 是一种轻量级微调技术。它的核心思想是：不修改原始模型权重，而是在 Transformer 的线性层旁边插入一对低秩分解矩阵作为适配器（Adapter）。训练时只更新适配器的权重，原始权重保持冻结。

这样做的好处：

- **参数量大减** —— 对 Gemma-2-2B（20 亿参数），LoRA 只需要训练几千万参数
- **显存需求低** —— 可以在消费级 GPU（如 RTX 4090 24GB）上运行
- **产出文件小** —— 训练出的 adapter 权重只有几百 MB，易于分发
- **推理时零延迟** —— Adapter 权重可以 merge 回原始模型，不增加推理开销
- **方便切换任务** —— 换任务只需换 adapter，无需保留多个完整模型副本

**配置参数**

Notebook 中的 LoRA 配置：

```python
from peft import LoraConfig, TaskType

peft_config = LoraConfig(
    r=16,              # 秩维度，越小压缩越大
    lora_alpha=64,     # 缩放因子，越大适配越强
    lora_dropout=0.05, # Dropout 防过拟合
    target_modules=[   # 要注入的模块
        "gate_proj", "q_proj", "lm_head", "o_proj",
        "k_proj", "embed_tokens", "down_proj", "up_proj", "v_proj"
    ],
    task_type=TaskType.CAUSAL_LM,
)
```

这里 target_modules 覆盖了注意力层（q/k/v/o_proj）和前馈层（gate/down/up_proj），甚至包括了 lm_head 和 embed_tokens——这值得注意。通常 LoRA 建议只 target 注意力层的 q 和 v，但这里覆盖了更多模块，原因可能是 function calling 需要改变模型的输出分布模式（包括 token embedding 空间），而不仅仅是调整注意力权重。

### 训练配置

```python
from trl import SFTConfig, SFTTrainer

training_arguments = SFTConfig(
    output_dir="gemma-2-2B-it-thinking-function_calling-V0",
    per_device_train_batch_size=1,
    per_device_eval_batch_size=1,
    gradient_accumulation_steps=4,
    learning_rate=1e-4,
    max_grad_norm=1.0,
    num_train_epochs=1,
    warmup_ratio=0.1,
    lr_scheduler_type="cosine",
    max_seq_length=1500,
    eval_strategy="epoch",
    bf16=True,
    gradient_checkpointing=True,
    packing=True,
    report_to="tensorboard",
)
```

几个关键配置的考量：

- **batch_size=1 + gradient_accumulation_steps=4** —— 有效 batch size 为 4，适合显存受限的场景
- **max_seq_length=1500** —— 函数调用的输入输出通常不会太长，1500 token 足够涵盖一次完整的多轮调用
- **学习率 1e-4** —— LoRA 微调的标准学习率范围（全参数微调通常只有 1e-5 到 5e-5，LoRA 可以更高因为只更新少量参数）
- **packing=True** —— 将多个短样本拼接到一个序列中，提高训练效率

训练完成后输出两样东西：LoRA adapter 权重（通过 `trainer.save_model()`）和 tokenizer 配置（包含新增的 special tokens）。

### 模型评估

Notebook 中的"评估"实际上是在测试集上做推理，观察生成结果。这更像是一个 sanity check，而不是严格意义上的评估。做法是：

1. 加载训练好的 adapter
2. 从测试集中取一条 prompt
3. 用 `model.generate()` 生成补全
4. 观察输出是否包含正确的 `<think>` 和 `<tool_call>` 格式

```python
model = PeftModel.from_pretrained(model, peft_model_id)
model.eval()

prompt = """<bos><start_of_turn>human
You are a function calling AI model...<end_of_turn><eos>
<start_of_turn>model
<think>"""

inputs = tokenizer(prompt, return_tensors="pt", add_special_tokens=False)
outputs = model.generate(**inputs, max_new_tokens=300,
                         do_sample=True, top_p=0.95,
                         temperature=0.01)
print(tokenizer.decode(outputs[0]))
```

课程没有提供量化的评估指标（如 function call accuracy、参数正确率等），这算是一个遗憾。生产环境中通常需要更系统化的评估：

- **格式合规率** —— 模型输出的函数调用是否能被正确解析
- **参数匹配率** —— 函数名和参数是否与预期一致
- **幻觉调用率** —— 模型是否调用了不存在的函数或提供了不存在的参数
- **多轮调用成功率** —— 多步工具调用的整体完成率

### 三种 Agent 范式的对比

结合 Unit 1 和本 Bonus Unit，我们可以对三种 Agent 范式做一个系统性对比：

| 维度 | ReAct + Prompt | Code Agent | Function-Calling Fine-tuned |
|------|---------------|-----------|---------------------------|
| 实现方式 | System prompt 加工具列表 | 生成 Python 代码执行 | 模型原生输出结构化函数调用 |
| 依赖 | 依赖模型的指令遵循能力 | 依赖模型的代码生成能力 | 依赖微调数据质量和训练 |
| 输出结构 | JSON 或文本，靠 Stop-and-Parse | Python 语法天然结构化 | 预定义的 special token 结构 |
| 表达力 | 有限，受限于 JSON 格式 | 最强，代码可包含逻辑和循环 | 中等，限于预定义函数集 |
| 训练成本 | 无 | 无 | 需要一次 LoRA 微调 |
| 推理效率 | 需输出 Thought 文本，token 消耗大 | 代码长度不确定 | token 最省（可省略推理） |
| 灵活性 | 高，随时换工具集 | 高，可执行任意代码 | 低，工具集变化需重新训练或推理时更新 prompt |
| 稳定性 | 中等，复杂场景可能格式走样 | 中等，代码可能执行错误 | 较高，格式一致性最好 |

## 代码实践

### 微调实验

**环境安装**

```python
!pip install -q -U bitsandbytes peft trl transformers datasets
```

**库引入与配置**

```python
from transformers import AutoModelForCausalLM, AutoTokenizer, TrainingArguments
from datasets import load_dataset
from trl import SFTConfig, SFTTrainer
from peft import LoraConfig, TaskType
import torch, json

seed = 42
torch.manual_seed(seed)
```

**数据准备**

```python
model_name = "google/gemma-2-2b-it"
dataset_name = "Jofthomas/hermes-function-calling-thinking-V1"

tokenizer = AutoTokenizer.from_pretrained(model_name)
dataset = load_dataset(dataset_name)
dataset = dataset.rename_column("conversations", "messages")

# 预处理：将 system message 合并到第一条 user message 中
def preprocess(sample):
    messages = sample["messages"]
    first_message = messages[0]
    if first_message["role"] == "system":
        system_content = first_message["content"]
        messages[1]["content"] = (
            system_content
            + "Also, before making a call to a function take the time "
              "to plan the function to take. Make that thinking process "
              "between <think>{your thoughts}</think>\n\n"
            + messages[1]["content"]
        )
        messages.pop(0)
    return {"text": tokenizer.apply_chat_template(messages, tokenize=False)}

dataset = dataset.map(preprocess, remove_columns="messages")
dataset = dataset["train"].train_test_split(0.1)

# 用小样本加速演示（实际训练应使用全量数据）
dataset["train"] = dataset["train"].select(range(100))
dataset["test"] = dataset["test"].select(range(10))
```

**添加 Special Tokens 并扩展模型词表**

```python
from enum import Enum

class ChatmlSpecialTokens(str, Enum):
    tools = "<tools>"
    eotools = "</tools>"
    think = "<think>"
    eothink = "</think>"
    tool_call = "<tool_call>"
    eotool_call = "</tool_call>"
    tool_response = "<tool_response>"
    eotool_response = "</tool_response>"
    pad_token = "<pad>"
    eos_token = "<eos>"
    
    @classmethod
    def list(cls):
        return [c.value for c in cls]

tokenizer = AutoTokenizer.from_pretrained(
    model_name,
    pad_token=ChatmlSpecialTokens.pad_token.value,
    additional_special_tokens=ChatmlSpecialTokens.list()
)

model = AutoModelForCausalLM.from_pretrained(
    model_name,
    attn_implementation='eager',
    device_map="auto"
)
model.resize_token_embeddings(len(tokenizer))
model.to(torch.bfloat16)
```

**LoRA 配置**

```python
peft_config = LoraConfig(
    r=16,
    lora_alpha=64,
    lora_dropout=0.05,
    target_modules=[
        "gate_proj", "q_proj", "lm_head", "o_proj",
        "k_proj", "embed_tokens", "down_proj", "up_proj", "v_proj"
    ],
    task_type=TaskType.CAUSAL_LM,
)
```

**训练配置与启动**

```python
training_arguments = SFTConfig(
    output_dir="gemma-2-2B-it-thinking-function_calling-V0",
    per_device_train_batch_size=1,
    per_device_eval_batch_size=1,
    gradient_accumulation_steps=4,
    eval_strategy="epoch",
    logging_steps=5,
    learning_rate=1e-4,
    max_grad_norm=1.0,
    num_train_epochs=1,
    warmup_ratio=0.1,
    lr_scheduler_type="cosine",
    max_seq_length=1500,
    bf16=True,
    gradient_checkpointing=True,
    packing=True,
    report_to="tensorboard",
)

trainer = SFTTrainer(
    model=model,
    args=training_arguments,
    train_dataset=dataset["train"],
    eval_dataset=dataset["test"],
    processing_class=tokenizer,
    peft_config=peft_config,
)

trainer.train()
trainer.save_model()
# trainer.push_to_hub("你的用户名/gemma-2-2B-it-thinking-function_calling-V0")
```

**推理验证**

```python
from peft import PeftModel

peft_model_id = "你的用户名/gemma-2-2B-it-thinking-function_calling-V0"
model = AutoModelForCausalLM.from_pretrained(
    config.base_model_name_or_path, device_map="auto"
)
tokenizer = AutoTokenizer.from_pretrained(peft_model_id)
model.resize_token_embeddings(len(tokenizer))
model = PeftModel.from_pretrained(model, peft_model_id)
model.eval()

# 测试 prompt（模板化）
prompt = """<bos><start_of_turn>human
You are a function calling AI model. You are provided with function signatures within <tools></tools> XML tags.
Here are the available tools:
<tools> [{'type': 'function', 'function': {'name': 'convert_currency', ...}}] </tools>

Hi, I need to convert 500 USD to Euros. Can you help me with that?<end_of_turn><eos>
<start_of_turn>model
<think>"""

inputs = tokenizer(prompt, return_tensors="pt", add_special_tokens=False)
outputs = model.generate(
    **inputs, max_new_tokens=300,
    do_sample=True, top_p=0.95,
    temperature=0.01, repetition_penalty=1.0,
    eos_token_id=tokenizer.eos_token_id
)
print(tokenizer.decode(outputs[0]))
```

**预期输出结构：**

```
<think>The user wants to convert 500 USD to Euros. I should use the convert_currency function with amount=500, from_currency="USD", to_currency="EUR".</think>
<tool_call>
{"name": "convert_currency", "arguments": {"amount": 500, "from_currency": "USD", "to_currency": "EUR"}}
</tool_call>
```

## 思考与疑问

**1. Function Calling 和 ReAct 本质上是同一回事的不同实现**

课程把两者区分得很清楚，但仔细想想，Function Calling 微调后的 `<think>` -> `<tool_call>` -> `<tool_response>` 循环不就是 ReAct 的 Thought -> Action -> Observation 吗？区别只在于前者是模型内化的能力、输出 token 级别可控，后者是靠外层框架解析。所以更准确的说法是：Function Calling 是 ReAct 的一种**硬化实现**。

**2. Special Token 的选择值得仔细推敲**

Notebook 中选择了 ChatML 风格的 `<>` 标记，但有些模型（如 Llama 系列）有自己原生的函数调用格式。是跟随模型的"母语"风格还是统一用 ChatML？不同的选择影响训练效率和模型泛化能力。这里 Gemma 对 ChatML 的兼容性不完全（不支持 system role），只能做 workaround——说明跨模型的函数调用标准化仍有很长的路要走。

**3. LoRA 的 target_modules 选择**

课程中 target 了包括 `embed_tokens` 和 `lm_head` 在内的 9 个模块。通常我们只 target 注意力层（q_proj, v_proj），但这里扩展了范围。一个可能的原因是：新增的 special tokens 改变了 embedding 空间的分布，需要让 `embed_tokens` 也参与适配。这是合理的，但我怀疑如果只 target q/v + lm_head，效果可能也差不多。值得做 ablation study。

**4. 数据集和评估是真正的瓶颈**

Notebook 只用了 100 条训练数据做演示，这在生产环境中远远不够。更根本的问题是：目前高质量的 function calling 微调数据集集中在少数几个（Hermes、Glaive），而且大多是 LLM 生成的合成数据。真实世界的工具调用数据（特别是多步、多工具的复杂场景）非常稀缺。评估方面，课程只做了肉眼观察生成结果，没有量化指标。如果真要做生产级微调，至少需要格式正确率、参数准确率和端到端任务完成率三个维度的评估。

**5. 微调 vs Prompt 的取舍不是二选一**

我的理解是，应该把两者看作互补而不是对立：
- 对常见、固定的工具集，微调值得投入——换来的是更稳定的格式和更少的 token 消耗
- 对快速变化、实验性的工具集，ReAct Prompt 方式更灵活——换工具只需改 prompt 无需重训
- 两者可以共存：用微调保证核心工具的精准度，用 Prompt 扩展新工具

**6. 这对 Agent 框架设计意味着什么**

如果模型本身就原生支持 Function Calling，那 Agent 框架可以做得更薄——不需要复杂的 prompt 工程来保证输出格式，不需要 Stop-and-Parse 做大量的后处理。框架只需负责：维护工具注册表、调度执行、管理对话历史。这让我想到 MCP 协议的价值：当工具接口标准化后，模型可以在训练时就见过各种工具的调用方式，效果应该比统一 prompt 更好。

## 关键收获

1. **Function Calling 的核心是让模型在训练阶段内化工具调用能力**，而不是在推理时靠 Prompt 引导。这带来了更稳定的输出格式和更少的 token 消耗，但代价是需要额外的训练成本。

2. **LoRA 微调让 Function Calling 的门槛大幅降低**——消费级 GPU 就能在 2B 模型上跑通，adapter 文件只有几百 MB。这意味着个人开发者也能为自己的场景定制函数调用能力。

3. **Special Token 是 Function Calling 微调的关键设计决策**，它决定了模型如何在推理、函数调用和观察结果之间做 token 级别的区分。不同的 token 设计会影响训练效率和模型的泛化行为。

4. **数据质量和评估体系是实际落地的瓶颈**——高质量的 function calling 数据集稀缺，而目前缺乏量化的评估标准。如果你要做生产级微调，这两块投入可能比训练本身更重要。

5. **微调不是 ReAct 的替代品，而是互补方案**：微调保证常用工具的精准度，Prompt 保持新工具的灵活性。理解各自的适用场景，才能做出合理的技术选型。
