---
title: Unit 5 - 钩子系统
publish: true
course_url: https://huggingface.co/learn/context-course/unit5/introduction
---

前面四个单元依次覆盖了技能（Skills）、MCP 协议、插件系统和子智能体——这些都是在"扩充能力"这个维度上做文章。但有一个问题始终悬而未决：**谁来约束 Agent 的行为？**

当 Agent 可以调用工具、分派子任务、读写文件时，我们不再只需要教它"做什么"，更需要告诉它"不能做什么"。这就是 Unit 5 的主题——**钩子系统（Hooks）**。它是 Context Engineering 的"规则即代码"层：把行为规范从模型提示词里提取出来，变成确定性的、不可绕过的执行逻辑。

## 课程章节清单

- [ ] Introduction — Hooks 是什么、为什么需要代码化的规则
- [ ] Hook Lifecycle — UserPromptSubmit → PreToolUse → PostToolUse → Stop → SessionEnd
- [ ] Observability Pattern — 捕获每一次工具调用与 Prompt
- [ ] Guardrails Pattern — 阻止危险操作（如 rm -rf /）
- [ ] Automation Pattern — 自动运行 Linter、Formatter、测试
- [ ] Platform Config Shapes — Claude Code / Codex / OpenCode / Pi 差异
- [ ] Building an Agent Activity Dashboard — Gradio 实时可视化
- [ ] Unit 5 测验
- [ ] 总结

## 核心概念

### Hooks 是什么

Hook 是用户定义的、在 Agent 生命周期确定节点上自动触发的处理程序。

听起来很简单，但这个概念背后有一个关键转变：**从"提示规则"到"代码规则"**。

| | 提示词规则（Prompt Rules） | 钩子规则（Hook Rules） |
|---|---|---|
| **执行方式** | 概率性的——模型"可能"遵守 | 确定性的——代码"必须"执行 |
| **时机** | 模型处理提示时参考 | 在模型看到提示之前/工具运行之前触发 |
| **可绕过性** | Agent 可以用"我没有那条规则"来辩解 | Agent 无法绕过，因为钩子在它的执行路径之外 |
| **适合场景** | 语气风格、输出格式、行为偏好 | 安全约束、强制审计、自动化流程 |

换句话说：提示规则是**建议**，钩子规则是**护栏**。如果你想阻止 `rm -rf /`，依赖提示词说"不要删除根目录"是不够的——模型可能误判、可能被越狱提示绕过。钩子则在工具调用真正执行前拦截检查，**模型根本没有机会执行这个操作**。

### 共享生命周期

整个 Agent 平台（Claude Code、Codex、OpenCode、Goose 等）正在趋同于一个标准化的生命周期。所有 Hook 事件可以分为以下几类：

#### Session 事件

| 事件 | 触发时机 | 常见用途 |
|---|---|---|
| `SessionStart` | 新会话开始（新建、恢复或压缩续传） | 初始化记录器、加载外部配置 |
| `SessionEnd` | 会话已结束 | 关闭日志文件、发送摘要报告 |

#### Prompt 事件

| 事件 | 触发时机 | 常见用途 |
|---|---|---|
| `InstructionsLoaded` | 已加载系统指令（如 CLAUDE.md） | 动态注入上下文 |
| `UserPromptSubmit` | 用户提交提示词，但模型尚未看到 | 注入额外指令、改写用户输入 |

#### Tool 事件（最常用的挂钩点）

| 事件 | 触发时机 | 常见用途 |
|---|---|---|
| `PreToolUse` | 工具调用即将运行 | **安全检查**——在工具执行前拦截危险操作 |
| `PermissionRequest` | 即将显示权限提示 | 自动批准或拒绝 |
| `PermissionDenied` | 自动模式分类器否决了工具调用 | 记录被拒绝的操作 |
| `PostToolUse` | 工具调用返回后 | 记录结果、触发自动化流程 |
| `PostToolUseFailure` | 工具调用失败 | 错误处理、重试逻辑 |
| `Stop` | 任务完成 | 执行收尾工作 |

#### Subagent 事件

| 事件 | 触发时机 | 常见用途 |
|---|---|---|
| `SubagentStart` | 派生子代理开始 | 记录子任务创建 |
| `SubagentStop` | 派生子代理结束 | 收集子任务结果 |
| `TaskCreated` | TodoWrite 任务清单新增条目 | 追踪进度 |
| `TaskCompleted` | TodoWrite 任务条目完成 | 验证完成状态 |

#### Environment 事件

| 事件 | 触发时机 | 常见用途 |
|---|---|---|
| `CwdChanged` | 工作目录切换 | 更新路径上下文 |
| `FileChanged` | 被跟踪的文件发生变化 | 触发自动格式化/Lint |
| `WorktreeCreate` | Git worktree 已创建 | 记录工作树信息 |
| `WorktreeRemove` | Git worktree 已移除 | 清理临时文件 |
| `PreCompact` | 上下文压缩即将发生 | 保存当前上下文快照 |
| `PostCompact` | 上下文压缩刚刚完成 | 恢复自定义状态 |
| `ConfigChange` | 设置已更改 | 重新加载配置 |
| `Notification` | Claude 触发了一个通知 | 记录通知 |

