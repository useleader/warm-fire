---
title: 【实践9】Plugins 与 Marketplaces：插件生态与分发
publish: true
---

# 【实践9】Plugins 与 Marketplaces：插件生态与分发

> Plugin = Skills + Hooks + MCP + Agents 的打包单元。理解 Marketplace 机制、学会创建和分发插件。

**参考文档**：
- `code.claude.com/docs/en/plugins.md` — Create plugins
- `code.claude.com/docs/en/discover-plugins.md` — Discover and install plugins
- `code.claude.com/docs/en/plugin-marketplaces.md` — Create and distribute marketplaces
- `code.claude.com/docs/en/plugins-reference.md` — Plugins reference

**先修知识**：[[【实践6】MCP深度实践：连接外部工具与数据源]]

**学习目标**：理解 Plugin 结构、学会发现安装、了解官方 Marketplace、学会创建 Plugin、了解团队分发

---

## 1. Plugin 是什么（Skills + Hooks + MCP Servers + Agents + Monitors + Themes 的组合打包）

Plugin 是 Claude Code 的扩展打包单元，将多个可复用组件封装为一个可分发的整体。

**核心组件一览：**

| 组件 | 位置 | 说明 |
|------|------|------|
| Skills | `skills/` | 技能目录，每个子目录包含 `SKILL.md`。用户通过 `/plugin-name:skill-name` 调用，Claude 也可自动触发 |
| Commands | `commands/` | 扁平 `.md` 技能文件（旧式），新项目建议用 `skills/` |
| Agents | `agents/` | 子代理定义文件（Markdown），支持 `name`、`description`、`model`、`effort`、`maxTurns`、`tools`、`disallowedTools`、`isolation` 等 frontmatter 字段 |
| Hooks | `hooks/hooks.json` | 事件钩子配置，可响应 `PostToolUse`、`SessionStart`、`UserPromptSubmit` 等 20+ 生命周期事件 |
| MCP Servers | `.mcp.json` | MCP 服务器定义，将外部工具集成到 Claude 的工具箱 |
| LSP Servers | `.lsp.json` | LSP 语言服务器配置，提供即时诊断、代码导航（跳转到定义、查找引用） |
| Monitors | `monitors/monitors.json` | 后台监控器，持续运行 shell 命令并将 stdout 行作为通知发送给 Claude |
| Themes | `themes/` | 颜色主题定义（实验性功能），出现在 `/theme` 列表中 |
| Executables | `bin/` | 可执行文件目录，插件启用时添加到 Bash 工具的 `PATH` |
| Settings | `settings.json` | 默认设置，插件启用时应用。目前仅支持 `agent` 和 `subagentStatusLine` 键 |
| Manifest | `.claude-plugin/plugin.json` | 清单文件（可选），定义插件元数据和自定义组件路径 |

**Plugin vs 独立配置（Standalone）：**

| 维度 | 独立配置 `.claude/` | Plugin |
|------|---------------------|--------|
| 技能命名 | `/hello`（短名称） | `/plugin-name:hello`（带命名空间） |
| 适用场景 | 个人工作流、单项目、快速实验 | 团队共享、社区分发、跨项目复用 |
| 版本管理 | 无 | 支持版本追踪和自动更新 |
| 分发方式 | 手动复制 | 通过 Marketplace 分发与安装 |

> 开发建议：先在 `.claude/` 中快速迭代，成熟后再迁移为 Plugin。

---

## 2. 发现与安装（Marketplace 机制、添加来源、CLI 管理命令）

### Marketplace 工作机制

Marketplace 是一个插件目录。使用分两步：
1. **添加 Marketplace** — 注册目录源，让 Claude Code 知道有哪些插件可用（此时并未安装任何插件）
2. **安装插件** — 从目录中浏览并选择要安装的插件

> 类比：添加应用商店让你能浏览，但你仍需要逐个选择下载的应用。

### 添加 Marketplace 的命令

