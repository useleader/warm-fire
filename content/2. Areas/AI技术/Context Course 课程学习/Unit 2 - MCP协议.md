---
title: Unit 2 - MCP 协议
publish: true
course_url: https://huggingface.co/learn/context-course/unit2/introduction
---

继 Unit 1 学习了 Agent Skills（静态知识包）之后，Unit 2 进入 MCP（Model Context Protocol）——解决 Agent 如何与外部世界动态交互的开放协议。本单元从"M×N 问题"切入，系统拆解了 MCP 的三角色架构、三种核心能力、JSON-RPC 通信协议和两种传输方式。笔记保留了我学习过程中的疑问和思考，作为整理思路的方式。

## 课程章节清单

- [ ] Introduction — MCP 是什么、Skills + MCP 的关系
- [ ] The M×N Problem — 为什么需要统一协议
- [ ] MCP Architecture — Host、Client、Server 三角色
- [ ] Three Capability Types — Tools、Resources、Prompts
- [ ] Building MCP Servers with FastMCP — 快速搭建 Server
- [ ] Building MCP Servers with Gradio — GUI 驱动的 Server
- [ ] Configuring Agents as MCP Clients — 接入 Claude Code/Codex/OpenCode
- [ ] Deploying to Hugging Face Spaces — 部署与分享
- [ ] Hands-on Project — 综合实战
- [ ] Unit 2 测验
- [ ] 总结

## 核心概念

### MCP 是什么

MCP（Model Context Protocol）是让 AI 应用与外部数据和工具进行交互的开放协议。它位于 Agent 与一切事物之间：文件、数据库、API、服务。

理解 MCP 最直接的方式是对比 Unit 1 的 Agent Skills：

| | Agent Skills | MCP |
|------|---------|------|
| 本质 | 静态、预先编写好的上下文知识 | 动态上下文，可实时调用外部系统 |
| 提供 | 知识和工作流指导 | 工具能力和数据访问 |
| 加载方式 | 渐进式披露（Tier 1→Tier 2→Tier 3） | 工具描述常驻系统提示，按需调用 |
| 典型用途 | "怎么做一个 PDF 处理任务" | "调用 API 获取当前天气、查询数据库" |

一句话总结：**Skill 提供知识，MCP 提供工具与数据**。两者互补——Skill 告诉 Agent 怎么做，MCP 给 Agent 做的能力。

### M×N 问题：为什么需要统一协议

在没有 MCP 之前，每个 LLM 应用要对接每个外部数据源，都需要单独写适配代码——M 个模型 × N 个工具的集成矩阵。MCP 将这个问题从 M×N 降到 M+N：工具只需实现一次 MCP Server，所有兼容 MCP 的 Agent 都能直接使用。这也是 Unit 1 中提到的"能力演化路径"的关键一环：从独立的 Tool 调用，到标准化的 MCP Server。

### MCP 架构：Host、Client、Server

MCP 采用三角色架构：

```
MCP Host (AI 应用，如 Claude Code)
├── MCP Client 1 ──── MCP Server A (本地，如文件系统)
├── MCP Client 2 ──── MCP Server B (本地，如数据库)
├── MCP Client 3 ──── MCP Server C (远程，如 Sentry)
└── MCP Client 4 ──── MCP Server C (远程，可多 Client 连同一 Server)
```

**Host**：运行 Agent 的环境，如 Claude Code、VS Code、Cursor。它负责创建和管理多个 MCP Client。关于 AI SDK——它是构建 Host 这类应用的开发工具包（如 Vercel AI SDK），不是 Host 本身，可以理解为造车平台而非成品车。

**Client**：位于 Host 内部的协议处理程序，与 MCP Server 建立一对一连接。职责包括：发现可用服务器及其能力、向服务器发送请求、处理响应与错误、维护连接生命周期、管理认证。一般内置于 Agent 的运行时中。

**Server**：通过 MCP 暴露能力的外部程序。它定义工具 schema 和描述、提供公共可访问的资源、实现指令模板、处理来自 Client 的请求、以 MCP 协议格式返回结果。

### 通信协议：JSON-RPC 与两层架构

MCP 由两层组成：
- **数据层**（内层）：定义 JSON-RPC 2.0 消息格式和语义，包括生命周期管理、三种核心原语、通知机制
- **传输层**（外层）：定义通信通道，与数据层解耦——同一套 JSON-RPC 消息可在不同传输层上使用

**什么是 JSON-RPC？** 一种以 JSON 为编码格式的远程过程调用协议。请求是一个 JSON 对象（含 `method` + `params` + `id`），响应是另一个 JSON 对象（含 `result` 或 `error` + 对应的 `id`）。轻量、无状态、传输无关。

MCP 定义两种标准传输：

