---
title: "【实践2】Permission Modes：掌控Claude能做什么、不能做什么"
publish: true
---

# 【实践2】Permission Modes：掌控 Claude 能做什么、不能做什么

> 理解 5 种权限模式的工作原理、Auto Mode 的分类器边界、Permission Rules 的精确语法，保护敏感文件。

**参考文档**：
- https://code.claude.com/docs/en/permission-modes.md — Choose a permission mode
- https://code.claude.com/docs/en/permissions.md — Configure permissions
- https://code.claude.com/docs/en/auto-mode-config.md — Configure auto mode

**先修知识**：[[【2】Claude Code配置体系详解]]

**学习目标**：理解 5 种模式差异、掌握 Auto Mode 分类器原理、学会编写 Permission Rules、了解 Sandbox 隔离

---

## 1. 5 种模式速览（Default / AcceptEdits / Plan / Auto / DontAsk / Bypass 对比与切换方式）

Claude Code 提供了 6 种权限模式（准确说是 5 种 + Bypass 特殊模式），每种模式在**便利性**和**安全性**之间做了不同的取舍。

### 模式对比表

| 模式 | 无需询问即可执行的操作 | 适用场景 |
|------|----------------------|----------|
| `default` | 仅读取操作 | 刚上手、敏感工作 |
| `acceptEdits` | 读取、文件编辑、常用文件系统命令（`mkdir`/`touch`/`mv`/`cp` 等） | 迭代已有代码时减少审批 |
| `plan` | 仅读取操作 | 探索代码库但暂不修改 |
| `auto` | 全部操作（后台安全检查） | 长任务、减少弹窗疲劳 |
| `dontAsk` | 仅预先批准的工具 | 锁定环境的 CI/脚本 |
| `bypassPermissions` | 全部操作 | **仅限**隔离容器/VM |

> **关键约束**：除 `bypassPermissions` 外，所有模式对 **Protected Paths** 的写入都**不会自动批准**。

### 切换方式

**CLI 中**：
- `Shift+Tab` 循环切换：`default` → `acceptEdits` → `plan`
- `auto` 模式需要账户满足条件后才会出现在循环中
- `bypassPermissions` 需要 `--permission-mode bypassPermissions` 或 `--dangerously-skip-permissions` 启动
- `dontAsk` 不会出现在循环中，只能用 `--permission-mode dontAsk` 启动
- 启动时指定：`claude --permission-mode plan`
- 设为默认（settings.json）：

```json
{
  "permissions": {
    "defaultMode": "acceptEdits"
  }
}
```

**VS Code 中**：
- 点击输入框底部的模式指示器
- 设置 `claudeCode.initialPermissionMode`

**桌面版 / claude.ai**：使用发送按钮旁的模式下拉菜单。

**Web / 移动端**：
- Cloud session：仅支持 "Auto accept edits" 和 "Plan mode"
- Remote Control：支持 "Ask permissions"、"Auto accept edits"、"Plan mode"

### 各模式要点

#### default
- 标准行为：每个工具首次使用时弹出权限请求
- 选择 "Yes, don't ask again" 后，该命令在该项目目录下永久记录到权限规则

#### acceptEdits
- 状态栏显示 `⏵⏵ accept edits on`
- 自动批准的 Bash 命令：`mkdir`、`touch`、`rm`、`rmdir`、`mv`、`cp`、`sed`
- 仅限 working directory 和 `additionalDirectories` 范围内的路径
- Protected paths 和范围外的路径仍然会弹窗

#### plan
- Claude 可以读取文件、运行只读 shell 命令来探索，但**不会修改源码**
- 前置 `/plan` 可在单条 prompt 中使用 plan mode
- Plan 完成后提供选项：Approve（可选择不同模式继续编辑）/ Keep planning / Refine with Ultraplan
- `Ctrl+G` 可在文本编辑器中直接编辑 plan

#### dontAsk
- 非交互模式：所有会弹窗的工具调用都被自动拒绝
- 只有 `permissions.allow` 规则中的工具和只读 Bash 命令可以执行
- 适用于 CI pipeline

#### bypassPermissions
- **禁止**在 root/sudo 下启动（除非在沙箱中）
- v2.1.126+ 连 Protected Paths 写入也不会弹窗
- 仅限隔离环境（容器、VM、dev container）

---

