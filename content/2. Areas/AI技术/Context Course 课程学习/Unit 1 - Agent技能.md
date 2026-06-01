---
title: Unit 1 - Agent 技能
publish: true
course_url: https://huggingface.co/learn/context-course/unit1/introduction
---

最近开始学习 Hugging Face 官方出品的 [Context Course](https://huggingface.co/learn/context-course/)，这是 Unit 1 的学习笔记，聚焦于一个正在改变 AI Agent 生态的开放标准——Agent Skills。本单元从上下文工程（Context Engineering）的概念切入，系统介绍了 Agent Skills 解决的问题、格式规范、渐进式加载机制和分发生态。笔记保留了我学习过程中的疑问和思考，作为整理思路的方式。

## 课程章节清单

- [ ] Introduction — 为什么 Context Engineering 重要
- [ ] What Are Agent Skills? — 技能定义、解决的问题、vs Prompt
- [ ] The Agent Skills Specification — 开放标准、跨平台兼容
- [ ] The SKILL.md Format — 元数据、目录结构、渐进式加载
- [ ] Building Your First Skill — 动手构建 Dataset Validation Skill
- [ ] Debugging Skills — 调试激活与描述词
- [ ] Sharing Skills — 版本管理、团队分发
- [ ] Unit 1 测验
- [ ] 总结

## 核心概念

### Context Engineering 上下文工程

没有恰当的上下文，Agent 会利用猜测填充空白，问出不该问的问题，反复把时间浪费在重做上，这使得它们可能在很多任务中失败。

一个直觉反应是：让模型在不确定时主动确认不就行了？但问题在于，LLM 的运作方式是预测下一个 token——当上下文缺失时，它根本**意识不到自己在猜**。它不会输出"我不确定"，而是基于训练数据中的统计规律生成一个听起来合理的答案。上下文工程的目的就是在模型开口之前把缺口填上，让它不需要猜。如果反过来每件事都确认，一个简单任务要来回十几轮，用户体验同样糟糕。好的上下文让模型在该确认时才确认，其余时候有足够信息自主决策。

所以上下文工程的定义就是：**对上下文进行结构化，使智能体能够找到并使用的实践**。

### Agent Skills 是什么

Agent Skills 是一个自成一体的知识包，与 Tool 并不是一个东西。它使 Agent 在特定任务上表现出色，技能在项目和 Agent 之间可移植，安装后可重复使用，结构化为 Agent 可解析的格式，并通过脚本和 API 连接进行扩展。

一个 skill 本质上是一个 Agent 的"入职文档"：类似于解决特定任务的逐步指南，只不过可以按需加载。这带来一个自然的问题——一个合格的工人显然不止需要一份入职文档，复杂任务可能需要多个 skill 联合，或者需要不同分工的人各尽所能。怎么处理？

答案就在 skill 的设计理念中：skill 被设计为**可组合的连贯工作单元**（coherent units that compose well）。一个复杂任务可以同时激活多个 skill——比如"分析数据并发布到 Hugging Face"可能触发 `data-analysis` 和 `dataset-publishing` 两个 skill。这不是 bug，是设计目标。

更进一步，规范中还描述了 **Subagent Delegation** 模式：将复杂任务拆解后，派子 Agent 各自加载相关 skill 在独立会话中执行，避免主对话上下文被污染，也防止多个 skill 的指令互相干扰。这正好印证了"拆解复杂任务、各司其职"的思路。

Skill 通常以目录形式存在，目录包含 `SKILL.md`（记录元数据与指令），以及可选的 `scripts/`、`references/`、`assets/` 子目录。通过 skill 之间的引用，针对特定任务可以通过各种子任务组合完成复杂工作。

### Portable Knowledge 与 Skill 的关系

技能与长提示有一个关键区别：结构化、可复用、可被发现。它超出单一对话，相关时才会加载。规范将 skill 描述为 "portable, version-controlled folders" 的知识包——**Portable Knowledge（可移植知识）就是 skill 的核心定位**：跨项目、跨团队、跨工具复用。

### 为什么需要 Agent Skills

规范需要确保四点：
- 技能能够跨 Agent 与平台使用
- Agent 能够自动发现并加载相关技能
- 社区可以共享和改进技能
- 可以构建支持技能工作流程的工具和平台

一个技能以结构化、可复用的格式打包领域特定知识，Agent 可以自动发现并使用。它可以包含：帮助 Agent 决定何时使用的元数据、完成任务的逐步说明、自动化常见操作的辅助脚本、文档链接与示例代码、常见问题排查指南。

这让我想到 `find-skill` 这类技能的设计意义——它不应该自动加载到每个 Agent 里，而是当某个任务需要其他 skill 时，靠它自动发现并获取。

传统基于提示的上下文有六大问题：

| 问题 | 说明 |
|------|------|
| 不可重用 | 每次都需要在对话中粘贴同一段内容 |
| 不可分享 | 团队成员必须各自复制粘贴 |
| 不可维护 | 一次更新，处处更新 |
| 不可发现 | 其他团队无法共享知识 |
| 不可组合 | 很难将多个领域组合在一起 |
| 未版本化 | 无法跟踪变更或回滚 |

基于技能的方法将知识结构化为可移植的包：
- **文件结构**：`<skill-name>` 目录，内含 `SKILL.md`、`scripts/`、`references/`
- **元数据格式**：带有 `name`、`description` 等 YAML 前置信息
- **发现协议**：Agent 自动发现和加载技能的机制
- **兼容性保证**：Agent 需要实现的内容以支持 skill

这整体结构可以看作 skill 的 `package.json`——确保兼容性并启用工具链。

### SKILL.md 格式规范

#### 渐进式加载的三个阶段

规范将 skill 的加载设计为三层渐进式披露（Progressive Disclosure），这是整个设计的精髓：

| 层级 | 加载内容 | 时机 | Token 成本 |
|------|---------|------|-----------|
| Tier 1 | name + description | 会话启动时 | ~50-100 tokens/skill |
| Tier 2 | SKILL.md body | 匹配到相关任务时 | 建议 <5000 tokens |
| Tier 3 | scripts/references/assets | 指令明确引用到时 | 按需 |

关键洞察：一个装了 20 个 skill 的 Agent，启动时只加载元数据（Tier 1），只有真正用到的 skill 才会展开完整内容。references/ 里的文件不是一次性全加载的——规范建议在 SKILL.md 中写明**何时**读哪个文件（如"API 返回非 200 状态码时读取 `references/api-errors.md`"），这样模型只在遇到对应情况时才加载。

#### 目录结构规则

```
skill-name/
├── SKILL.md          # 必需：元数据 + 指令
├── scripts/          # 可选：可执行代码
├── references/       # 可选：补充文档
├── assets/           # 可选：模板、资源
└── ...               # 其他文件
```

命名规则：
- 目录名必须小写，单词间用连字符连接（`skill-name`，而非 `SkillName`）
- 不允许两个连续的连字符（`--`）
- 必须包含 `SKILL.md`，且必须放在根目录
- 所有相对文件路径都从技能根目录解析

为什么强制小写？三个工程原因：跨平台文件系统兼容（Windows 不区分大小写但 macOS/Linux 区分）、`name` 字段必须精确匹配目录名、URL/CLI 友好（无需转义）。

#### Frontmatter 字段

```yaml
---
name: "dataset-publishing"
description: "Publish datasets to Hugging Face Hub. Use when uploading datasets, creating dataset cards, or managing dataset versions."
license: "Apache-2.0"
compatibility: "Tested with Python 3.8+ and huggingface_hub 0.16+"
metadata:
  author: "ml-team"
  version: "1.0.0"
allowed-tools: "Bash(hf:*) Python(huggingface_hub:*)"
---
```

各字段说明：
- **name**：skill 自身的标识符，必须与父目录名一致，类似 npm 包名——用于查找、引用和去重
- **description**：描述 skill 做什么以及**何时使用**。规范中**没有单独的 "triggers" 字段**——description 承担了触发匹配的全部职责，要同时说清楚"我做什么"和"什么时候用我"
- **license**：许可证信息
- **compatibility**：环境兼容性说明
- **metadata**：自定义键值对，可以放作者、版本等任意信息
- **allowed-tools**：实验性功能，限制了 skill 可以使用的工具范围

关于 `allowed-tools` 为什么是实验性的：不同 Agent 实现的工具命名和能力模型差异巨大——Claude Code 的 `Bash` 和 Gemini CLI 的 `Bash` 能力范围不同；有的叫 `Read`，有的叫 `read_file`。在生态统一工具命名之前，跨平台的工具白名单无法保证语义一致。

`allowed-tools` 的值使用 **glob 通配符模式**（不是 JSON Schema）。例如 `Bash(git:*) Bash(jq:*) Read` 表示允许 Bash 但只能用 `git` 或 `jq` 开头的命令，允许 `Read` 无限制。`git:*` 匹配 `git diff`、`git log` 等所有 git 子命令。

#### Body 内容

SKILL.md 的正文不宜太长（建议不超过 500 行），详细参考材料移到 `references/` 目录。核心原则是 **"添加 Agent 不知道的，省略它知道的"**——写项目特定的约定、非显而易见的边界情况、特定工具的使用方式，而不需要解释基础概念。

关于 Prerequisites Checklist：这是 skill 作者（人）预先识别出的前置条件——LLM 难以自己推断的依赖、环境配置、数据格式约定。这正是"add what the agent lacks"的体现。

### 渐进式加载与调用方式

加载流程分两步：

1. **Discovery（发现）**：启动时扫描所有 skill 的 `name + description`，形成目录。Agent 先不加载 body，仅凭元数据判断哪些 skill 可能相关
2. **Activation（激活）**：当任务匹配某个 skill 的 description，Agent 读取完整 `SKILL.md` 进入上下文

调用技能的两种方式：
- **显式调用**：按名称直接调用（如 Claude Code 的 `/skill-name`）
- **隐式激活**：Agent 根据 description 匹配，自行判断加载

两种机制并存，互不排斥。Claude Code 同时支持手动 slash command 和模型自主判断激活。

### 生态与分发

技能作为可安装的包通过市场分发——**类似 VS Code 插件市场或 npm registry**，社区可以发布 skill，其他人可以搜索和安装。目前生态还在早期，skill 主要通过 git 仓库分发，市场机制正在建设中。

通过 `plugin-namespace:skill-name` 的命名空间前缀来避免命名冲突，类似 npm 的 `@scope/package`。

Agent Skills 格式最初由 Anthropic 开发并以开放标准发布，目前已被大量工具和 Agent 产品支持——包括 Claude Code、Gemini CLI、Cursor、GitHub Copilot、VS Code、OpenCode、Spring AI 等 30+ 平台。

## 思考与疑问

- 模型猜填空白的根源不是"不愿问"，而是 LLM 的自回归生成机制使其根本意识不到不确定性——上下文工程的意义在于从源头消除猜的需求
- 复杂任务的 skill 组合是设计目标：多个 skill 可同时激活，也可通过 Subagent Delegation 派子 Agent 独立执行
- 渐进式加载（三层披露）是 skill 设计的精髓——用最小的上下文成本换取最大的知识覆盖
- `allowed-tools` 的实验性状态反映了跨平台工具命名尚未统一，但不影响当前在单一平台内的使用
- Skill 的能力演化路径：传统 Prompt → Tool 调用 → MCP Server → Skill，每一层都在提升抽象级别和可复用性
- 显式调用和隐式激活并存意味着 skill 既是"命令"也是"建议"——这个双重身份值得在后续实践中验证
