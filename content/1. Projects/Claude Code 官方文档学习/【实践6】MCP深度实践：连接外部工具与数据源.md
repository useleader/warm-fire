---
title: 【实践6】MCP深度实践：连接外部工具与数据源
publish: true
---

# 【实践6】MCP 深度实践：连接外部工具与数据源

> 从安装MCP Server到生产级配置——三种传输方式、认证机制、Tool Search、企业管控全覆盖。

**参考文档**：
- `code.claude.com/docs/en/mcp.md` — Connect Claude Code to tools via MCP
- `code.claude.com/docs/en/agent-sdk/mcp.md` — MCP in the SDK
- `code.claude.com/docs/en/agent-sdk/tool-search.md` — Tool search

**先修知识**：已有MCP理论基础（Context Course）

**学习目标**：掌握三种安装方式、理解作用域层级、学会实战配置、了解OAuth认证、了解Tool Search和企业管控

---

## 1. MCP Server三种安装方式（HTTP / SSE / stdio 的差异、选型、配置语法）

MCP Server 可以通过三种传输（transport）方式与 Claude Code 通信，选择取决于服务器的部署方式和使用场景。

### 1.1 HTTP（streamable-http）— 推荐用于远程服务

HTTP 是推荐的远程传输方式，适用于云服务。MCP 规范中称为 `streamable-http`，在 `.mcp.json` 等 JSON 配置中 `type` 字段可接受 `streamable-http` 作为 `http` 的别名。

```bash
# 基本语法
claude mcp add --transport http <name> <url>

# 示例：连接 Notion
claude mcp add --transport http notion https://mcp.notion.com/mcp

# 带 Bearer token 认证
claude mcp add --transport http secure-api https://api.example.com/mcp \
  --header "Authorization: Bearer your-token"
```

**选型建议**：优先选择 HTTP。它支持 OAuth 2.0 认证、自动重连（exponential backoff）、以及远程集中管理。

### 1.2 SSE（Server-Sent Events）— 已弃用

SSE 传输方式已被官方标记为 deprecated，建议尽可能迁移到 HTTP：

```bash
claude mcp add --transport sse <name> <url>

# 示例：连接 Asana
claude mcp add --transport sse asana https://mcp.asana.com/sse

# 带认证 header
claude mcp add --transport sse private-api https://api.company.com/sse \
  --header "X-API-Key: your-key-here"
```

### 1.3 stdio — 用于本地进程

Stdio 服务器作为本地子进程运行，适合需要直接系统访问或自定义脚本的场景：

```bash
# 基本语法：所有选项必须在 server name 之前，-- 之后是命令和参数
claude mcp add [options] <name> -- <command> [args...]

# 示例：添加 Airtable 服务器
claude mcp add --transport stdio --env AIRTABLE_API_KEY=YOUR_KEY airtable \
  -- npx -y airtable-mcp-server
```

**关键行为**：
- Claude Code 会在启动时设置 `CLAUDE_PROJECT_DIR` 环境变量，指向项目根目录，方便服务器解析项目相对路径
- Stdio 服务器**不会**自动重连（只有 HTTP/SSE 支持自动重连，最多 5 次，指数退避）
- 可通过 `MCP_TIMEOUT` 环境变量配置服务器启动超时（如 `MCP_TIMEOUT=10000 claude` 设置 10 秒）

> **注意**：选项顺序很重要。所有选项（`--transport`、`--env`、`--scope`、`--header`）必须放在 server name 之前，`--`（双横线）分隔 server name 与命令参数，避免 Claude Code 的标志与服务器标志冲突。

### 传输方式对比

| 特性 | HTTP | SSE | stdio |
|------|------|-----|-------|
| 远程调用 | 是 | 是 | 否（本地进程） |
| OAuth 认证 | 支持 | 支持 | 不支持 |
| 自动重连 | 是（指数退避） | 是（指数退避） | 否 |
| 推荐度 | 推荐 | 已弃用 | 本地工具推荐 |

