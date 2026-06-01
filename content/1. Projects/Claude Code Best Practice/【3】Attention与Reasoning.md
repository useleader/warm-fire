---
publish: true
---

# 【3】Attention与Reasoning：LLM内部过程的工程映射

> 理解Attention的计算本质、Chain-of-Thought的内部机制，以及reasoning model与fast model的取舍逻辑。

## 先修知识
- [[【序】LLM基础速通]]
- [[【1】LLM Agents机制]]

## 学习目标
- 理解Attention计算的本质（QK^T加权求和）
- 理解O(n²)复杂度对Context Window的限制
- 理解visible vs invisible reasoning tokens
- 理解reasoning model vs fast model的工程取舍
- 掌握Context窗口截断的应对策略

---

## 1. Attention的计算本质

### 核心公式

### QK^T决定"看哪里"

### O(n²)复杂度

### 优化技术

---

## 2. Chain-of-Thought的内部机制

### Visible vs Invisible Reasoning Tokens

### "Let me think step by step"为什么有效

### 功能分叉（Iteration Head）

---

## 3. Reasoning Model vs Fast Model

### 架构差异

### 性能基准对比

### 工程选型建议

---

## 4. Context窗口截断与应对

### Lost in the Middle问题

### 首尾强化策略

### Subagent做上下文隔离

---

## 5. 工程启示

### 基于Attention成本的建议

### 基于推理模型特性的建议

### 基于Context窗口的建议

---

## 练习题

## 相关概念
- [[【7】Token计量与优化]]
- [[【9】上下文管理策略]]
- [[【2】Claude Code配置体系]]

## 参考
