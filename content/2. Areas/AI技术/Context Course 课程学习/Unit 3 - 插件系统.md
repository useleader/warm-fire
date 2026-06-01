---
title: Unit 3 - 插件系统
publish: true
course_url: https://huggingface.co/learn/context-course/unit3/introduction
---

经过 Unit 1 的 Agent Skills（静态知识包）和 Unit 2 的 MCP（动态工具协议）之后，Unit 3 来到 Plugin（插件系统）——一个将前面两者整合包装的**分发层**。如果说 Skill 是给 Agent 的"知识手册"、MCP 是给 Agent 的"工具箱"，那 Plugin 就是装好手册和工具箱的**快递包裹**，拿到手就能用。本单元从 Prompt 到 Plugin 的演进路径切入，拆解 Plugin 的结构与跨平台差异。笔记保留了我学习过程中的疑问和思考，作为整理思路的方式。

## 课程章节清单

- [ ] Introduction — 从 Prompt → Skill → MCP → Plugin 的演进
- [ ] Plugin Anatomy — Manifest、Skills、MCP Servers、Integrations
- [ ] Claude Code Plugins — Manifest 格式与安装机制
- [ ] Codex Plugins — 插件结构与差异
- [ ] OpenCode Plugins — TypeScript 模块化插件
- [ ] Plugin Marketplace — 发布、发现、共享
- [ ] Hands-on Project — 构建并发布一个 Plugin
- [ ] Unit 3 测验
- [ ] 总结

## 核心概念

### 从 Prompt 到 Plugin 的演进

这是整个 Context Course 的核心叙事线之一。回看 Unit 1 和 Unit 2，能力的发展路径其实是一条清晰的抽象递增链：

```
Prompt → Skill → MCP Server → Plugin
```

每一层都在解决上一层遗留的问题：

| 层级 | 本质 | 解决的问题 | 仍有的不足 |
|------|------|-----------|-----------|
| **Prompt** | 临场编写的自然语言指令 | 让模型按特定方式工作 | 不可复用、不可分享、不可版本化 |
| **Skill** | 结构化的可移植知识包 | 封装领域知识，支持渐进式加载 | 只有知识，没有工具能力 |
| **MCP Server** | 标准化的工具与数据接口 | 统一 Agent 与外部系统的交互协议 | 需要额外配置才能接入，缺乏上下文知识 |
| **Plugin** | Skill + MCP + 配置的整合包 | 一站式分发，即装即用 | 平台尚未完全标准化，生态仍在早期 |

Plugin 本身并没有引入什么全新的技术概念。刚开始学习时，我一度写下"感觉这里并没有什么技术知识"——这句话没错，但它恰恰说中了 Plugin 的定位：**Plugin 不是新技术，而是新包装**。它做的事情本质上和 VS Code 插件或 npm package 一样：把分散的零件（SKILL.md、MCP 配置、辅助脚本）收集起来，加上元数据清单，做成一个可分享、可安装、可卸载的单元。

这个认识很重要——如果以"学新东西"的心态学 Plugin 可能会失望，但如果以"学怎么打包和分发"的心态来看，就会发现整个 Context Engineering 体系开始闭合了：知识有 Skill 管、工具有 MCP 管、分发和复用有 Plugin 管。

### Plugin 的结构

一个 Plugin 的核心组成部分：

**1. Manifest（清单文件）**

插件的 `plugin.json` 或 `manifest.json`，类似 `package.json`。包含：
- 插件名、描述、版本号
- 作者与许可证信息
- 依赖声明（需要哪些 MCP Server、哪些 Skill）
- 平台兼容性标记

**2. Bundled Skills**

Plugin 可以内嵌一个或多个 Agent Skills。安装 Plugin 时，内嵌的 skill 自动注册到 Agent 的 skill 目录中，用户无需手动复制 `SKILL.md`。这解决了 Skill 分发的"文件散落"问题——之前分享一个 skill 需要 git clone、手动放到 `.claude/skills/`；有了 Plugin，一条命令搞定。

**3. Bundled MCP Server 配置**

Plugin 可以声明自己依赖或内嵌哪些 MCP Server，并提供默认配置。安装时 Agent 自动注册这些 Server，用户无需手动编辑 `claude.json` 或配置文件。这对于工具链类的场景尤其有用——"装一个插件，所有的工具自动到位"。

**4. Platform Integrations**

不同平台允许 Plugin 定义额外的集成钩子：
- **Claude Code**：快捷键绑定、slash command 注册、系统提示扩展
- **Codex**：UI 扩展点、活动条图标、侧边栏面板
- **OpenCode**：TypeScript 模块、导出函数注册表

### Plugin vs Skill vs MCP：各自解决什么问题

三者常被放在一起讨论，但它们的定位完全不同：