> **笔记区**：

---

## 2. 安装作用域（Local .mcp.json vs Project .mcp.json vs User ~/.claude.json，优先级与合并规则）

MCP 服务器可以在三个作用域（scope）下配置，控制服务器在哪些项目中加载以及是否与团队共享。

### 作用域一览

| Scope | 加载范围 | 团队共享 | 存储位置 |
|-------|---------|---------|---------|
| **Local**（默认） | 仅当前项目 | 否 | `~/.claude.json`（按项目路径存储） |
| **Project** | 仅当前项目 | 是（通过版本控制） | `.mcp.json`（项目根目录） |
| **User** | 所有项目 | 否 | `~/.claude.json` |

### Local 作用域

默认作用域。服务器仅在你添加它的项目中可用，存储在你的 home 目录下的 `~/.claude.json` 中：

```bash
claude mcp add --transport http stripe https://mcp.stripe.com
```

`~/.claude.json` 中的存储格式：
```json
{
  "projects": {
    "/path/to/your/project": {
      "mcpServers": {
        "stripe": {
          "type": "http",
          "url": "https://mcp.stripe.com"
        }
      }
    }
  }
}
```

### Project 作用域

适合团队协作，配置存储在项目根目录的 `.mcp.json` 文件中，应纳入版本控制：

```bash
claude mcp add --transport http paypal --scope project https://mcp.paypal.com/mcp
```

生成的 `.mcp.json` 文件格式：
```json
{
  "mcpServers": {
    "shared-server": {
      "command": "/path/to/server",
      "args": [],
      "env": {}
    }
  }
}
```

> **安全**：Claude Code 会在使用 project-scoped 服务器前提示审批。如需重置审批选择，运行 `claude mcp reset-project-choices`。

### User 作用域

跨项目可用，存储在你的 `~/.claude.json` 中，私有于你的用户账户：

```bash
claude mcp add --transport http hubspot --scope user https://mcp.hubspot.com/anthropic
```

### 优先级与合并规则

当同一服务器在多个位置定义时，Claude Code 按以下优先级连接（高优先级覆盖低优先级）：

1. **Local 作用域**（最高优先级）
2. **Project 作用域**
3. **User 作用域**
4. **Plugin 提供的服务器**
5. **claude.ai Connectors**（最低优先级）

> 前三个作用域按**名称**匹配去重；Plugin 和 Connectors 按**端点**匹配去重。

### 环境变量展开

`.mcp.json` 支持环境变量展开，方便团队共享配置模板：

```json
{
  "mcpServers": {
    "api-server": {
      "type": "http",
      "url": "${API_BASE_URL:-https://api.example.com}/mcp",
      "headers": {
        "Authorization": "Bearer ${API_KEY}"
      }
    }
  }
}
```

支持的展开位置：`command`、`args`、`env`、`url`、`headers`。语法：`${VAR}` 或 `${VAR:-default}`。

> **笔记区**：

---

## 3. 实战示例（连接GitHub、PostgreSQL、Sentry的完整配置步骤）

### 3.1 GitHub — 代码审查与 Issue 管理

GitHub 的远程 MCP 服务器通过 GitHub Personal Access Token 进行认证：

```bash
# 1. 生成 GitHub PAT（fine-grained token，选择要访问的仓库）
# 2. 添加 MCP 服务器
claude mcp add --transport http github https://api.githubcopilot.com/mcp/ \
  --header "Authorization: Bearer YOUR_GITHUB_PAT"
```

**使用示例**：
```text
Review PR #456 and suggest improvements
Create a new issue for the bug we just found
Show me all open PRs assigned to me
```

### 3.2 PostgreSQL — 数据库查询

使用 stdio 方式连接本地数据库工具（如 @bytebase/dbhub）：

```bash
claude mcp add --transport stdio db -- npx -y @bytebase/dbhub \
  --dsn "postgresql://readonly:pass@prod.db.com:5432/analytics"
```

