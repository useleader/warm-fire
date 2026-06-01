---
title: Beam Search：超越贪心的序列解码
publish: true
---

# Beam Search：超越贪心的序列解码

在序列生成任务（如机器翻译、文本生成）中，我们通常需要从模型输出的概率分布中选择下一个 token。简单的方法是贪心选择概率最高的词，但这种方法容易陷入局部最优。Beam Search（束搜索）是一种折中方案，在计算效率和生成质量之间取得平衡。本文将带你深入理解 Beam Search 的核心原理及其变种。

## 贪心解码的问题

贪心解码（Greedy Decoding）在每一步都选择概率最高的 token：

```
P("The") = 0.3 → 选中
P("cat") = 0.5 → 选中
P("sat") = 0.4 → 选中
```

问题在于：**全局最优路径**可能包含**局部低概率**的 token。

例如，"The cat sat on the mat"（全局最优）的生成路径中，可能在某些步骤的概率低于其他候选，但整体句子才是最合理的。

## 核心原理：每一步保留 Top-k 候选

Beam Search 的核心思想是：**在每一步保留多个候选路径**，而不是只选一个。

### 搜索树可视化

```
Step 1: "The" (0.3)
           ├── "cat" (0.5) → "The cat"
           └── "dog" (0.4) → "The dog"

Step 2: Beam Width = 2
    Path 1: "The cat" (0.3×0.5 = 0.15)
    Path 2: "The dog" (0.3×0.4 = 0.12)
```

### 伪代码实现

```python
def beam_search(model, beam_width=5, max_length=50):
    # 初始化：开始 token
    beams = [("<bos>", 1.0)]

    for step in range(max_length):
        candidates = []
        for seq, score in beams:
            # 生成下一个 token 的概率分布
            probs = model.predict_next(seq)
            # 扩展所有候选
            for token, prob in enumerate(probs):
                new_seq = seq + token
                new_score = score * prob
                candidates.append((new_seq, new_score))

        # 选择 top-k 候选
        beams = heap.nlargest(beam_width, candidates, key=lambda x: x[1])

    # 返回概率最高的路径
    return max(beams, key=lambda x: x[1])[0]
```

## 长度归一化：为什么需要 Length Penalty

直接使用概率乘积会导致短句子占优势（概率乘积更大）。为此，我们引入**长度归一化**：

```
normalized_score = score / (length^alpha)
```

其中 `alpha` 通常取 0.6-0.7。

### 示例

```
路径 A: "Hello" (概率 0.5^5 = 0.03125)
路径 B: "Hello world today" (概率 0.5^6 = 0.015625)

未归一化: A 胜出 (0.03125 > 0.015625)
归一化后 (alpha=0.7):
  A: 0.03125 / 5^0.7 = 0.03125 / 3.34 = 0.00936
  B: 0.015625 / 6^0.7 = 0.015625 / 4.11 = 0.00380
  A 仍然胜出（短句优势被保留但不那么极端）
```

## Beam Search 变种

### 1. Simple Beam Search

最基本的版本，如上面的伪代码所示。

### 2. Normalized Beam Search

引入长度归一化：

```python
def normalized_beam_search(model, beam_width=5, alpha=0.7):
    beams = [("<bos>", 0.0, 0)]  # (sequence, log_prob, length)

    for step in range(max_length):
        candidates = []
        for seq, score, length in beams:
            probs = model.predict_next(seq)
            for token, prob in enumerate(probs):
                new_seq = seq + token
                new_score = score + math.log(prob)
                new_length = length + 1
                # 长度归一化
                length_penalty = (new_length ** alpha)
                normalized = new_score / length_penalty
                candidates.append((new_seq, new_score, new_length, normalized))

        # 基于归一化分数选择
        beams = heap.nlargest(beam_width, candidates, key=lambda x: x[3])

    return beams[0][0]  # 返回最佳路径
```

### 3. Coverage Beam Search

在机器翻译等任务中，我们希望生成的译文**覆盖**源端的所有信息。Coverage 机制惩罚重复生成相同内容：

```
coverage_penalty = sum(min(attention[i], 1.0) for i in range(src_length))
final_score = log_prob + beta * coverage_penalty
```

其中 `beta` 通常取 0.5-1.0。

## Hugging Face Transformers 用法

```python
from transformers import AutoModelForSeq2SeqLM, AutoTokenizer

model_name = "t5-small"
model = AutoModelForSeq2SeqLM.from_pretrained(model_name)
tokenizer = AutoTokenizer.from_pretrained(model_name)

# 输入
input_text = "translate English to German: The weather is nice today"
inputs = tokenizer(input_text, return_tensors="pt")

# Beam Search 解码
outputs = model.generate(
    inputs["input_ids"],
    num_beams=5,           # beam width
    length_penalty=0.6,    # 长度归一化参数
    no_repeat_ngram_size=2, # 防止重复 n-gram
    early_stopping=True     # 遇到 <eos> 时停止
)

result = tokenizer.decode(outputs[0], skip_special_tokens=True)
print(result)  # "Das Wetter ist heute schön"
```

## 优缺点

### 优点

- **局部+全局平衡**：比贪心搜索考虑更多候选路径
- **可控复杂度**：通过 beam_width 控制计算量
- **质量提升**：显著改善生成文本的连贯性

### 缺点

- **计算成本**：是贪心搜索的 beam_width 倍
- **仍非全局最优**：仍然是近似算法，不保证全局最优
- **重复生成问题**：需要配合 no_repeat_ngram_size 等技巧

## 延伸阅读

### Hugging Face 官方文档（强烈推荐）

Hugging Face 提供了目前最详尽的 Beam Search 学习资源，建议按以下顺序学习：

**1. [Generation strategies](https://huggingface.co/docs/transformers/en/generation_strategies)**
官方权威指南，涵盖 Greedy Search、Beam Search、Multinomial Sampling 等所有解码策略的原理与代码示例。

**2. [Text generation strategies](https://huggingface.co/docs/transformers/v4.30.0/generation_strategies)**
图文并茂的早期版本，清晰展示 Beam Search 的搜索树演变过程。

**3. [How to generate text: using different decoding methods](https://huggingface.co/blog/how-to-generate)**
经典博客，对比 Greedy Search、Beam Search 和 Sampling 的优劣，配有直观图解。

**4. [LLM Tutorial](https://huggingface.co/docs/transformers/en/llm_tutorial)**
学习 LLM 生成必读，涵盖 GenerationConfig 和各种生成策略的实战技巧。

### 论文

- [Neural Machine Translation by Jointly Learning to Align and Translate](https://arxiv.org/abs/1409.0473) — 引入 Attention 机制的经典论文
- [Google's Neural Machine Translation System](https://arxiv.org/abs/1609.08144) — Google NMT 系统，覆盖 Beam Search 变种