```shell
# 从 GitHub 添加
/plugin marketplace add owner/repo
/plugin marketplace add anthropics/claude-code

# 从 Git URL 添加（GitLab / Bitbucket / 自托管）
/plugin marketplace add https://gitlab.com/company/plugins.git
/plugin marketplace add git@gitlab.com:company/plugins.git

# 指定分支或标签
/plugin marketplace add https://gitlab.com/company/plugins.git#v1.0.0

# 从本地路径添加
/plugin marketplace add ./my-marketplace
/plugin marketplace add ./path/to/marketplace.json

# 从远程 URL 添加
/plugin marketplace add https://example.com/marketplace.json
```

### CLI 管理命令

```bash
# 列出所有已配置的 Marketplace
claude plugin marketplace list

# 安装插件（默认 user 作用域）
claude plugin install plugin-name@marketplace-name

# 指定安装作用域
claude plugin install plugin-name@marketplace-name --scope project

# 卸载插件
claude plugin uninstall plugin-name@marketplace-name

# 启用/禁用
claude plugin enable plugin-name@marketplace-name
claude plugin disable plugin-name@marketplace-name

# 更新插件
claude plugin update plugin-name@marketplace-name

# 更新 Marketplace（获取最新插件列表）
/plugin marketplace update marketplace-name

# 移除 Marketplace（会同时卸载从它安装的所有插件）
/plugin marketplace remove marketplace-name

# 验证插件/目录结构
claude plugin validate .

# 重载所有插件（安装/启用/禁用后无需重启）
/reload-plugins
```

### 安装作用域（Scopes）

| 作用域 | 配置文件 | 用途 |
|--------|----------|------|
| `user` | `~/.claude/settings.json` | 个人插件，跨项目可用（默认） |
| `project` | `.claude/settings.json` | 团队插件，通过版本控制共享 |
| `local` | `.claude/settings.local.json` | 项目特定插件，被 gitignore |
| `managed` | Managed settings | 管理员托管插件，只读 |

### 自动更新配置

- 官方 Marketplace 默认启用自动更新
- 第三方和本地开发 Marketplace 默认禁用
- 管理员可在 `extraKnownMarketplaces` 中设置 `"autoUpdate": true`
- 通过 `DISABLE_AUTOUPDATER=1` 禁用所有自动更新
- `FORCE_AUTOUPDATE_PLUGINS=1` 可在禁用 Claude Code 更新的同时保持插件更新

---

## 3. 官方 Marketplace 精选

官方 Marketplace（`claude-plugins-official`）在 Claude Code 启动时自动可用。运行 `/plugin` 进入 Discover 标签浏览，或访问 `claude.com/plugins`。

### Code Intelligence 类（LSP 插件）

这些插件为 Claude 提供语言服务器能力，包括即时诊断和代码导航（跳转到定义、查找引用、悬停类型信息等）。

| 语言 | 插件名 | 需安装的二进制 |
|------|--------|---------------|
| C/C++ | `clangd-lsp` | `clangd` |
| C# | `csharp-lsp` | `csharp-ls` |
| Go | `gopls-lsp` | `gopls` |
| Java | `jdtls-lsp` | `jdtls` |
| Kotlin | `kotlin-lsp` | `kotlin-language-server` |
| Lua | `lua-lsp` | `lua-language-server` |
| PHP | `php-lsp` | `intelephense` |
| Python | `pyright-lsp` | `pyright-langserver` |
| Rust | `rust-analyzer-lsp` | `rust-analyzer` |
| Swift | `swift-lsp` | `sourcekit-lsp` |
| TypeScript | `typescript-lsp` | `typescript-language-server` |

安装 LSP 插件后，Claude 每次编辑文件后自动获得诊断结果，能即时看到类型错误、缺失导入和语法问题，无需手动运行编译器。同时支持通过代码导航查找定义、引用和类型信息。

### 外部集成类