## 2. Auto Mode 深度解析（分类器判断逻辑、边界、fallback 机制）

Auto Mode 是 Claude Code 中最复杂的权限模式。它不是一个简单的"全部允许"，而是引入了一个**独立的分类器模型**来审查每个操作。

### 启用条件

Auto Mode 仅在以下条件**全部满足**时可用：
- **套餐**：Max、Team、Enterprise 或 API（Pro 不可用）
- **管理员**：Team/Enterprise 需管理员在后台启用（管理员可设为 `disable` 彻底关闭）
- **模型**：Claude Sonnet 4.6、Opus 4.6、Opus 4.7（Max 仅 Opus 4.7）
- **提供商**：仅限 Anthropic API（Bedrock/Vertex/Foundry 不可用）
- **版本**：Claude Code v2.1.83+

### 分类器判断流程

每个操作按**固定优先级**依次判断，第一条匹配即止：

1. **用户自定义规则优先**：匹配 `allow` / `deny` 规则立即决定
2. **只读操作和工作目录文件编辑**：自动批准（Protected Paths 除外）
3. **其余操作**：交给分类器
4. **分类器阻止时**：Claude 收到原因并尝试替代方案

### 默认阻止 vs 允许的操作

| 默认阻止 | 默认允许 |
|---------|---------|
| `curl \| bash` 下载并执行代码 | 本地文件操作（working directory） |
| 向外部端点发送敏感数据 | 安装 lock file 中声明的依赖 |
| 生产部署和迁移 | 读取 `.env` 并向匹配的 API 发送凭据 |
| 云存储批量删除 | 只读 HTTP 请求 |
| 授予 IAM/仓库权限 | 推送到起始分支或 Claude 创建的分支 |
| 修改共享基础设施 | |
| 不可逆地删除会话前就存在的文件 | |
| force push 或直接推送到 `main` | |

> 运行 `claude auto-mode defaults` 查看完整规则列表。

### 对话边界（Boundaries）

用户在对话中明确说的"不要 push"等指令，会被分类器当作 block 信号。即使默认规则允许的操作，只要有明确的用户边界声明就会被阻止。边界**持续有效**直到用户明确解除。

**限制**：边界不是持久存储的规则。上下文压缩（context compaction）如果移除了包含边界的消息，边界可能丢失。需要硬保证请使用 deny 规则。

### Fallback 机制

当分类器连续拒绝操作时：
- **连续 3 次**拒绝 → Auto Mode 暂停，恢复弹窗模式
- **累计 20 次**拒绝 → Auto Mode 暂停
- 用户批准弹窗后可恢复 Auto Mode
- 计数器：允许的操作重置连续计数器；累计计数器在整个会话中持续，达到 20 后重置
- 非交互模式（`-p`）多次阻拦会直接中止会话

### 子代理（Subagent）的处理

分类器在**三个时点**审查子代理：
1. **启动前**：评估委托的任务描述，危险任务在 spawn 时即被阻止
2. **运行时**：子代理的每个操作都独立通过分类器
3. **结束时**：审查完整操作历史，发现问题会在结果中附加安全警告

### 成本和延迟

- 分类器使用独立的服务器模型，与用户 `/model` 选择无关
- 分类器调用计入 token 用量
- 每次检查发送部分对话历史 + 待定操作，增加一个往返延迟
- 只读操作和工作目录文件编辑**跳过**分类器

### 配置可信基础设施

```json
{
  "autoMode": {
    "environment": [
      "$defaults",
      "Source control: github.example.com/acme-corp and all repos under it",
      "Trusted cloud buckets: s3://acme-build-artifacts, gs://acme-ml-datasets",
      "Trusted internal domains: *.corp.example.com, api.internal.example.com",
      "Key internal services: Jenkins at ci.example.com, Artifactory at artifacts.example.com"
    ]
  }
}
```

**重要规则**：
- 使用 `"$defaults"` 保留内置规则并添加自己的条目
- 不使用 `"$defaults"` 会**完全替换**默认列表
- 配置读取范围：用户 settings / local settings / managed settings，但**不读取**共享项目 settings（`.claude/settings.json`）

### 自定义规则优先级

```
hard_deny → soft_deny → allow → 用户显式意图
```

