---
title: Context Course 课程学习
publish: false
publish: true
---

Hugging Face 官方出品的 [Context Engineering 课程](https://huggingface.co/learn/context-course/)，系统学习如何为 AI Code Agent 构建高效上下文。课程从 Agent Skills、MCP 协议、插件系统、子智能体、钩子系统到从零实现 Agent Loop，覆盖了上下文工程的完整技术栈。

## 课程笔记

| 单元 | 主题 | 核心内容 |
|------|------|---------|
| [[Unit 1 - Agent技能]] | Agent Skills 开放标准 | Context Engineering、SKILL.md 格式、渐进式加载、Skill vs Prompt |
| [[Unit 2 - MCP协议]] | Model Context Protocol | Host-Client-Server 架构、Tools/Resources/Prompts、JSON-RPC、Token 开销分析 |
| [[Unit 3 - 插件系统]] | Plugin 打包与分发 | Prompt→Skill→MCP→Plugin 演进、Manifest 结构、Marketplace 生态 |
| [[Unit 4 - 子智能体]] | 多 Agent 协作模式 | Fan-Out/Pipeline/Supervisor/Swarm、何时该用/不该用 |
| [[Unit 5 - 钩子系统]] | Hooks 规则引擎 | 生命周期事件、Observability/Guardrails/Automation 三种模式 |
| [[Unit 6 - Nano Harness迷你循环]] | 从零构建 Agent Loop | Code-First Agent、沙箱执行、~220 行源码解读 |

## 关键收获

1. **上下文工程是分层的**：Prompt → Skill → MCP → Plugin → Subagent → Hook，每一层解决不同维度的问题
2. **Skill 提供知识，MCP 提供能力**：两者互补，Skill 用渐进式加载节省 Token，MCP 用全量描述保证能力可见性
3. **Plugin 是打包层**：不是新技术，而是将 Skills + MCP + Integrations 打包为一键安装的分发单元
4. **Hooks 是确定性护栏**：模型是概率性的，规则必须是确定性的——Hooks 填补了这个空白
5. **Agent Loop 是这一切的引擎**：理解 ~220 行的 Nano Harness 就理解了所有 Agent 的运作原理

## 相关笔记

- [[AI Agents 课程学习]] — 更偏 Agent 基础概念的课程
