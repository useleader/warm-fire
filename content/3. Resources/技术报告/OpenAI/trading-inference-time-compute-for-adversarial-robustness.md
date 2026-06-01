---
title: "Trading Inference-Time Compute for Adversarial Robustness"
publish: true
source: OpenAI
url: https://openai.com/index/trading-inference-time-compute-for-adversarial-robustness
date: 2025-01-22
tags:
  - AI
  - Adversarial Robustness
  - Inference-Time Compute
---

# Trading Inference-Time Compute for Adversarial Robustness

## 核心内容

一篇新论文展示了初步证据，表明增加推理时计算——让推理模型有更多时间和资源"思考"——可以提高对多种类型攻击的鲁棒性。

## 主要发现

1. **推理模型的优势**：像 o1 这样的推理模型在思考时间更长时变得更鲁棒

2. **实验设置**：使用 OpenAI 的 o1-preview 和 o1-mini 进行实验，使用静态和自适应攻击方法

3. **攻击成功率**：随着推理时计算增加，攻击成功率通常下降至接近零

4. **计算量影响**：模型不知道自己正在受到攻击，改进的鲁棒性完全是由于改善推理时计算

## 重要价值

- 推理时扩展可用于处理甚至不可预见的攻击
- 这表明推理时扩展是提高对抗鲁棒性的有前途的方向

---
*来源：OpenAI Research*
