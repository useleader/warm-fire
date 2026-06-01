---
title: "Scaling LLM Inference: Innovations in Tensor Parallelism, Context Parallelism, and Expert Parallelism"
publish: true
source: Meta AI
url: https://engineering.fb.com/2025/10/17/ai-research/scaling-llm-inference-innovations-tensor-parallelism-context-parallelism-expert-parallelism/
date: 2025-10-17
tags:
  - AI
  - LLM Inference
  - System Optimization
---

# Scaling LLM Inference: Innovations in Tensor Parallelism, Context Parallelism, and Expert Parallelism

## 核心内容

Meta 分享如何开发和实现先进的并行技术来优化与资源效率、吞吐量和延迟相关的关键性能指标。

## 三种并行技术

### 1. Tensor Parallelism (TP)
用于将模型分布在多个 GPU 上。

### 2. Context Parallelism (CP)
帮助处理极长上下文（如 Llama 4 引入的 1M/10M token 能力）。

### 3. Expert Parallelism (EP)
用于扩展混合专家（MoE）模型，其中大量"专家"（神经网络模块）使整个模型无法放入单个主机。

## 未来方向

1. **N-D 并行性**：跨节点的多维并行（CP、PP、EP、TP）
2. **预填充和解码分层**：允许更好的资源平衡
3. **异构硬件**：计算密集型硬件用于预填充，内存带宽密集型硬件用于解码

---
*来源：Meta Engineering Blog*