**使用示例**：
```text
What's our total revenue this month?
Show me the schema for the orders table
Find customers who haven't made a purchase in 90 days
```

### 3.3 Sentry — 错误监控

```bash
# 添加 Sentry MCP 服务器
claude mcp add --transport http sentry https://mcp.sentry.dev/mcp

# 在 Claude Code 中完成 OAuth 认证
/mcp
```

**使用示例**：
```text
What are the most common errors in the last 24 hours?
Show me the stack trace for error ID abc123
Which deployment introduced these new errors?
```

### 3.4 服务器管理命令

```bash
# 列出所有已配置的服务器
claude mcp list

# 查看特定服务器详情
claude mcp get github

# 移除服务器
claude mcp remove github

# 在 Claude Code 内部查看 MCP 状态
/mcp
```

`/mcp` 面板会显示每个连接服务器的工具数量，并标记那些声称支持 tools 功能但实际未暴露任何工具的服务器。

> **笔记区**：

---

## 4. OAuth认证与自定义鉴权（OAuth回调端口、动态Headers、Scope限制）

### 4.1 OAuth 2.0 认证流程

Claude Code 支持 OAuth 2.0 用于远程 HTTP 服务器的安全连接。当服务器返回 `401 Unauthorized` 或 `403 Forbidden` 时，Claude Code 会在 `/mcp` 中标记该服务器需要认证。

```bash
# 添加需要 OAuth 认证的服务器
claude mcp add --transport http sentry https://mcp.sentry.dev/mcp

# 在 Claude Code 中触发认证
/mcp   # 然后按照浏览器中的步骤登录
```

**Tips**：
- 认证令牌会安全存储并自动刷新
- 可通过 `/mcp` 菜单中的 "Clear authentication" 撤销访问权限
- 如果浏览器未自动打开，可手动复制提供的 URL
- 如果认证后浏览器重定向失败，将浏览器地址栏中的完整回调 URL 粘贴到 Claude Code 的 URL 提示中

### 4.2 固定 OAuth 回调端口

某些 MCP 服务器要求预先注册特定的 redirect URI。使用 `--callback-port` 固定端口：

```bash
claude mcp add --transport http \
  --callback-port 8080 \
  my-server https://mcp.example.com/mcp
```

### 4.3 预配置 OAuth 凭证

当服务器不支持自动 OAuth 设置（Dynamic Client Registration）时，需要预注册 OAuth 应用：

```bash
claude mcp add --transport http \
  --client-id your-client-id --client-secret --callback-port 8080 \
  my-server https://mcp.example.com/mcp
```

或通过 JSON 配置：
```bash
claude mcp add-json my-server \
  '{"type":"http","url":"https://mcp.example.com/mcp","oauth":{"clientId":"your-client-id","callbackPort":8080}}' \
  --client-secret
```

CI/CD 环境变量方式（跳过交互式输入）：
```bash
MCP_CLIENT_SECRET=your-secret claude mcp add --transport http \
  --client-id your-client-id --client-secret --callback-port 8080 \
  my-server https://mcp.example.com/mcp
```

### 4.4 覆盖 OAuth 元数据发现

通过 `authServerMetadataUrl` 指定 OAuth 授权服务器元数据 URL，绕过默认发现链：

```json
{
  "mcpServers": {
    "my-server": {
      "type": "http",
      "url": "https://mcp.example.com/mcp",
      "oauth": {
        "authServerMetadataUrl": "https://auth.example.com/.well-known/openid-configuration"
      }
    }
  }
}
```

### 4.5 限制 OAuth Scope

通过 `oauth.scopes` 限制授权范围，确保只请求安全团队批准的子集：

```json
{
  "mcpServers": {
    "slack": {
      "type": "http",
      "url": "https://mcp.slack.com/mcp",
      "oauth": {
        "scopes": "channels:read chat:write search:read"
      }
    }
  }
}
```

