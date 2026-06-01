---
publish: true
---

# 【7】Token计量与优化：LLM使用的成本决策框架

> 覆盖：输入/输出/rthinking token计费逻辑、prompt caching机制、context膨胀的真实成本、compact vs subagent切分的成本对比、RAG vs 全量灌入的取舍、Claude Code工程的token优化实践。

## 先修知识
- [[【序】LLM基础速通]]
- [[【3】Attention与Reasoning]]

## 学习目标
- 理解输入/输出/rthinking token计费规则
- 理解prompt caching机制
- 理解context膨胀的真实成本
- 掌握token优化策略
- 理解Claude Code token经济

---

## 1. Token计费基本逻辑

### 输入token

### 输出token

### rthinking token

### 计费规则总结

---

## 2. Prompt Caching机制

### cache原理

### cache hit计费差异

### 使用场景

### 配置方式

---

## 3. Context膨胀的真实成本

### O(n²)计算复杂度

### 性能随距离衰退

### 双重成本叠加

### 成本对比实例

---

## 4. Token优化策略

### 模型降级

### 禁用Extended thinking

### Subagent切分

### MCP外置

### RAG vs 全量灌入

---

## 5. Claude Code token经济

### Autocompact阈值

### Spinner verb优化

### Status line

### 决策框架优先级

---

## 练习题

## 相关概念
- [[【3】Attention与Reasoning]]
- [[【9】上下文管理策略]]
- [[【2】Claude Code配置体系]]

## 参考