- **源码控制**：`github`、`gitlab`
- **项目管理**：`atlassian`（Jira/Confluence）、`asana`、`linear`、`notion`
- **设计**：`figma`
- **基础设施**：`vercel`、`firebase`、`supabase`
- **通信**：`slack`
- **监控**：`sentry`

### 开发工作流类

- `commit-commands`：Git 提交流程（commit、push、PR 创建）
- `pr-review-toolkit`：PR Review 专用代理
- `agent-sdk-dev`：Claude Agent SDK 构建工具
- `plugin-dev`：创建自己的 Plugin 的工具包

### 输出风格类

- `explanatory-output-style`：实现选择的深入教育见解
- `learning-output-style`：互动学习模式

> 提交 Plugin 到官方 Marketplace：`claude.ai/settings/plugins/submit` 或 `platform.claude.com/plugins/submit`

---

## 4. 创建你的第一个 Plugin

### 目录结构

```
my-first-plugin/
├── .claude-plugin/
│   └── plugin.json          # 插件清单
└── skills/
    └── hello/
        └── SKILL.md          # 技能定义
```

### 创建清单文件 `.claude-plugin/plugin.json`

```json
{
  "name": "my-first-plugin",
  "description": "A greeting plugin to learn the basics",
  "version": "1.0.0",
  "author": {
    "name": "Your Name"
  }
}
```

| 字段 | 说明 |
|------|------|
| `name` | 唯一标识符（kebab-case），也是技能命名空间。技能自动获得 `/my-first-plugin:hello` 形式的前缀 |
| `description` | 插件管理器中的描述文本 |
| `version` | 语义版本。设置后用户仅在你更新此字段时收到更新；省略则使用 git commit SHA |
| `author` | 作者归属信息 |

### 创建技能文件 `skills/hello/SKILL.md`

```markdown
---
description: Greet the user with a friendly message
disable-model-invocation: true
---

Greet the user warmly and ask how you can help them today.
```

### 测试插件

```bash
claude --plugin-dir ./my-first-plugin
```

在 Claude Code 中运行：

```shell
/my-first-plugin:hello
```

添加参数支持：

```markdown
---
description: Greet the user with a personalized message
---

Greet the user named "$ARGUMENTS" warmly and ask how you can help them today.
```

```shell
/my-first-plugin:hello Alex
```

> 修改后运行 `/reload-plugins` 即可生效，无需重启。

### 从独立配置迁移到 Plugin

```bash
mkdir -p my-plugin/.claude-plugin
cp -r .claude/commands my-plugin/
cp -r .claude/agents my-plugin/
cp -r .claude/skills my-plugin/
mkdir my-plugin/hooks
```

| 独立配置（`.claude/`） | Plugin |
|------------------------|--------|
| 仅在一个项目中可用 | 可通过 Marketplace 共享 |
| 文件在 `.claude/commands/` | 文件在 `plugin-name/commands/` |
| Hooks 在 `settings.json` | Hooks 在 `hooks/hooks.json` |
| 需要手动复制共享 | 通过 `/plugin install` 安装 |

---

## 5. Plugin 组件详解

### Skills

- 位置：`skills/` 目录（每个技能一个子目录 + `SKILL.md`）
- 名称自动带有插件命名空间前缀，如 `/my-plugin:code-review`
- 可包含辅助文件（`reference.md`、`scripts/` 等）
- YAML frontmatter 定义 `description`、`disable-model-invocation` 等
- 被 Claude 根据上下文自动发现和调用

### Agents

- 位置：`agents/` 目录
- 文件格式：带 YAML frontmatter 的 Markdown 文件

```markdown
---
name: security-reviewer
description: Reviews code for security vulnerabilities
model: sonnet
effort: medium
maxTurns: 20
disallowedTools: Write, Edit
---

You are a security review agent. Analyze code for:
- SQL injection, XSS, CSRF
- Authentication and authorization flaws
- Insecure dependencies
- Sensitive data exposure
```

