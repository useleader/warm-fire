---
title: 深入理解 BPE：字节对编码
publish: true
---

# 深入理解 BPE：字节对编码

BPE（Byte Pair Encoding，字节对编码）是一种广泛应用于 NLP 的子词切分算法，是现代大语言模型（如 GPT、RoBERTa）的重要基础技术。本文将带你深入理解 BPE 的核心原理，并与相关算法进行对比。

## 为什么需要 BPE？

在自然语言处理中，我们通常面临两个核心问题：

1. **OOV（Out-of-Vocabulary）问题**：传统词级别词表无法处理未见过的单词
2. **词表膨胀**：词表越大，Embedding 矩阵和 softmax 层也越大

BPE 通过将文本切分为子词（subword）来解决这些问题。它既保留了词的语义完整性，又通过共享子词单元来处理低频词和未知词。

## 核心原理

BPE 的核心思想非常简单：**迭代合并频率最高的字符对**。

### 训练流程

1. **初始化词表**：将每个字符作为独立的 token
2. **统计字符对频率**：计算所有相邻字符对的出现频率
3. **合并最高频字符对**：将频率最高的字符对合并为一个新 token
4. **重复步骤 2-3**：直到达到预设的词表大小

### 可视化示例

以单词 `"low"` 为例（假设 ░ 表示空格）：

```
初始词表: l o w ░
第1步合并: lo w ░ (假设 "lo" 频率最高)
第2步合并: low ░ (假设 "low" 频率最高)
最终结果: [low]
```

更完整的例子：

```
原始词频统计:
"low": 2, "lower": 2, "lowest": 1, "new": 5, "low": 3

初始切分:
l o w ░, l o w e r ░, l o w e s t ░, n e w ░, l o w ░

合并 "o"+"w" → "ow":
l ow ░, l ow e r ░, l ow e s t ░, n e w ░

合并 "e"+"r" → "er":
l ow ░, l ow er ░, l ow e s t ░, n e w ░

...
```

## 与其他切词方法对比

| 特性 | BPE | WordPiece | Unigram |
|------|-----|-----------|---------|
| 合并策略 | 频率最高的字符对 | 最大化语言模型概率 | 最大化词表概率 |
| 学习方式 | 频率统计 | 语言模型评估 | EM 算法 |
| 词表构建 | 贪心 | 贪心 | 迭代减法 |
| OOV 处理 | 共享子词 | 共享子词 | 共享子词 |
| 典型应用 | GPT, BART | BERT | SentencePiece |

## 实际应用

### 编码示例

```python
from tokenizers import Tokenizer
from tokenizers.trainers import BpeTrainer

# 初始化 tokenizer
tokenizer = Tokenizer()
trainer = BpeTrainer(vocab_size=30000, min_frequency=2)

# 训练
files = ["train.txt"]
tokenizer.train(files, trainer)

# 编码
output = tokenizer.encode("Hello, world!")
print(output.tokens)  # ['Hello', '▁', 'world', '!']
```

### 解码示例

```python
# 解码：将 token IDs 转回文本
decoded = tokenizer.decode(output.ids)
print(decoded)  # "Hello, world!"
```

## 优缺点

### 优点

- **子词共享**：低频词可以通过公共子词表示，减少词表大小
- **平衡性**：通过合并频率控制，既不会太细碎，也不会太粗糙
- **可逆性**：编码/解码完全可逆，不会丢失信息
- **处理 OOV**：有效处理未见过的单词

### 缺点

- **贪心近似**：无法保证全局最优的词表
- **无跨字符依赖**：无法学习跨越合并边界的信息
- **固定词表**：一旦训练完成，词表固定，无法动态调整

## 延伸阅读

- [Hugging Face Tokenizers 文档](https://huggingface.co/docs/tokenizers)
- [SentencePiece: A simple language-independent subword tokenizer](https://github.com/google/sentencepiece)
- [GPT-2 Paper: Language Models are Unsupervised Multitask Learners](https://d4mucfpksywv.cloudfront.net/better-language-models/language-models.pdf)