### 4.6 动态 Headers 自定义鉴权

对于非 OAuth 的鉴权方案（如 Kerberos、短生命期 Token、内部 SSO），使用 `headersHelper`：

```json
{
  "mcpServers": {
    "internal-api": {
      "type": "http",
      "url": "https://mcp.internal.example.com",
      "headersHelper": "/opt/bin/get-mcp-auth-headers.sh"
    }
  }
}
```

内联命令形式：
```json
{
  "mcpServers": {
    "internal-api": {
      "type": "http",
      "url": "https://mcp.internal.example.com",
      "headersHelper": "echo '{\"Authorization\": \"Bearer '$(get-token)'\"}'"
    }
  }
}
```

**要求**：
- 命令必须输出 JSON 对象（string key-value pairs）到 stdout
- 在 shell 中执行，超时 10 秒
- 动态 headers 会覆盖同名静态 headers
- 每次连接（会话启动和重连时）重新运行，不缓存
- 设置 `CLAUDE_CODE_MCP_SERVER_NAME` 和 `CLAUDE_CODE_MCP_SERVER_URL` 环境变量给 helper

> **笔记区**：

---

## 5. MCP Tool Search（大量工具时的按需发现机制、配置与优化）

### 5.1 什么是 Tool Search

当 MCP 工具数量庞大时，Tool Search 通过**按需延迟加载**机制解决上下文窗口占用问题。默认启用。

**解决的问题**：
- **上下文效率**：50 个工具的定义可能消耗 10-20K tokens
- **选择准确性**：一次性加载超过 30-50 个工具时，模型选择准确性下降

### 5.2 工作原理

1. 工具定义被 withheld（保留在上下文之外）
2. 模型收到可用工具的摘要信息
3. 当任务需要某能力时，模型搜索相关工具
4. 搜索返回 3-5 个最相关的工具并加载到上下文
5. 发现的工具在后续轮次中保持可用
6. 长对话 compact 时可能移除已发现的工具，模型会重新搜索

**代价**：首次发现工具时增加一次往返（search step），但对于大型工具集，每轮更小的上下文足以抵消这个开销。

### 5.3 配置选项

通过 `ENABLE_TOOL_SEARCH` 环境变量控制：

| 值 | 行为 |
|----|------|
| （未设置） | 默认启用。在 Vertex AI 或非 first-party `ANTHROPIC_BASE_URL` 时回退到 upfront 加载 |
| `true` | 强制启用，即使在 Vertex AI 或通过代理 |
| `auto` | 阈值模式：工具定义超过上下文窗口 10% 时激活 |
| `auto:N` | 自定义百分比阈值，如 `auto:5` 表示 5% |
| `false` | 禁用，所有工具 upfront 加载 |

```bash
# 使用自定义 5% 阈值
ENABLE_TOOL_SEARCH=auto:5 claude

# 完全禁用 Tool Search
ENABLE_TOOL_SEARCH=false claude
```

### 5.4 SDK 中配置 Tool Search

```typescript
const options = {
  mcpServers: {
    "enterprise-tools": {
      type: "http",
      url: "https://tools.example.com/mcp"
    }
  },
  allowedTools: ["mcp__enterprise-tools__*"],
  env: {
    ENABLE_TOOL_SEARCH: "auto:5"
  }
};
```

### 5.5 豁免服务器免于延迟加载

当某服务器工具需要始终对模型可见时，设置 `alwaysLoad: true`：

```json
{
  "mcpServers": {
    "core-tools": {
      "type": "http",
      "url": "https://mcp.example.com/mcp",
      "alwaysLoad": true
    }
  }
}
```

MCP Server 也可以在单个工具上设置 `"anthropic/alwaysLoad": true`（在工具的 `_meta` 对象中）。

> **注意**：`alwaysLoad: true` 也会阻塞启动，直到服务器连接成功（上限 5 秒）。