- 支持的 frontmatter 字段：`name`、`description`、`model`、`effort`、`maxTurns`、`tools`、`disallowedTools`、`skills`、`memory`、`background`、`isolation`
- **安全限制**：Plugin 代理不支持 `hooks`、`mcpServers`、`permissionMode`
- 出现在 `/agents` 列表中，可手动或由 Claude 自动调用

### Hooks

- 位置：`hooks/hooks.json`（或 `plugin.json` 中内联）
- 支持的钩子事件（20+）：`SessionStart`、`Setup`、`UserPromptSubmit`、`UserPromptExpansion`、`PreToolUse`、`PermissionRequest`、`PermissionDenied`、`PostToolUse`、`PostToolUseFailure`、`PostToolBatch`、`Notification`、`SubagentStart`、`SubagentStop`、`Stop`、`StopFailure`、`FileChanged`、`WorktreeCreate`、`WorktreeRemove`、`PreCompact`、`PostCompact`、`SessionEnd` 等
- 钩子类型：`command`（shell 命令）、`http`（POST 请求）、`mcp_tool`（MCP 工具调用）、`prompt`（LLM 评估）、`agent`（代理验证器）

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "\"${CLAUDE_PLUGIN_ROOT}\"/scripts/format-code.sh"
          }
        ]
      }
    ]
  }
}
```

### MCP Servers

- 位置：`.mcp.json`（或在 `plugin.json` 中内联）
- 插件启用时自动启动
- 使用 `${CLAUDE_PLUGIN_ROOT}` 引用插件内脚本路径
- 使用 `${CLAUDE_PLUGIN_DATA}` 引用持久化数据目录（跨版本保留）

```json
{
  "mcpServers": {
    "plugin-database": {
      "command": "${CLAUDE_PLUGIN_ROOT}/servers/db-server",
      "args": ["--config", "${CLAUDE_PLUGIN_ROOT}/config.json"],
      "env": {
        "DB_PATH": "${CLAUDE_PLUGIN_DATA}"
      }
    }
  }
}
```

### LSP Servers

- 位置：`.lsp.json`（或在 `plugin.json` 中内联）
- 通用语言优先使用官方 Marketplace 的现成 LSP 插件
- 仅对官方未覆盖的语言需要自定义

```json
{
  "go": {
    "command": "gopls",
    "args": ["serve"],
    "extensionToLanguage": {
      ".go": "go"
    }
  }
}
```

配置字段：`command`（必须）、`extensionToLanguage`（必须）、`args`、`transport`（stdio/socket）、`env`、`initializationOptions`、`settings`、`workspaceFolder`、`startupTimeout`、`shutdownTimeout`、`restartOnCrash`、`maxRestarts`

### Monitors

- 位置：`monitors/monitors.json`（实验性功能）
- 插件激活时自动启动后台监控进程
- 每行 stdout 以通知方式发送给 Claude
- 运行于交互式 CLI 会话，跳过不可用主机
- 支持 `when` 字段：`"always"`（默认）或 `"on-skill-invoke:<skill-name>"`

```json
[
  {
    "name": "deploy-status",
    "command": "\"${CLAUDE_PLUGIN_ROOT}\"/scripts/poll-deploy.sh",
    "description": "Deployment status changes"
  },
  {
    "name": "error-log",
    "command": "tail -F ./logs/error.log",
    "description": "Application error log",
    "when": "on-skill-invoke:debug"
  }
]
```

### Themes

- 位置：`themes/` 目录（实验性功能）
- JSON 格式，定义 `base`（dark/light）和 `overrides`

### ${CLAUDE_PLUGIN_ROOT} 与 ${CLAUDE_PLUGIN_DATA}

| 变量 | 说明 | 用途 |
|------|------|------|
| `${CLAUDE_PLUGIN_ROOT}` | 插件安装目录的绝对路径 | 引用插件内的脚本、二进制、配置文件。更新时会改变 |
| `${CLAUDE_PLUGIN_DATA}` | 持久化数据目录（`~/.claude/plugins/data/{id}/`） | 存储 `node_modules`、缓存、生成代码，跨版本保留 |
| `${CLAUDE_PROJECT_DIR}` | 项目根目录 | 引用项目本地脚本或配置文件 |

> 常见错误：所有组件目录（`skills/`、`agents/`、`hooks/` 等）必须在 plugin 根目录，**不能**在 `.claude-plugin/` 内部。只有 `plugin.json` 属于 `.claude-plugin/`。

---

## 6. 搭建团队 Marketplace

### Marketplace 文件结构

```
my-marketplace/
├── .claude-plugin/
│   └── marketplace.json
└── plugins/
    └── quality-review-plugin/
        ├── .claude-plugin/
        │   └── plugin.json
        └── skills/
            └── quality-review/
                └── SKILL.md