- `hard_deny`：无条件阻止（用户意图和 allow 例外都无效）
- `soft_deny`：阻止，但用户显式意图和 allow 可覆盖
- `allow`：覆盖 matching soft_deny 的例外规则
- **用户显式意图**：用户消息中直接明确描述即将执行的操作时，可覆盖 soft_deny

### 诊断命令

```bash
claude auto-mode defaults   # 打印内置规则（JSON）
claude auto-mode config     # 打印实际生效的配置
claude auto-mode critique   # AI 审查自定义规则
```

---

## 3. Permission Rules 语法（工具特定规则、通配符、Bash/Read/Edit/WebFetch/MCP/Agent 各自控制）

### 规则类型与评估顺序

三条规则类型：`allow` / `ask` / `deny`

**评估优先级**：`deny` > `ask` > `allow`

即 deny 规则具有最高优先级，在任何层级中定义的 deny 都会阻止该操作。

### 设置层级优先级

1. Managed settings（不可被覆盖）
2. 命令行参数
3. 本地项目设置（`.claude/settings.local.json`）
4. 共享项目设置（`.claude/settings.json`）
5. 用户设置（`~/.claude/settings.json`）

如果**任何层级**有 deny，其他层级都无法 allow。

### 通用语法

```json
{
  "permissions": {
    "allow": [
      "Tool",
      "Tool(specifier)"
    ],
    "deny": [
      "Tool(specifier)"
    ]
  }
}
```

### Bash 规则

**基础用法**：
- `Bash` = `Bash(*)` = 匹配所有 Bash 命令
- `Bash(npm run build)` = 精确匹配
- `Bash(npm run *)` = 通配符匹配前缀

**通配符行为**：
- `*` 可出现在任意位置，匹配任意字符序列（包括空格）
- `Bash(ls *)` 和 `Bash(ls:*)` 要求前缀后跟空格或字符串结束（防止匹配 `lsof`）
- `Bash(ls*)` 无空格要求，会同时匹配 `ls -la` 和 `lsof`

**复合命令处理**：
Claude Code 能识别 shell 操作符（`&&`、`||`、`;`、`|`、`|&`、`&`、换行），要求规则**对每个子命令独立匹配**。例如 `Bash(safe-cmd *)` 不会允许 `safe-cmd && other-cmd`。

当你选择 "Yes, don't ask again" 批准复合命令时，每条需要批准的子命令都会保存为独立规则。

**进程包装器（Process Wrappers）**：
内置可剥离的包装器：`timeout`、`time`、`nice`、`nohup`、`stdbuf`、`xargs`（无 flag 时）
这样规则 `Bash(npm test *)` 也能匹配 `timeout 30 npm test`。

**非包装器**（不会被剥离）：`direnv exec`、`devbox run`、`mise exec`、`npx`、`docker exec`

**只读命令（Read-only Commands）**：
内置只读命令集：`ls`、`cat`、`echo`、`pwd`、`head`、`tail`、`grep`、`find`、`wc`、`which`、`diff`、`stat`、`du`、`cd`、只读形式的 `git`

这些命令在所有模式下都无需权限提示。如需限制，用 `ask` 或 `deny` 规则。

### PowerShell 规则

与 Bash 规则相同格式，支持通配符和 `:*` 后缀。
别名在匹配前被规范化，因此 `PowerShell(Get-ChildItem *)` 也匹配 `gci`、`ls`、`dir`。
大小写无关。

```json
{
  "permissions": {
    "allow": [
      "PowerShell(Get-ChildItem *)",
      "PowerShell(git commit *)"
    ],
    "deny": [
      "PowerShell(Remove-Item *)"
    ]
  }
}
```

### Read 和 Edit 规则

遵循 **gitignore 规范**，支持四种路径模式：

| 模式 | 含义 | 示例 | 匹配 |
|------|------|------|------|
| `//path` | 文件系统根绝对路径 | `//Users/alice/secrets/**` | `/Users/alice/secrets/**` |
| `~/path` | home 目录路径 | `~/Documents/*.pdf` | `/Users/alice/Documents/*.pdf` |
| `/path` | 项目根相对路径 | `/src/**/*.ts` | 项目根下的 `/src/**/*.ts` |
| `path` / `./path` | 当前目录相对路径 | `*.env` | 当前目录下的 `.env` |

> **注意**：`/Users/alice/file` 不是绝对路径！它被解释为项目根下的路径。要用 `//Users/alice/file`。

