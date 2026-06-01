---
title: 【实践8】三端对比：CLI vs Desktop vs Web 怎么选
publish: true
---

# 【实践8】三端对比：CLI vs Desktop vs Web 怎么选

> CLI的高灵活度、Desktop的可视化、Web的零安装——理解每个端的独特能力和适用场景，构建自己的多端工作流。

**参考文档**：
- `code.claude.com/docs/en/terminal-config.md` — Configure your terminal
- `code.claude.com/docs/en/desktop.md` — Claude Code Desktop
- `code.claude.com/docs/en/claude-code-on-the-web.md` — Claude Code on the web
- `code.claude.com/docs/en/vs-code.md` — VS Code
- `code.claude.com/docs/en/remote-control.md` — Remote Control

**先修知识**：无

**学习目标**：了解各端特性与差异、掌握选型矩阵、理解Remote Control、了解VS Code/JetBrains扩展

---

## 1. CLI Terminal（最大能力、最高灵活度）

### 概述

Claude Code CLI 是功能最完整、最灵活的接入方式。它可以在任何终端中运行——从系统原生终端（Terminal.app、Windows Terminal）到 IDE 内置终端（VS Code、Cursor、JetBrains），甚至在 tmux 会话中。

官方参考：[Configure your terminal for Claude Code](https://code.claude.com/docs/en/terminal-config.md)

### 关键特性

- **所有权限模式**：包括 `default`（Ask permissions）、`acceptEdits`、`plan`、`auto`、`dontAsk`、`bypassPermissions`
- **第三方模型提供商**：支持 Amazon Bedrock、Google Vertex AI、Microsoft Foundry
- **完整的脚本与自动化**：`--print` 模式、Agent SDK 集成
- **Tab 补全**：按 Tab 获得命令建议
- **`!` Bash 快捷命令**：在聊天中直接输入 `!` 执行 shell 命令
- **完整的命令和技能支持**：所有斜杠命令均可用
- **`--worktree`**：Git 工作树隔离，并行任务不互相干扰

### 终端配置要点

| 功能 | 配置方法 |
|------|---------|
| 多行输入 | `Ctrl+J` 或 `\`+Enter；Shift+Enter 因终端而异 |
| macOS Option 键 | 终端中启用 "Use Option as Meta Key" |
| 通知/响铃 | 设置 `preferredNotifChannel` 为 `"terminal_bell"`，或配置 Notification hook |
| tmux 适配 | 添加 `allow-passthrough on` 和 `extended-keys on` 到 `~/.tmux.conf` |
| 全屏渲染 | `/tui fullscreen` 或 `CLAUDE_CODE_NO_FLICKER=1` 环境变量 |
| Vim 编辑模式 | `/config` → Editor mode 设置为 `"vim"` |
| 自定义主题 | `/theme` 创建，保存在 `~/.claude/themes/` |

### 适用场景

- 需要全部功能权限控制的团队
- 使用 Bedrock/Vertex/Foundry 等第三方提供商的组织
- 自动化流水线、CI/CD 集成
- 偏好终端工作流的开发者
- **Linux 用户**（Desktop 不支持 Linux）

> **笔记区**：
>
>

---

## 2. Desktop App（可视化与图形工作流）

### 概述

Claude Code Desktop 是 macOS/Windows 上的原生桌面应用，提供图形化界面，运行与 CLI 相同的底层引擎。包含 Chat、Cowork、Code 三个标签页，其中 Code 标签页是主力界面。

官方参考：[Use Claude Code Desktop](https://code.claude.com/docs/en/desktop.md)

### 关键特性

#### 可视化开发环境
- **Diff 视图**：逐文件对比修改，支持行内评论（Cmd+Enter 提交）
- **多面板布局**：Chat、Diff、Preview、Terminal、File、Plan 等面板可任意拖拽排列
- **嵌入式浏览器预览**：运行 dev server 并自动验证，Claude 可自动截图、检查 DOM、操作 UI
- **任务面板**：查看 subagent、后台命令、工作流的实时输出

#### 并行会话
- 自动使用 **Git worktrees** 实现会话隔离
- 侧边栏管理多个会话，支持 Ctrl+Tab 切换
- Cmd+点击可在分屏中并排查看两个会话
- **侧边聊天**（Side Chat）：`Cmd+;` 开一个不影响主会话的临时对话

#### 环境选择
- **Local**：在本地机器运行
- **Remote**：在 Anthropic 云基础设施运行，关 app 也不中断
- **SSH**：连接远程服务器，自动在远端安装 Claude Code

#### 其他特性
- **Computer Use**（研究预览）：让 Claude 控制你的屏幕和 app（macOS/Windows，Pro/Max）
- **Connectors**：图形化添加 GitHub、Slack、Linear、Notion 等集成
- **Plugins**：图形化插件管理器，支持从市场安装
- **Dispatch**（Cowork 标签）：从手机发任务给 Desktop，自动创建 Code 会话
- **文件附件**：支持图片、PDF 拖拽到输入框
- **PR CI 监控**：PR 创建后自动监控 CI 状态，支持 Auto-fix 和 Auto-merge

### CLI vs Desktop 对比（来自官方文档）

| 特性 | CLI | Desktop |
|------|-----|---------|
| 权限模式 | 全部（含 `dontAsk`） | Ask、Auto accept、Plan、Auto、Bypass |
| 第三方提供商 | Bedrock, Vertex, Foundry | 仅 Anthropic API（企业可配 Vertex） |
| 会话隔离 | `--worktree` 标志 | 自动 Git worktrees |
| 多会话 | 分离的终端窗口 | 侧边栏标签页 |
| Computer Use | `/mcp` 仅 macOS | 原生支持 macOS/Windows |
| 文件附件 | 不支持 | 图片、PDF 拖拽 |
| Dispatch 集成 | 不支持 | 侧边栏集成 |
| 脚本/自动化 | `--print`, Agent SDK | 不支持 |
| Agent Teams | 支持 | 不支持 |
| Linux | 完全支持 | 不支持 |

### Desktop 独有快捷键

| 快捷键 | 操作 |
|--------|------|
| `Cmd+N` / `Ctrl+N` | 新建会话 |
| `Cmd+W` / `Ctrl+W` | 关闭会话 |
| `Ctrl+Tab` / `Ctrl+Shift+Tab` | 切换会话 |
| `Cmd+Shift+D` | 切换 diff 面板 |
| `Cmd+Shift+P` | 切换预览面板 |
| `Ctrl+` ` | 切换终端面板 |
| `Cmd+;` / `Ctrl+;` | 打开侧边聊天 |
| `Ctrl+O` | 切换视图模式（Normal/Verbose/Summary） |
| `Cmd+\` / `Ctrl+\` | 关闭面板 |

### Desktop 缺失功能
- 第三方提供商（仅 Anthropic API 默认）
- Linux 支持
- 行内代码补全
- Agent Teams（多 Agent 编排）
- `--print` 和 `--output-format` 输出模式

### CLI ↔ 桌面互转

在 CLI 中运行 `/desktop` 可将当前会话发送到 Desktop。两者共享同一个配置文件（`~/.claude/settings.json`、`CLAUDE.md`、MCP 配置）。

### 适用场景

- 偏好图形化界面和可视化 diff 的开发者
- 需要并排查看代码、终端和预览的多任务场景
- 使用 Computer Use 控制桌面应用和浏览器
- 需要集成 GitHub/Slack/Linear 等 Connectors

> **笔记区**：
>
>

---

## 3. Web claude.ai/code（零安装与云环境）

### 概述

Claude Code on the web（claude.ai/code）是研究预览功能，Pro/Max/Team/Enterprise 用户可用。任务在 Anthropic 托管的云 VM 上运行，关掉浏览器也不中断，可通过手机 Claude app 查看进度。

官方参考：[Use Claude Code on the web](https://code.claude.com/docs/en/claude-code-on-the-web.md)

### 关键特性

#### 零安装工作流
- 不需要在本地安装任何工具
- 通过 GitHub App 授权或 `/web-setup`（CLI 同步 `gh` token）连接仓库
- 每次会话从 GitHub 克隆仓库到全新 VM

#### 云环境（Cloud Environment）
- **预装工具**：Python 3.x + pip/poetry/uv/pytest、Node.js 20-22 + npm/yarn/pnpm/bun、Ruby 3.1-3.3、PHP 8.4 + Composer、OpenJDK 21 + Maven/Gradle、Go、Rust、GCC/Clang/cmake、Docker、PostgreSQL 16、Redis 7.0
- **资源限制**：4 vCPU、16 GB RAM、30 GB 磁盘
- **网络访问等级**：None（无网络）、Trusted（默认，允许主流包仓库和 GitHub）、Full（所有域名）、Custom（自定义白名单）
- **Setup Script**：开机运行的 Bash 脚本，输出被缓存到快照，约 7 天过期
- **环境缓存**：setup script 首次运行后快照文件系统，后续会话跳过安装步骤

#### 云会话携带的配置

| 配置项 | 是否传递到云 |
|--------|------------|
| 仓库的 `CLAUDE.md` | ✅ 是 |
| `.claude/settings.json` hooks | ✅ 是 |
| `.mcp.json` MCP 服务器 | ✅ 是 |
| `.claude/rules/` | ✅ 是 |
| `.claude/skills/`、`.claude/agents/` | ✅ 是 |
| Plugins（声明在 `.claude/settings.json`） | ✅ 是 |
| 用户级 `~/.claude/CLAUDE.md` | ❌ 否 |
| 本地 MCP 服务器（`claude mcp add`） | ❌ 否 |
| 交互式 SSO（AWS SSO 等） | ❌ 不支持 |
| API 令牌/凭据 | ❌ 暂无专用 secrets store |

#### 超强 Web 专属工作流
- **Ultraplan**：在云端生成执行计划，浏览器中审阅批注，可选择远程执行或发回终端
- **Ultrareview**：云端深度多 Agent 代码审查
- **Routines**：定时任务编排（按计划、API 调用、或 GitHub 事件触发）
- **Auto-fix PR**：自动监听 CI 失败和 review comments，推送修复
- **会话分享**：Enterprise/Team 可分享给团队成员（带仓库访问验证）

### CLI ↔ Web 会话互转

| 方向 | 命令 | 说明 |
|------|------|------|
| 终端 → Web | `claude --remote "task"` | 创建云端会话，克隆当前分支 |
| Web → 终端 | `claude --teleport` | 交互式选择器，拉取会话到本地 |
| 会话内拉取 | `/teleport` 或 `/tp` | 当前 CLI 会话内使用 |
| 任务列表查看 | `/tasks` | 查看所有后台云端会话 |
| 非 GitHub 仓库 | `CCR_FORCE_BUNDLE=1` | 打包本地仓库上传（<100MB） |

> `--remote` 是创建云会话；`--remote-control` 是暴露本地会话，两者不同。

### Web 限制
- 共享 rate limit（与所有 Claude/Claude Code 使用合计）
- 仅 GitHub 仓库原生支持 push；GitLab/Bitbucket 只能 bundle，不能 push
- 无用户级 CLAUDE.md、无交互式 SSO、无专用 secrets store
- 使用 IP 白名单的组织需联系 Anthropic 豁免
- 不支持 `--teleport` 到 API key / Bedrock / Vertex 认证的会话

### 适用场景
- 临时环境、没有本地开发工具的电脑
- 需要大规模并行任务（多个 `--remote` 并发）
- 团队协作场景：共享会话、定时 Routine、Auto-fix PR
- 从手机查看和指令云端任务

> **笔记区**：
>
>

---

## 4. VS Code 扩展与 JetBrains 插件

### VS Code 扩展

官方参考：[Use Claude Code in VS Code](https://code.claude.com/docs/en/vs-code.md)

#### 安装
- VS Code 1.98.0+，在 Extensions 搜索 "Claude Code"
- 通过 Spark 图标（编辑器右上角）、Activity Bar、Status Bar 快速启动
- Cursor/Windsurf/Kiro 等 VS Code fork 也可安装

#### 核心体验
- **@-mention 文件**：`@filename` 带入上下文，支持模糊匹配和行范围（如 `@app.ts#5-10`）
- **Plan Review**：Claude 在 Plan 模式下生成完整 markdown 文档，可直接行内评论编辑
- **Diff 视图**：原生 VS Code side-by-side diff，接受前可手动编辑
- **Checkpoints**：悬停在消息上可选择 Fork conversation、Rewind code、同时 fork + rewind
- **会话历史**：搜索/按时间浏览，重命名和删除
- **远程会话恢复**：从 claude.ai「拉取」远程会话到本地 VS Code
- **Chrome 集成**：`@browser` 在 VS Code 内控制 Chrome 浏览器

#### VS Code 扩展 vs CLI

| 特性 | CLI | VS Code 扩展 |
|------|-----|-------------|
| 命令和技能 | 全部 | 子集（`/` 查看可用） |
| MCP 配置 | 完整 | 部分（添加需 CLI，管理用 `/mcp`） |
| Checkpoints | 支持 | 支持（分支/回滚） |
| `!` Bash 快捷 | 支持 | 不支持 |
| Tab 补全 | 支持 | 不支持 |

#### 扩展设置一览

| 设置项 | 默认值 | 说明 |
|--------|--------|------|
| `useTerminal` | `false` | 是否使用终端模式而非图形界面 |
| `initialPermissionMode` | `default` | 新会话的默认权限模式 |
| `preferredLocation` | `panel` | Claude 面板位置 |
| `autosave` | `true` | 自动保存文件 |
| `useCtrlEnterToSend` | `false` | 使用 Ctrl+Enter 替代 Enter 发送 |
| `respectGitIgnore` | `true` | 排除 `.gitignore` 文件 |
| `enableNewConversationShortcut` | `false` | 启用 `Cmd+N` 新会话 |
| `allowDangerouslySkipPermissions` | `false` | 添加 Auto/Bypass 模式 |

#### VS Code 快捷键

| 快捷键 | 操作 |
|--------|------|
| `Cmd+Esc` / `Ctrl+Esc` | 切换编辑器和 Claude 焦点 |
| `Cmd+Shift+Esc` / `Ctrl+Shift+Esc` | 新标签页会话 |
| `Option+K` / `Alt+K` | 插入 @-mention 引用 |
| `Cmd+Shift+T` / `Ctrl+Shift+T` | 恢复最近关闭的会话 |

### URI Handler

```
vscode://anthropic.claude-code/open?prompt=review%20my%20changes
```

可从终端、书签或脚本启动 Claude Code 标签页，支持 `prompt` 和 `session` 参数。

### JetBrains 插件

JetBrains 用户目前主要通过 CLI + IDE 联动使用 Claude Code。插件生态系统支持情况以官方更新为准。

> **笔记区**：
>
>

---

## 5. Remote Control（远程控制）

### 概述

Remote Control 让你从手机、平板、或任何浏览器继续本地 Claude Code 会话。与云会话不同，**代码仍在本地执行**，不离开你的机器。

官方参考：[Continue local sessions from any device with Remote Control](https://code.claude.com/docs/en/remote-control.md)

### 工作原理

- 本地 Claude Code 进程仅发起出站 HTTPS 请求（不开放入站端口）
- 通过 Anthropic API 注册并轮询任务
- 多设备间会话实时同步（终端、浏览器、手机可交替输入）
- 笔记本休眠或网络断开后自动重连
- 短生命周期凭证，各自独立过期

### 启动方式

| 方法 | 命令 | 说明 |
|------|------|------|
| 服务端模式 | `claude remote-control` | 驻留终端等待连接，空格键显示 QR 码 |
| 交互模式 | `claude --remote-control` / `claude --rc` | 同时支持本地和远程输入 |
| 现有会话 | `/remote-control` / `/rc` | 携带当前对话历史 |
| VS Code | `/remote-control` | 输入框上方显示连接状态横幅 |
| 全部自动开启 | `/config` 或 Desktop Settings | 每个交互会话自动注册远程 |

### 服务端模式标志

| 标志 | 说明 |
|------|------|
| `--name "My Project"` | 自定义会话标题 |
| `--spawn same-dir`（默认） | 所有会话共享工作目录 |
| `--spawn worktree` | 每个按需会话使用独立 worktree |
| `--spawn session` | 单会话模式，拒绝额外连接 |
| `--capacity N` | 最大并发会话数（默认 32） |
| `--verbose` | 详细连接和会话日志 |
| `--sandbox` / `--no-sandbox` | 启用/禁用沙箱隔离 |

### Push 通知

- 长任务完成或需决策时自动推送
- 需 Claude Code v2.1.110+ 和 Claude 手机 app
- `/config` 中启用 "Push when Claude decides"
- 提示词中可指定：`notify me when the tests finish`

### Remote Control vs 云会话

| 维度 | Remote Control | Claude Code on the Web |
|------|---------------|----------------------|
| 执行位置 | 你的机器 | Anthropic 云 VM |
| 本地 MCP/工具 | 完整可用 | 需提交到仓库 |
| 关闭本地进程 | 会话结束 | 不受影响 |
| 手机控制 | 操控已有会话 | 监控和交互 |
| 最佳用途 | 在途中继续本地工作 | 从零开始远程任务 |

### 限 制

- 每个交互进程一个远程会话（服务端模式可多个）
- 本地进程必须保持运行
- 网络断开约 10 分钟超时退出
- Ultraplan 会断开 Remote Control
- 部分命令本地专用：`/mcp`、`/plugin`、`/resume`

### 不同远程接入方式对比

| | 触发方式 | 执行环境 | 最适合 |
|--|---------|---------|--------|
| **Dispatch** | 从手机 app 发消息 | Desktop（你的机器） | 不用管设置，直接发任务的随心模式 |
| **Remote Control** | 从浏览器/手机操控已有会话 | CLI/VS Code（你的机器） | 在途中继续进行中的工作 |
| **Channels** | Telegram/Discord 等消息事件 | CLI（你的机器） | CI 失败等外部事件响应 |
| **Slack** | 团队频道 `@Claude` | Anthropic 云 | 团队聊天中的 PR 和审查 |
| **Scheduled tasks** | 按计划 | CLI/Desktop/云 | 每日自动化 |

> **笔记区**：
>
>

---

## 6. 场景选型矩阵

| 场景 | 推荐端 | 原因 |
|------|--------|------|
| **日常个人开发** | CLI + VS Code 扩展 | 终端灵活 + IDE 的 diff/@-mention 和 Checkpoints |
| **代码审查 / PR 管理** | Desktop 或 Web | 可视化 diff、行内评论、CI 状态监控、Auto-fix |
| **移动办公（外出途中）** | Remote Control + Claude 手机 app | 在手机上继续本地会话，收 push 通知 |
| **团队协作** | Web + Routines | 会话分享、定时任务、Auto-fix PR、团队可见 |
| **Linux 开发者** | CLI | Desktop 不支持 Linux，CLI 最完善 |
| **CI/CD 和自动化** | CLI（`--print` + Agent SDK） | 标准 I/O 模式，可集成到任何管道 |
| **原型快速搭建** | Desktop 或 Web | 零配置启动，嵌入式预览立刻看到效果 |
| **使用第三方模型（Bedrock/Vertex）** | CLI | CLI 是唯一支持第三方提供商的全功能端 |
| **需要 Computer Use** | Desktop（macOS/Windows） | 原生支持 app/屏幕控制 |
| **教学 / Triage** | Web（零安装） | 浏览器打开即用，无需本地配置 |
| **需要 Connectors** | Desktop 或 CLI | 都支持 Connectors/MCP |
| **多任务并行开发** | Desktop 或 多个 `--remote` | Desktop 内建 worktree + 分片；Web 可批量创建云会话 |
| **离线 / 受限网络** | CLI | 无云依赖，可在断网或内网环境使用 |

> **笔记区**：
>
>

---

## 7. 各端功能差异速查表

| 功能维度 | CLI Terminal | Desktop App | Web (claude.ai/code) | VS Code 扩展 |
|---------|:-----------:|:-----------:|:-------------------:|:-----------:|
| **平台支持** | | | | |
| macOS | ✅ | ✅ | ✅（浏览器） | ✅ |
| Windows | ✅ | ✅ | ✅（浏览器） | ✅ |
| Linux | ✅ | ❌ | ✅（浏览器） | ✅ |
| iOS/Android | ❌ | ❌ | ✅（app 监控） | ❌ |
| **运行环境** | | | | |
| 本地运行 | ✅ | ✅ | ❌ | ✅ |
| Anthropic 云环境 | `--remote` | Remote 选项 | ✅ 原生 | 可恢复云会话 |
| SSH 远程 | ✅ | ✅ | ❌ | ❌ |
| **代码审查** | | | | |
| Diff 可视审查 | 文本 diff | ✅ 原生高亮 | ✅ 原生高亮 | ✅ VS Code diff |
| 行内评论 | ❌ | ✅ | ✅ | Plan markdown 评论 |
| 嵌入式预览 | ❌ | ✅ 内置浏览器 | ❌ | ❌ |
| 自动验证 | ❌ | ✅ | ❌ | ❌ |
| **上下文引用** | | | | |
| @-mention 文件 | 文本方式 | ✅ 自动补全 | ❌ | ✅ 模糊+行范围 |
| 文件附件（图片/PDF） | ❌ | ✅ | ❌ | ✅ |
| Checkpoints | ✅ | ❌ | ❌ | ✅ 分支回滚 |
| **权限模式** | | | | |
| Ask permissions | ✅ | ✅ | ✅ | ✅ |
| Plan mode | ✅ | ✅ | ✅ | ✅ |
| Auto accept edits | ✅ | ✅ | ✅ | ✅ |
| Auto mode | ✅ | ✅ | ❌ | ✅（需设置） |
| Bypass permissions | ✅ | ✅（设置中） | ❌ | ✅（需设置） |
| `dontAsk` | ✅ | ❌ | ❌ | ❌ |
| **会话管理** | | | | |
| 并行会话 | 多终端 | ✅ 侧边栏 | ✅ 侧边栏 | ✅ 标签/窗口 |
| Git worktree 隔离 | `--worktree` | ✅ 自动 | ✅ 全新 VM | ✅（CLI 模式） |
| 会话历史 | ✅ | ✅ | ✅ | ✅ |
| 会话分享 | ❌ | ❌ | ✅（Team/Public） | ❌ |
| 分屏并排会话 | ❌ | ✅ Cmd+点击 | ❌ | ❌ |
| 侧边聊天 | ❌ | ✅ `Cmd+;` | ❌ | ❌ |
| **扩展性** | | | | |
| 第三方模型 | Bedrock/Vertex/Foundry | 仅 Anthropic | 仅 Anthropic | Bedrock/Vertex/Foundry |
| MCP 服务器 | ✅ 配置文件 | ✅ Connectors UI | ✅ `.mcp.json` | ✅ CLI + 面板管理 |
| Plugins | `/plugin` 命令 | ✅ 图形化 | ✅ 提交到仓库 | ✅ 图形化 |
| Connectors | ✅ | ✅ UI | Routines 配置 | ✅ |
| Computer Use | ✅ `/mcp`（仅 macOS） | ✅ macOS/Windows | ❌ | ❌ |
| Chrome 集成 | ❌ | ❌ | ❌ | ✅ `@browser` |
| **自动化** | | | | |
| `--print` 模式 | ✅ | ❌ | ❌ | ❌ |
| Agent SDK | ✅ | ❌ | ❌ | ❌ |
| Routines | ✅ | ✅ | ✅ | ❌ |
| Auto-fix PR | ❌ | ❌ | ✅ | ❌ |
| Dispatch | ❌ | ✅ | ❌ | ❌ |
| Scheduled tasks | ✅ | ✅ | ✅ | ❌ |
| **远程控制** | | | | |
| Remote Control | ✅ `--rc` | ✅ 默认开关 | ✅ 浏览器接入 | ✅ `/rc` |
| Push 通知 | ✅ | ✅（OS 通知） | ❌ | ❌ |
| 手机 app | ✅ 推+Remote Control | ✅ Dispatch | ✅ 监控 | ❌ |
| **终端体验** | | | | |
| Tab 补全 | ✅ | ❌ | ❌ | ❌ |
| Vim 编辑模式 | ✅ | ❌ | ❌ | ❌ |
| `!` Bash 快捷 | ✅ | ❌ | ❌ | ❌ |
| 自定义主题 | ✅ | ❌ | ❌ | ❌ |
| 全屏渲染 | ✅ | ❌ | ❌ | ❌ |
| **CLI 标志支持** | | | | |
| `--model` | ✅ | ✅ 下拉 | ✅ 下拉 | ✅ 面板选 |
| `--resume` / `--continue` | ✅ | ✅ 侧边栏 | ✅ 侧边栏 | ✅ 会话历史 |
| `--verbose` | ✅ | ✅ 视图模式 | ❌ | ❌ |
| `--dangerously-skip-permissions` | ✅ | ✅ 设置开启 | ❌ | ✅ 设置开启 |
| `--print` / `--output-format` | ✅ | ❌ | ❌ | ❌ |
| **其他核心差异** | | | | |
| Linux 原生支持 | ✅ | ❌ | ✅（浏览器） | ✅ |
| Agent Teams | ✅ | ❌ | ✅（env 开关） | ❌ |
| 自动化 | ✅ | ❌ | ❌ | ❌ |

> **笔记区**：
>
>

---

## 总结

- **CLI** 是能力最全的「瑞士军刀」—— 所有权限模式、所有提供商、所有自动化接口。适合深度开发和 CI/CD。
- **Desktop** 是「可视化作战指挥中心」—— 多面板布局、diff 审查、预览、Dispatch、Computer Use。适合日常开发和多任务管理。
- **Web** 是「零部署移动指挥部」—— 云端环境、Ultraplan、Auto-fix PR、Routines。适合临时环境、团队协作和自动化流程。
- **VS Code 扩展**是「IDE 原生助手」—— @-mention、Plan review、Checkpoints、Chrome 集成。适合在编辑器内无缝使用。
- **Remote Control** 是「跨设备桥梁」—— 把以上所有端的能力串联起来，在手机/平板上操控本地会话。

**选择策略**：日常用 CLI + Desktop 桌面环境，项目协作用 Web 云会话，需要移动时用 Remote Control。多个端可以同时打开使用，共享同一套配置文件、项目记忆（`CLAUDE.md`）和 MCP 服务器。