| | Stdio Transport | Streamable HTTP Transport |
|------|---------|---------|
| 适用场景 | 本地连接 | 远程连接 |
| 运行方式 | Server 作为 Host 的子进程 | Server 独立运行，Client 通过 HTTP 访问 |
| 通信通道 | stdin/stdout | HTTP POST 请求 + SSE 流 |
| 网络开销 | 无 | 有（可跨互联网、穿透防火墙） |
| 典型用途 | 开发、本地工具 | 云部署、共享服务 |

**什么是 SSE 流？** SSE（Server-Sent Events）是服务器向客户端的单向 HTTP 推送机制。服务器持续推送事件，客户端无需轮询。在 MCP 中用于传递服务器主动发出的通知（工具列表变更、资源更新等）。比 WebSocket 更轻量，天然支持 HTTP 重连。

### MCP 握手流程

Agent 发起初始化请求 → MCP Server 返回参数格式和能力声明，建立连接 → Agent 请求 tool list → Server 返回 → Agent 请求调用具体工具 → Server 返回调用结果。初始化阶段的关键动作是**能力协商**（Capability Negotiation）——双方互报自己支持的功能，后续交互严格遵循协商结果。

### 三种核心能力：Tools、Resources、Prompts

MCP Server 向 Client 暴露三种原语：

**Tools（工具）**：可执行的函数，AI 应用通过 `tools/list` 发现、通过 `tools/call` 调用。每个工具包含 name、description、inputSchema（JSON Schema 定义参数）。工具的注册表是**动态可变**的——运行时可以添加、移除、修改工具，Server 通过 `listChanged` 通知主动推送变更。这不是指某次调用的参数可变（那显然是可变的），而是工具集合本身可以随 Server 状态变化。

**Resources（资源）**：Agent 可访问的只读数据源，每个资源有一个 URI、可读名称、描述和 MIME 类型。类似 REST API 的 GET 端点，但通过统一协议暴露。

**Prompts（提示模板）**：可复用的指令模板。关键理解：MCP Prompts 是**用户主动触发**的（user-controlled），不是 Server 自动发给 LLM 的工具使用说明。类似 slash command——用户选择后，模板内容注入对话。它可以包含给 LLM 的指令（如"请审查代码"）、few-shot 示例、或结构化对话模板。工具说明书由 `tools/list` 返回的 `description` + `inputSchema` 承担，Prompts 是另一套独立的交互机制。

### 为什么都说 MCP 费 Token？

这是学习 MCP 时最常见的疑问，值得展开：

**开销来源**：
- **工具描述常驻系统提示**：所有 Server 的所有工具的 name + description + inputSchema 全部注入系统提示。一个 Server 有 20 个工具、每个 schema 几百字，合计数千 tokens
- **工具调用结果进入上下文**：每次 `tools/call` 的返回值都不可缓存，持续消耗上下文空间

**对比 Skill 的设计取舍**：

| | MCP Tools | Agent Skills |
|------|---------|------|
| 加载策略 | 全部工具描述前置注入 | 渐进式披露，不激活不展开 |
| 设计目标 | "让模型随时知道有什么能力可用" | "按需加载，最小化上下文成本" |
| 缓存友好度 | 系统提示部分可缓存，结果不可缓存 | Tier 1 极轻（~100 tokens/skill） |

**好消息**：注入系统提示的工具描述可被 prompt caching 缓存，在重复对话中开销不高。真正持续消耗的是每次工具调用的返回数据。

这个取舍是刻意的——MCP 优先保证能力可见性（模型能发现并使用所有工具），Skill 优先保证上下文效率（只加载当前任务需要的知识）。两者互补：**MCP 给 Agent 手脚，Skill 给 Agent 头脑**。

### 错误处理

MCP 的最佳实践是返回结构化的错误信息（含错误码和上下文描述），而非抛出裸异常。输入参数在调用前需验证，记录失败以便调试。工程细节在实际搭建中再具体掌握。

## 思考与疑问

- Skills 和 MCP 是互补而非替代：Skill 提供静态知识与工作流，MCP 提供动态工具与数据，Agent 需要两者兼备
- MCP 的 Token 开销源于"全量工具描述前置"的设计选择，这是能力可见性和上下文效率之间的经典权衡
- JSON-RPC 的传输无关设计让 MCP 可以一套协议覆盖本地开发和云部署两种场景
- 工具注册表的动态可变性（listChanged 通知）让 MCP Server 可以响应运行时变化，这对长生命周期 Agent 尤其重要
- MCP Prompts 的 user-controlled 定位意味着它更像"快捷指令"而非"自动说明书"，与 Tool 的自动调用机制形成互补
- 能力演化路径再次印证：独立 Tool 调用 → 标准化 MCP Server → 更高层的抽象（Skills 编排），每一层都在提升可复用性和互操作性