```

### Marketplace 清单 `marketplace.json`

```json
{
  "name": "company-tools",
  "owner": {
    "name": "DevTools Team",
    "email": "devtools@example.com"
  },
  "plugins": [
    {
      "name": "code-formatter",
      "source": "./plugins/formatter",
      "description": "Automatic code formatting on save",
      "version": "2.1.0"
    },
    {
      "name": "deployment-tools",
      "source": {
        "source": "github",
        "repo": "company/deploy-plugin"
      },
      "description": "Deployment automation tools"
    }
  ]
}
```

### Plugin 源类型

| 源类型 | 格式 | 说明 |
|--------|------|------|
| 相对路径 | `"./plugins/my-plugin"` | 同仓库内，必须以 `./` 开头 |
| GitHub | `{"source": "github", "repo": "owner/repo"}` | 支持 `ref`（分支/标签）和 `sha`（精确 commit）锁定 |
| Git URL | `{"source": "url", "url": "https://..."}` | 支持 `ref` 和 `sha` 锁定 |
| Git 子目录 | `{"source": "git-subdir", "url": "...", "path": "..."}` | 稀疏克隆，节省带宽 |
| npm | `{"source": "npm", "package": "@org/plugin"}` | 支持版本范围和私有 registry |

### 发布与分发

**托管方案推荐**：

1. **GitHub（推荐）** — `/plugin marketplace add owner/repo`
2. **GitLab / Bitbucket / 自托管** — `/plugin marketplace add https://gitlab.com/company/plugins.git`
3. **私有仓库** — 需配置 token 环境变量

**团队自动配置**：在项目 `.claude/settings.json` 中配置：

```json
{
  "extraKnownMarketplaces": {
    "company-tools": {
      "source": {
        "source": "github",
        "repo": "your-org/claude-plugins"
      }
    }
  },
  "enabledPlugins": {
    "code-formatter@company-tools": true,
    "deployment-tools@company-tools": true
  }
}
```

当团队成员信任项目文件夹时，Claude Code 会自动提示安装这些 Marketplace 和插件。

### Release Channels（发布频道）

通过设置两个不同的 Marketplace 指向同一仓库的不同 ref 来实现 stable 和 latest 频道：

```json
// stable-tools marketplace → ref: "stable"
// latest-tools marketplace → ref: "latest"
```

通过 managed settings 将不同频道分配给不同用户组。

### 容器预填充（Pre-populate）

```bash
# 镜像构建时安装插件
CLAUDE_CODE_PLUGIN_CACHE_DIR=/opt/claude-seed claude plugin marketplace add your-org/plugins
CLAUDE_CODE_PLUGIN_CACHE_DIR=/opt/claude-seed claude plugin install my-tool@your-plugins

# 运行时设置种子目录
export CLAUDE_CODE_PLUGIN_SEED_DIR=/opt/claude-seed
```

种子目录只读，自动更新被禁用，由管理员通过更新镜像来更新插件。

---