**Read/Edit deny 的应用范围**：
- 作用于 Claude 的内置文件工具和 Claude Code 识别的 Bash 文件命令（`cat`、`head`、`tail`、`sed` 等）
- **不作用于**间接读写文件的子进程（如 Python/Node 脚本内部的文件操作）
- 如需 OS 级强制，请启用 sandbox

**符号链接（Symlink）处理**：
- Allow 规则：要求 symlink 路径和目标路径都匹配才生效。目录内指向目录外的 symlink 仍会弹窗。
- Deny 规则：symlink 或目标任一匹配即阻止。

### WebFetch 规则

```json
{
  "permissions": {
    "allow": ["WebFetch(domain:example.com)"]
  }
}
```

### MCP 规则

```json
{
  "permissions": {
    "allow": [
      "mcp__puppeteer",                // puppeteer 服务器的所有工具
      "mcp__puppeteer__*",             // 同上，通配符形式
      "mcp__puppeteer__puppeteer_navigate" // 精准匹配单个工具
    ]
  }
}
```

### Agent（子代理）规则

```json
{
  "permissions": {
    "deny": [
      "Agent(Explore)",
      "Agent(Plan)",
      "Agent(my-custom-agent)"
    ]
  }
}
```

也可用 `--disallowedTools` CLI flag 禁用特定子代理。

### Auto Mode 中的规则特殊行为

进入 Auto Mode 时，以下宽泛规则会被**丢弃**：
- 空泛的 `Bash(*)` 或 `PowerShell(*)`
- 通配的解释器如 `Bash(python*)`
- 包管理器运行命令
- `Agent` allow 规则

窄规则如 `Bash(npm test)` 会保留。离开 Auto Mode 时丢弃的规则会恢复。

### URL 过滤注意事项

用 `Bash(curl http://github.com/ *)` 来限制 curl 到 GitHub 是不可靠的，因为：
- 选项可能在 URL 前：`curl -X GET http://github.com/...`
- 不同协议：`curl https://github.com/...`
- 重定向：`curl -L http://bit.ly/xyz`
- 变量：`URL=http://github.com && curl $URL`

更可靠的做法：用 deny 规则限制 `curl`/`wget`，然后用 `WebFetch(domain:github.com)` 允许 WebFetch 工具访问。

> 注意：仅使用 WebFetch 不阻止网络访问。如果 Bash 被允许，Claude 仍可用 `curl` 等工具访问任意 URL。

---

## 4. Protected Paths（保护敏感配置文件不被修改）

Protected Paths 是 Claude Code 的安全底线——在所有模式（`bypassPermissions` 除外）下，对这些路径的写入**永远不会自动批准**。

### 受保护的目录

| 目录 | 说明 |
|------|------|
| `.git/` | 仓库状态 |
| `.vscode/` | VS Code IDE 配置 |
| `.idea/` | JetBrains IDE 配置 |
| `.husky/` | Git hooks |
| `.claude/` | Claude Code 自身配置（`commands/`、`agents/`、`skills/`、`worktrees/` 除外） |

### 受保护的文件

| 文件 | 说明 |
|------|------|
| `.gitconfig`、`.gitmodules` | Git 配置 |
| `.bashrc`、`.bash_profile`、`.zshrc`、`.zprofile`、`.profile` | Shell 配置 |
| `.ripgreprc` | ripgrep 配置 |
| `.mcp.json`、`.claude.json` | MCP / Claude 配置 |

### 各模式下的行为

| 模式 | Protected Paths 写入行为 |
|------|------------------------|
| `default` | 弹窗询问 |
| `acceptEdits` | 弹窗询问（普通文件编辑自动批准，但 Protected Paths 不行） |
| `plan` | 弹窗询问 |
| `auto` | 路由到分类器 |
| `dontAsk` | 被拒绝 |
| `bypassPermissions` | 允许（v2.1.126+） |

---

## 5. Sandbox 简介（文件系统隔离、网络隔离、OS 级强制）

Sandbox 提供**操作系统级的强制隔离**，与 Permissions 系统互补。

### 分层防御

| 层 | 覆盖范围 | 控制方式 |
|----|---------|---------|
| Permission Rules | 所有工具（Bash/Read/Edit/WebFetch/MCP/Agent） | settings.json 规则 |
| Sandbox | 仅 Bash 命令及其子进程 | OS 级隔离 |
| 二者结合 | Permissions deny + Sandbox 边界合并 | 深度防御 |