| 维度 | Agent Skills | MCP Server | Plugin |
|------|-------------|-----------|--------|
| **提供什么** | 知识 + 工作流指导 | 工具 + 数据 | 知识 + 工具 + 配置的整合 |
| **消费方式** | Agent 按需加载（渐进式披露） | Agent 通过 Client 调用 | 用户一次性安装，内部自动分发到 Skills 和 MCP |
| **封装粒度** | 单领域知识包 | 单组工具接口 | 多 skill + 多 MCP 的组合 |
| **是否可独立存在** | 是 | 是 | 否——Plugin 是对前两者的编排 |
| **典型类比** | 一本操作手册 | 一个工具箱 | 一个"即插即用"的工位，含手册和工具箱 |

Plugin 的独特价值在"组合"和"分发"两个词上——没有 Plugin，用户需要手动配置 skill 目录和 MCP 连接参数；有了 Plugin，一次安装自动完成。

### 跨平台差异

虽然 Context Course 推崇开放标准，但目前的 Plugin 实现仍存在明显的平台差异：

**Claude Code Plugin**

插件以 `plugin.json` 为入口元数据文件，声明依赖的 Skills 和 MCP Servers。通过配置文件注册插件，安装后自动展开。Claude Code 对 Skill 的依赖最深——Plugin 的主要价值在于分发 skill 包，MCP 配置更像附加功能。

**Codex Plugin**

Codex 更进一步，允许 Plugin 定义 UI 扩展——活动条图标、侧边栏面板、编辑器上下文菜单等。这意味着 Codex Plugin 不仅是"后台能力包"，也是"前台界面包"。UI 扩展点让 Plugin 可以提供可视化交互界面，而不只是 Agent 的文字交互。

**OpenCode Plugin**

OpenCode 采用 TypeScript 模块化方式定义插件。插件导出函数或能力集，Agent 在运行时动态加载。这种方案更"工程化"——用编程接口替代声明式配置，灵活性更高，但对使用者的技术要求也更高。

| 维度 | Claude Code | Codex | OpenCode |
|------|-------------|-------|----------|
| 元数据格式 | plugin.json | plugin.json | package.json + 导出模块 |
| UI 扩展 | 有限（slash command） | 丰富（面板、菜单、图标） | 有限 |
| Skill 支持 | 核心功能 | 支持 | 支持 |
| MCP 集成 | 声明式配置 | 声明式配置 | 编程式绑定 |
| 安装方式 | 配置文件注册 | 插件市场安装 | npm install + 注册 |

这些差异反映的是各平台的产品哲学——Claude Code 偏向命令行 Agent 体验、Codex 偏向 IDE 深度集成、OpenCode 偏向开源开发者灵活性。

### Plugin Marketplace

Plugin 让 Context Engineering 的生态拼图接近完整：

- **发现**：市场（marketplace）提供搜索、分类、评分，用户按需寻找插件
- **发布**：开发者通过 git 仓库或平台市场发布插件，附带版本号和兼容性标记
- **共享**：团队内部可以通过私有 registry 分发，社区通过公共市场扩散

这与 Unit 1 中提到的 agentskills.io 生态一脉相承——Skill 市场是插件市场的子集或前身。Agent Skills 社区已经建立起 30+ 平台的兼容网络，Plugin 市场则在这个网络上叠加了"组合包"的概念。一个写好的 Plugin 可以同时发布到多个平台的市场。

不过生态仍在早期。目前 Plugin 的分发方式还比较原始——大部分通过 git 仓库 + 手动配置来完成，真正的"一键 marketplace 安装"体验尚未在所有平台上普及。Unit 3 的 Hands-on Project 会带我们从零构建一个 Plugin，到时候再补充实操观察。

## 思考与疑问

- Plugin 没有引入新技术，它做的是**包装和分发**——这个定位和 npm package、VS Code extension 一样。理解这一点比背 plugin.json 的字段重要得多
- 从 Prompt 到 Plugin 的演进链看到一条清晰的抽象递增路径：每一次抽象不是"替代"上一层的，而是"在上一层之上加一个编排层"。Skill 不替代 Prompt，Plugin 不替代 Skill——它们只是在更高的纬度上组织低维的零件
- 跨平台的 Plugin 格式差异本质上反映了各平台的产品哲学差异。要实现统一的 Plugin 标准，需要先在 Skill 规范和 MCP 协议层面达成更高程度的互操作性——这也是 Context Course 推广开放标准的原因
- 如果 Plugin 的主要价值是"组合 + 分发"，那么在没有成熟市场的阶段，Plugin 的实际用处有多大？个人项目中，直接管理 skill 和 MCP 配置可能更灵活；团队和企业场景下，Plugin 的"即装即用"优势才能真正体现
- 好奇 Plugin 的更新机制——如果已安装的 Plugin 发布新版本，Agent 如何感知并升级？目前课程尚未提及，理论上依赖部署平台（git pull、registry webhook）来通知变更