## 7. 插件依赖版本约束

### 声明依赖

在 `plugin.json` 中通过 `dependencies` 字段声明：

```json
{
  "name": "my-plugin",
  "dependencies": [
    "helper-lib",
    { "name": "secrets-vault", "version": "~2.1.0" }
  ]
}
```

- 字符串格式：接受任何版本
- 对象格式：指定名称和 semver 版本范围

### 版本解析规则

版本按照以下优先级确定（从高到低）：

1. `plugin.json` 中的 `version` 字段
2. Marketplace 条目中的 `version` 字段
3. Git commit SHA（用于 `github`、`url`、`git-subdir` 及 git 托管的相对路径源）
4. `unknown`（npm 源或不属于 git 仓库的本地目录）

| 版本策略 | 更新行为 | 适用场景 |
|----------|----------|----------|
| 显式版本 | 仅当手动 bump 时用户收到更新 | 有稳定发布周期的 Plugin |
| Commit SHA | 每次新 commit 都是新版本 | 内部/团队活跃开发 Plugin |

**避免**同时在 `plugin.json` 和 `marketplace.json` 中设置 `version`——`plugin.json` 始终优先，可能掩盖 marketplace 条目中的版本变更。

### 跨 Marketplace 依赖

需要在 Marketplace 清单中设置 `allowCrossMarketplaceDependenciesOn` 数组，声明允许依赖的其他 Marketplace。未被列表包含的 Marketplace 依赖在安装时被阻止。

### 依赖清理

```bash
claude plugin prune                    # 清理不再需要的自动安装依赖
claude plugin uninstall --prune        # 卸载时同时清理依赖
claude plugin prune --dry-run          # 预览但不执行
claude plugin uninstall --keep-data    # 保留数据目录
```

---

## 8. 安全考量

### 私有仓库认证

| 平台 | 环境变量 | 说明 |
|------|----------|------|
| GitHub | `GITHUB_TOKEN` 或 `GH_TOKEN` | Personal access token 或 GitHub App token |
| GitLab | `GITLAB_TOKEN` 或 `GL_TOKEN` | Personal access token 或 project token |
| Bitbucket | `BITBUCKET_TOKEN` | App password 或 repository access token |

手动安装时使用 git credential helpers（`gh auth login`、SSH key、macOS Keychain 等）。后台自动更新需要设置环境变量 token，因为交互式提示会阻塞 Claude Code 启动。

### Managed Marketplace 限制

管理员通过 `strictKnownMarketplaces` 限制允许的 Marketplace 源，设置在 managed settings 中：

```json
// 完全锁定：用户不能添加任何新 Marketplace
{ "strictKnownMarketplaces": [] }

// 仅允许特定 Marketplace
{
  "strictKnownMarketplaces": [
    { "source": "github", "repo": "acme-corp/approved-plugins" },
    { "source": "url", "url": "https://plugins.example.com/marketplace.json" }
  ]
}

// 通过正则匹配内网 Git 服务器
{
  "strictKnownMarketplaces": [
    { "source": "hostPattern", "hostPattern": "^git\\.example\\.com$" }
  ]
}

// 允许特定文件系统路径
{
  "strictKnownMarketplaces": [
    { "source": "pathPattern", "pathPattern": "^/opt/approved/" }
  ]
}
```

限制在插件添加、安装、更新和自动更新时都会检查。精确匹配不规范化 URL，因此推荐使用 `hostPattern` 替代字面 URL。

### 插件安全要点

- Plugin 是高度信任的组件，可在用户权限下执行任意代码
- 仅在安装前确认来源可信
- Anthropic 不控制第三方插件包含的 MCP 服务器、文件或其他软件
- `managed` 作用域的插件为只读，用户无法修改
- Plugin 缓存使用复制而非原地引用，防止路径穿越
- Symlink 安全策略：
  - 指向插件自身目录的 symlink → 保留
  - 指向 Marketplace 内其他位置的 symlink → 解引用复制
  - 指向外部的 symlink → 跳过（安全原因）
