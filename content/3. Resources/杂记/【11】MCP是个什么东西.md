---
publish: true
---

> [(25 封私信 / 63 条消息) 火爆 AI 编程圈的 MCP 到底是个什么东西？ - 知乎](https://zhuanlan.zhihu.com/p/26834797144)


MCP 是Anthropic (Claude)主导发布的一个开放的、通用的、有共识的协议标准。
Model Context Protocol
- MCP 是一个标准协议，就像给 AI 大模型装了一个 “万能接口”，让 AI 模型能够与不同的数据源和工具进行无缝交互。它就像 USB-C 接口一样，提供了一种标准化的方法，将 AI 模型连接到各种数据源和工具。
- MCP 旨在替换碎片化的 Agent 代码集成，从而使 AI 系统更可靠，更有效。通过建立通用标准，服务商可以基于协议来推出它们自己服务的 AI 能力，从而支持开发者更快的构建更强大的 AI 应用。开发者也不需要重复造轮子，通过开源项目可以建立强大的 AI Agent 生态。
- MCP 可以在不同的应用 / 服务之间保持上下文，从而增强整体自主执行任务的能力。

MCP 遵循客户端 - 服务器架构，包含以下几个核心部分：

- **MCP 主机（MCP Hosts）**：发起请求的 AI 应用程序，比如聊天机器人、AI 驱动的IDE 等。
- **MCP 客户端（MCP Clients）**：在主机程序内部，与 MCP 服务器保持 1:1 的连接。
- **MCP 服务器（MCP Servers）**：为 MCP 客户端提供上下文、工具和提示信息。
- **本地资源（Local Resources）**：本地计算机中可供 MCP 服务器安全访问的资源，如文件、数据库。
- **远程资源（Remote Resources）**：MCP 服务器可以连接到的远程资源，如通过 API 提供的数据。
比如说我现在使用的Cherry，就算是MCP Client，MCP服务器是额外提供上下文、工具、prompt的地方
![[Pasted image 20251109114545.png]]
> [MCP 终极指南](https://guangzhengli.com/blog/zh/model-context-protocol)
> 学习的教程