### 5.6 优化工具发现

搜索机制匹配工具名称和描述：

- **命名**：`search_slack_messages` 比 `query_slack` 更容易被匹配
- **描述**：带关键词的描述（"Search Slack messages by keyword, channel, or date range"）比泛泛的描述（"Query Slack"）匹配更准确
- **系统提示**：添加工具类别概览帮助模型搜索：
  ```text
  You can search for tools to interact with Slack, GitHub, and Jira.
  ```

### 5.7 限制

- 最大工具数：10,000
- 每次搜索返回：3-5 个最相关工具
- 模型支持：Sonnet 4+、Opus 4+（不支持 Haiku）
- Claude Code 截断工具描述和服务器 instructions 到 2KB 以内

> **笔记区**：

---

## 6. MCP Prompts作为命令使用（/mcp prompt机制）

MCP 服务器可以暴露 prompts，这些 prompts 可以作为 Claude Code 中的命令使用。

### 6.1 发现可用 Prompts

在 Claude Code 中键入 `/` 可以看到所有可用命令，包括来自 MCP 服务器的 prompts。格式为 `/mcp__servername__promptname`。

### 6.2 执行 Prompt

```text
# 无参数
/mcp__github__list_prs

# 带参数（空格分隔）
/mcp__github__pr_review 456

/mcp__jira__create_issue "Bug in login flow" high
```

### 6.3 关键行为

- 已连接服务器的 prompts 会被动态发现
- 参数基于 prompt 定义的参数进行解析
- Prompt 结果直接注入对话
- 服务器名称和 prompt 名称会被规范化（空格变为下划线）

> **笔记区**：

---

## 7. 输出限制与Channels（结果截断处理、频道推送到session）

### 7.1 输出限制

当 MCP 工具产生大型输出时，Claude Code 帮助管理 token 用量：

| 参数 | 默认值 | 说明 |
|------|--------|------|
| 警告阈值 | 10,000 tokens | 超出时显示 warning |
| 最大限制 | 25,000 tokens | 可通过 `MAX_MCP_OUTPUT_TOKENS` 调整 |
| 硬上限 | 500,000 chars | 通过 `anthropic/maxResultSizeChars` 设置 |

```bash
# 增大限制
export MAX_MCP_OUTPUT_TOKENS=50000
claude
```

### 7.2 为特定工具提升限制

如果你是 MCP 服务器开发者，可以在工具的 `tools/list` 响应中设置 `_meta["anthropic/maxResultSizeChars"]`：

```json
{
  "name": "get_schema",
  "description": "Returns the full database schema",
  "_meta": {
    "anthropic/maxResultSizeChars": 200000
  }
}
```

> 该注解仅对文本内容独立生效（不受 `MAX_MCP_OUTPUT_TOKENS` 影响），但图像内容仍受 token 限制约束。

### 7.3 Channels — 推送消息到 Session

MCP 服务器可以通过 channels 将消息推送到会话中，使 Claude 能够响应外部事件（如 CI 结果、监控告警、聊天消息）。

**工作机制**：
1. 服务器声明 `claude/channel` 能力
2. 在启动时通过 `--channels` 标志启用
3. 外部事件通过 channel 推送到会话
4. Claude 在对话中实时响应

```bash
claude --channels
```

> **笔记区**：

---

## 8. Managed MCP：企业级管控（managed-mcp.json / allowlist / denylist）

### 8.1 Option 1: 独占控制 — managed-mcp.json

部署 `managed-mcp.json` 后，用户**无法**添加、修改或使用任何其他 MCP 服务器。

**文件位置**（需管理员权限）：
| 平台 | 路径 |
|------|------|
| macOS | `/Library/Application Support/ClaudeCode/managed-mcp.json` |
| Linux / WSL | `/etc/claude-code/managed-mcp.json` |
| Windows | `C:\Program Files\ClaudeCode\managed-mcp.json` |