### 关键行为

- **文件系统限制**：合并 `sandbox.filesystem` 设置与 Read/Edit deny 规则
- **网络限制**：合并 WebFetch 权限规则与 sandbox 的 `allowedDomains`/`deniedDomains` 列表
- **autoAllowBashIfSandboxed**（默认 `true`）：启用 sandbox 时，Bash 命令在沙箱内无需弹窗——沙箱边界替代了逐命令弹窗。
  - 显式 deny 规则仍然适用
  - 针对 `/`、home 目录等关键系统路径的 `rm`/`rmdir` 仍然弹窗

### 可配置的 sandbox 设置

Sandbox 支持 `filesystem.allowRead`、`filesystem.denyRead`、`network.allowedDomains`、`network.deniedDomains` 等配置项。Managed settings 中还有 `allowManagedReadPathsOnly` 和 `allowManagedDomainsOnly` 等严格模式。

---

## 6. 实战配置示例（个人开发 vs 团队协作两种场景）

### 场景一：个人开发者

**需求**：日常编码减少弹窗，但保护敏感配置不被意外修改。

```json
{
  "permissions": {
    "defaultMode": "acceptEdits",
    "allow": [
      "Bash(npm *)",
      "Bash(git commit *)",
      "Bash(git push origin *)",
      "Bash(git status *)",
      "Bash(gh pr *)"
    ],
    "deny": [
      "Bash(git push origin main)",
      "Bash(git push * main)",
      "Bash(git push --force *)",
      "Bash(rm -rf *)"
    ]
  }
}
```

**理由**：`acceptEdits` 模式允许文件编辑自动批准；`allow` 规则覆盖日常 npm 和 git 操作；`deny` 规则防止意外推送到 main 和破坏性删除。

### 场景二：团队协作 / CI 环境

**需求**：严格限制执行范围，CI 环境零交互。

**项目 settings.json（`.claude/settings.json`）**：

```json
{
  "permissions": {
    "defaultMode": "dontAsk",
    "allow": [
      "Bash(npm test *)",
      "Bash(npm run build *)",
      "Bash(npm run lint *)",
      "Bash(npx playwright test *)"
    ],
    "deny": [
      "Bash(npm publish *)",
      "Bash(npm run deploy *)",
      "Bash(gh pr merge *)",
      "Bash(rm -rf *)",
      "Agent(Explore)",
      "Agent(Plan)"
    ]
  }
}
```

**Managed settings（组织级）**：

```json
{
  "permissions": {
    "allowManagedPermissionRulesOnly": true,
    "disableBypassPermissionsMode": "disable",
    "sandbox": {
      "filesystem": {
        "allowRead": ["//**"],
        "denyRead": ["//etc/**", "//home/**/.ssh/**"]
      },
      "network": {
        "allowedDomains": ["*.internal.company.com", "registry.npmjs.org"],
        "deniedDomains": ["*"]
      }
    }
  }
}
```

**理由**：
- `dontAsk` 模式使 CI 脚本零交互
- 只允许测试、构建、lint 命令
- Managed settings 锁定 bypass 模式，团队成员只能使用受限模式
- Sandbox 限制网络和文件系统边界

### Auto Mode 团队配置参考

```json
{
  "autoMode": {
    "environment": [
      "$defaults",
      "Organization: ACME Corp. Primary use: software development",
      "Source control: github.com/acme-corp and github.com/acme-oss",
      "Cloud provider: AWS",
      "Trusted cloud buckets: s3://acme-build-artifacts, s3://acme-lambda-zips",
      "Trusted internal domains: *.acme.internal, api.acme.com",
      "Key internal services: Jenkins at ci.acme.com, Artifactory at artifacts.acme.com"
    ]
  }
}
```

---

## 学习检验

- [ ] 我能说出 5 种权限模式（不含 Bypass）的核心区别
- [ ] 我理解 Auto Mode 分类器的三层判断流程
- [ ] 我知道如何编写含通配符的 Bash permission rule
- [ ] 我理解 Protected Paths 保护的路径范围
- [ ] 我知道 Sandbox 和 Permissions 的分工关系
- [ ] 我能为个人项目和团队项目分别设计权限方案

---

## 个人笔记

<!-- 在此记录学习过程中的疑问、心得或补充发现 -->

-
-
-
