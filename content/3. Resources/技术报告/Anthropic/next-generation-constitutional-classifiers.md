---
title: "Next-generation Constitutional Classifiers: More Efficient Protection Against Universal Jailbreaks"
publish: true
source: Anthropic
url: https://www.anthropic.com/research/next-generation-constitutional-classifiers
date: 2025
tags:
  - AI
  - Security
  - Jailbreak Defense
---

# Next-generation Constitutional Classifiers: More Efficient Protection Against Universal Jailbreaks

## 核心内容

Anthropic 开发了下一代 Constitutional Classifiers++，在不影响性能的情况下显著提高了对通用越狱攻击的防护能力。

## 技术突破

1. **双阶段架构**：
   - 探针检查 Claude 的内部激活（非常便宜）
   - 更强大的分类器进行二次检查

2. **显著改进**：
   - 拒绝率降低 87%（从原系统的 0.38% 降至 0.05%）
   - 仅增加约 1% 的计算开销

3. **鲁棒性测试**：
   - 超过 1,700 小时的 red-teaming
   - 198,000 次攻击尝试
   - 尚未发现通用越狱方法

## 创新点

新系统比之前的版本更加鲁棒，具有最低的成功攻击率，并且显着降低了误拒绝率。

---
*来源：Anthropic Research*
