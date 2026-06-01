---
title: "Why Language Models Hallucinate"
publish: true
source: OpenAI
url: https://openai.com/blog/why-language-models-hallucinate
date: 2025-09-05
tags:
  - AI
  - Hallucination
  - Language Models
---

# Why Language Models Hallucinate

## 核心内容

OpenAI 的新研究论文认为，语言模型产生幻觉是因为标准的训练和评估程序奖励猜测而非承认不确定性。

## 主要发现

1. **幻觉的根本原因**：标准训练和评估程序奖励猜测而非承认不确定性

2. **三类回答**：对于有单一"正确答案"的问题，响应可分为三类：
   - 准确的回答
   - 错误
   - 放弃回答（abstentions）

3. **模型Spec**：强调承认不确定性是核心价值观之一

4. **评估问题**：广泛使用的基于准确性的评估需要更新，以减少对猜测的奖励

## 解决方案方向

- 主要记分牌继续奖励幸运猜测，模型将继续学习猜测
- 修复记分牌可以广泛采用幻觉减少技术

## 进展

最新模型的幻觉率较低，OpenAI 继续努力进一步降低大型语言模型输出自信错误的发生率。

---
*来源：OpenAI Blog*