- `sensitive: true` 的 `userConfig` 值存储在系统密钥链中，而非 `settings.json`

### 安全检查清单

- [ ] `marketplace.json` 的 `name` 是否使用了保留名称？
- [ ] 私有仓库的 token 是否已配置环境变量？
- [ ] 是否设置了 `strictKnownMarketplaces` 限制用户？
- [ ] Plugin 是否使用了 `${CLAUDE_PLUGIN_ROOT}` 而非硬编码路径？
- [ ] 敏感配置是否使用了 `userConfig` 的 `sensitive: true`？

---

## 调试与常见问题

### CLI 调试命令

```bash
# 查看插件加载详情
claude --debug

# 验证插件/目录结构
claude plugin validate .

# 查看已安装插件列表
claude plugin list --json --available

# 查看插件详情（组件清单、预估 token 消耗）
claude plugin details plugin-name@marketplace-name
```

### 常见问题速查

| 问题 | 原因 | 解决 |
|------|------|------|
| 插件未加载 | `plugin.json` 无效 | 运行 `claude plugin validate .` |
| 技能不出现 | 目录结构错误 | 确保 `skills/` 在 plugin 根目录 |
| Hooks 不触发 | 脚本不可执行 | `chmod +x script.sh` |
| MCP 服务器失败 | 缺少变量引用 | 使用 `${CLAUDE_PLUGIN_ROOT}` |
| 路径错误 | 使用了绝对路径 | 所有路径以 `./` 开头 |
| LSP "Executable not found" | 未安装二进制 | 安装对应的语言服务器 |
| Marketplace 不加载 | 缺少 `marketplace.json` | 确认文件路径和访问权限 |
| 离线失败 | git pull 失败后缓存被清空 | `CLAUDE_CODE_PLUGIN_KEEP_MARKETPLACE_ON_FAILURE=1` |
| Git 超时 | 仓库过大或网络慢 | `CLAUDE_CODE_PLUGIN_GIT_TIMEOUT_MS=300000` |
| 相对路径在 URL Marketplace 中失败 | URL Marketplace 只下载 `marketplace.json` | 改用 Git 源 |

---

## 思维导图

```
Plugin 生态
├── 核心概念
│   ├── Plugin = Skills + Hooks + MCP + Agents + Monitors + Themes
│   ├── 命名空间机制：/plugin-name:skill-name
│   └── 版本管理：显式 version / git commit SHA
├── 组件详解
│   ├── Skills → 技能定义（SKILL.md）
│   ├── Agents → 子代理（Markdown frontmatter）
│   ├── Hooks → 生命周期钩子（20+ 事件）
│   ├── MCP → 外部工具集成
│   ├── LSP → 代码智能（即时诊断 + 导航）
│   └── Monitors → 后台监控
├── Marketplace
│   ├── 官方（claude-plugins-official）
│   ├── 社区/团队（GitHub / GitLab / npm）
│   └── 安全管理（strictKnownMarketplaces）
├── 安装管理
│   ├── 作用域：user / project / local / managed
│   ├── CLI 命令：install / uninstall / enable / disable
│   └── 自动更新策略
├── 创建插件
│   ├── 清单 plugin.json
│   ├── 技能（$ARGUMENTS 参数）
│   └── 测试：--plugin-dir / --plugin-url
└── 安全与分发
    ├── 私有仓库认证（token 环境变量）
    ├── Release Channels（stable / latest）
    └── 容器预填充（CLAUDE_CODE_PLUGIN_SEED_DIR）
```

---

## 笔记区

> **待解决问题**：
> - [ ] 如何在实际团队中管理多个 Plugin 的版本兼容性？
> - [ ] 对比 GitHub Marketplace 和 npm 分发方式的优劣（权限、更新延迟、私有支持）

> **关键发现**：

> **疑问**：
