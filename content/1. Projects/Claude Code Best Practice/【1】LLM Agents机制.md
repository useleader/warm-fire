---
publish: true
---

# 【1】LLM Agents机制：为什么agent这个抽象存在

> 理解agent的本质——有边界的规划循环，以及Claude Code如何通过frontmatter实现这套机制。

## 先修知识
- [[【序】LLM基础速通]]

## 学习目标
- 理解ReAct框架（Reasoning + Acting交替）
- 理解Tool Use的本质——不是函数调用，是LLM生成的文本指令
- 理解Loop为什么会发生以及如何防止
- 理解Multi-agent协作的两种模式
- 理解Claude Code agent frontmatter字段的理论映射

---

## 1. ReAct框架：推理与行动的交替

### 核心思想

这个涉及到2022的一篇论文，介绍了ReAct框架，可以阅读一下经典，这个Yao是谁，yao shun yu吗，里面讲的内容是将两部分能力整合起来了，thought and action，二者交替可以产生良好的协同效应。

其中 reasoning traces 帮助模型归纳、追踪、更新行动计划，处理异常；
首先这个是什么，指的是 reasoning 与 action的结合吗
处理异常是指什么，思维过程的异常还是操作行为的异常，工具的调用也是它输出一个指令，如果这个错误应该也是算作思考环节，如果是指原本操作的对象是有异常的，则AI在act时干的事情就是处理异常

actions 使模型能与外部环境交互，收集额外信息；这个交互是通过什么，通过function calling 是一种方式，但是这个本身就是一种工具的调用，而非模型本身的能力，一个比较好理解的例子就是，当前的大模型相当于一个大脑，然后各种工具都是肢体，这个肢体越多样，它获取信息的渠道越多样，但是这个都是针对模型本身来说的，难道没有办法让大模型直接具备肢体吗，就是它不只是在一味的调用工具，而是在有需要时自行进化出所谓的肢体来收集信息

reasoning为act提供规划框架，act为reasoning 提供信息反馈
提供的规划框架指的是把具体的指令发给bash等终端，然后进行操作吗，如果是这样的话那agent的能力应该是小于等于bash的；
然后信息反馈的内容是什么：指令执行的结果吗，反馈的格式是什么？

### 为什么要交替？

知行合一了，先思考干啥，然后去行动，接着根据行动的反馈观察再思考

### 提升数据

效果还是很好的，但本质上还是一种工程问题，当前所有的 agent 框架 LangChain, LlamaIndex, AutoGen 本质上都是 ReAct + tooling 的实现  tooling 指的是 function calling吗

---

## 2. Tool Use的本质

### 常见误解

应该叫function calling 就行，我都不知道 RPC function call是什么，也不会混淆

### 真实机制

这块感觉就是将ReAct架构的Act部分正式实现了，模型生成一个结构化的文本指令请求，描述好具体要执行的操作与参数；然后由外部代码执行，结果再反注入回 conservation

模型永远不执行任何操作，始终是一个大脑的状态，只输出想要什么样的数据结构

我有个问题，就是谁来将正常输出的JSON格式化，他怎么知道哪些信息是要执行的，而不是仅仅是在说的内容

### 关键含义

在对话过程中有一个tool_use

---

## 3. Loop为什么会发生

### 健康循环 vs 不健康循环

### 不健康循环的根因

### Claude Code的防护机制

---

## 4. Multi-agent协作

### 串行Delegation（线性依赖链）

### 并行Decomposition（独立子任务）

### 通信协议

---

## 5. Claude Code Agent Frontmatter映射

### 核心字段

### Agent类型

---

## 练习题

## 相关概念
- [[【4】Subagents深度指南]]
- [[【2】Claude Code配置体系]]
- [[【6】Skills与Commands]]

## 参考