在这二十多个事件中，**`PreToolUse` 是最核心的挂钩点**——它是安全护栏的最后一道防线。其次常用的是 `PostToolUse`（可观测性与自动化）和 `UserPromptSubmit`（上下文注入）。

### 三种核心模式

#### 1. Observability（可观测性）：每一次操作都留下痕迹

没有钩子的时候，你只能看到模型输出的文本，而不知道它背后做了什么——调用了什么工具、传递了什么参数、得到了什么结果。钩子让这一切变得透明。

典型的可观测性钩子会：

- 在 `PreToolUse` 记录：什么工具、什么参数、什么时候
- 在 `PostToolUse` 记录：返回结果、耗时
- 在 `UserPromptSubmit` 记录：用户输入的原始内容
- 将所有日志持久化到文件或数据库

这不仅仅是调试工具。在生产环境中，可观测性是**审计合规的基础**——你能回答"谁在什么时候让 Agent 做了什么"。

#### 2. Guardrails（护栏）：阻止危险操作——这是杀手级功能

这是 Hook 系统最强大的模式。Guardrails 的核心逻辑是：**在操作执行之前检查它，如果危险就直接阻止。**

```
UserPromptSubmit → PreToolUse ──→ [Guardrail Check] ──→ 通过 → 执行
                                   ↓
                                 不通过 → 阻止 + 记录日志
```

关键在于：**模型无法绕过这道检查**。Hook 代码运行在模型的控制流之外。即使是模型自己生成的工具调用，也必须先经过 `PreToolUse` 的审查。这意味着：

- 你不需要信任模型不会删除文件——钩子替你拦截 `rm -rf /`
- 你不需要提示词说"不要推送代码"——钩子检查 `git push --force`
- 你不需要相信模型会遵守"先写测试"——钩子在写代码前强制运行测试

这是我在这门课中看到的最重要的安全机制。没有之一。

#### 3. Automation（自动化）：让 Agent 自动遵守工程规范

自动化模式是 Guardrails 的"积极面"——不是在阻止，而是在自动执行操作。

例如，每次文件写之后自动运行：

1. `PostToolUse` 检测到文件写入完成
2. 运行 `npm run format` 格式化代码
3. 运行 `npm run lint` 检查代码规范
4. 运行 `npm test` 执行测试
5. 如果测试失败，阻止 Agent 继续

这个模式让工程规范的执行变成了**基础设施级别的自动化**，不再依赖开发者的自律或代码审查。

### 跨平台差异

不同 Agent 平台的 Hook 配置方式各有不同：

| 平台 | 配置方式 | Hook 语言 | 安装 |
|---|---|---|---|
| Claude Code | `~/.claude/settings.json` 中的 `hooks` 块 | Shell 脚本或任意可执行文件 | 用户级或项目级配置 |
| Codex | `~/.codex/hooks/` 目录 | Shell 脚本 | 文件系统安装 |
| OpenCode | `opencode.config.ts` 中的 `hooks` | JavaScript/TypeScript | 配置文件声明 |
| Goose | `~/.config/goose/config.yaml` | Shell | YAML 配置 |
| Pi | 暂不支持自定义 Hook | — | — |

形式虽有差异，底层的生命周期模型正在趋同——这也是为什么课程强调"共享生命周期"的概念。

## 思考与疑问

1. Guardrails 模式在理念上非常强大，但它引入了一个信任转移的问题：**以前需要信任模型，现在需要信任钩子脚本本身**。如果钩子脚本有 bug 或安全漏洞怎么办？谁审计审计者？

2. 可观测性钩子记录每一次工具调用——这意味着在长时间会话中可能产生海量日志。**日志的存储、检索和隐私保护**是一个值得提前规划的问题，尤其是当 Agent 处理敏感数据时。

3. `PreToolUse` 虽然可以阻止危险操作，但如果 Agent 通过分派子代理来间接执行操作，父进程的钩子是否仍然有效？**钩子的作用域边界**——子代理是否继承父代理的钩子——在设计复杂的 Agent 系统时会影响安全模型。

4. 不同平台虽然共享相似的生命周期，但钩子的执行顺序和参数细节仍有差异。**跨平台迁移** Hook 策略时，需要仔细核对每个平台的事件语义是否完全对齐。

5. 自动化模式把工程规范变成了基础设施——这很好。但反过来思考：**过度自动化是否可能降低开发者的判断力？** 如果每次文件保存后都自动格式化和测试，开发者可能不再主动理解代码的问题，而是完全依赖钩子来"擦屁股"。