**格式**（与 `.mcp.json` 相同）：
```json
{
  "mcpServers": {
    "github": {
      "type": "http",
      "url": "https://api.githubcopilot.com/mcp/"
    },
    "sentry": {
      "type": "http",
      "url": "https://mcp.sentry.dev/mcp"
    },
    "company-internal": {
      "type": "stdio",
      "command": "/usr/local/bin/company-mcp-server",
      "args": ["--config", "/etc/company/mcp-config.json"],
      "env": {
        "COMPANY_API_URL": "https://internal.company.com"
      }
    }
  }
}
```

### 8.2 Option 2: 策略控制 — Allowlist / Denylist

在受管设置文件中使用 `allowedMcpServers` 和 `deniedMcpServers` 实现基于策略的控制。

**限制方式**（每个条目只能选一种）：
| 字段 | 匹配目标 | 适用场景 |
|------|---------|---------|
| `serverName` | 服务器名称 | 用名称限制 |
| `serverCommand` | 确切命令+参数 | stdio 服务器 |
| `serverUrl` | URL 模式（支持通配符 `*`） | 远程服务器 |

**示例配置**：
```json
{
  "allowedMcpServers": [
    { "serverName": "github" },
    { "serverName": "sentry" },
    { "serverCommand": ["npx", "-y", "@modelcontextprotocol/server-filesystem"] },
    { "serverUrl": "https://mcp.company.com/*" },
    { "serverUrl": "https://*.internal.corp/*" }
  ],
  "deniedMcpServers": [
    { "serverName": "dangerous-server" },
    { "serverUrl": "https://*.untrusted.com/*" }
  ]
}
```

**通配符示例**：
- `https://mcp.company.com/*` — 允许特定域名的所有路径
- `https://*.example.com/*` — 允许任何子域名
- `http://localhost:*/*` — 允许 localhost 的任何端口

### 8.3 行为规则

| 配置 | 行为 |
|------|------|
| `allowedMcpServers: undefined` | 无限制（默认） |
| `allowedMcpServers: []` | 完全封锁，用户不能配置任何 MCP 服务器 |
| `deniedMcpServers: undefined/[]` | 无服务器被屏蔽（默认） |
| 两者结合 | Denylist 绝对优先——即使在 allowlist 中的服务器，如果匹配 denylist 也会被屏蔽 |

> **重要**：Option 1 和 Option 2 可以结合使用。如果 `managed-mcp.json` 存在，它具有独占控制权，用户无法添加服务器。但 allowlist/denylist 仍会作用于受管服务器本身。

> **笔记区**：

---

## 总结与最佳实践

| 场景 | 推荐方案 |
|------|---------|
| 个人开发工具 | Local scope + stdio |
| 团队协作工具 | Project scope（`.mcp.json` 纳入版本控制） |
| 跨项目个人工具 | User scope |
| 远程云服务 | HTTP transport + OAuth |
| 大型工具集（>30 个工具） | 启用 Tool Search（默认） |
| 企业管控 | `managed-mcp.json` 或 allowlist/denylist |
| 敏感凭证 | 使用环境变量展开 `${VAR}`，避免硬编码 |

**关于 `allowedTools`**：在 SDK 中使用 MCP 时，推荐使用 `allowedTools` + 通配符（`mcp__servername__*`）而非 `permissionMode: "bypassPermissions"`，后者会禁用所有安全提示，范围过大。

---

> **延伸阅读**：
> - 自定义 MCP Server 构建指南
> - Plugin MCP 服务器：插件可以捆绑 MCP 服务器，自动提供工具和集成
> - Claude Code 作为 MCP Server：`claude mcp serve` 可将 Claude Code 自身暴露为 MCP 服务器
> - MCP Resources：通过 `@` 引用 MCP 资源（如 `@github:issue://123`）
> - 动态工具更新：支持 `list_changed` 通知，无需断开重连即可更新工具列表
