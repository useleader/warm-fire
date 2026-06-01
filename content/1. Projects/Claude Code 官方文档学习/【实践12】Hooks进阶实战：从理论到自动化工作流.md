---
title: 【实践12】Hooks进阶实战：从理论到自动化工作流
publish: true
---

# 【实践12】Hooks 进阶实战：从理论到自动化工作流

> Hook事件从SessionStart到SessionEnd覆盖会话全生命周期。掌握6个最实用的自动化场景，理解Prompt/Agent/HTTP三种Hook类型。

**参考文档**：
- `code.claude.com/docs/en/hooks-guide.md` — Automate workflows with hooks
- `code.claude.com/docs/en/hooks.md` — Hooks reference

**先修知识**：[[【8】Hooks自动化工作流]]

**学习目标**：了解Hook事件全景、掌握6个常用实战场景、理解三种Hook类型差异、学会调试Hook

---

## 1. Hook事件全景 —— Full Hook Event Lifecycle

Hook事件覆盖从会话启动到结束的每一个关键节点。以下是完整的生命周期事件表：

| Event (事件) | When it fires (触发时机) | Typical Use (典型用途) |
|---|---|---|
| `SessionStart` | 会话开始/恢复时 | 注入项目上下文、设置环境变量 |
| `Setup` | `--init-only` / `--init` / `--maintenance` 模式 | CI环境一次性准备 |
| `UserPromptSubmit` | 用户提交prompt后、Claude处理前 | prompt验证、注入额外上下文 |
| `UserPromptExpansion` | slash command展开为prompt时 | 控制命令展开行为 |
| `PreToolUse` | 工具调用**执行前** | 权限控制、参数校验、敏感文件保护 |
| `PermissionRequest` | 权限对话框弹出时 | 自动批准/拒绝权限请求 |
| `PermissionDenied` | auto mode拒绝工具调用时 | 记录拒绝原因、允许重试 |
| `PostToolUse` | 工具调用**成功完成后** | 代码格式化、日志记录 |
| `PostToolUseFailure` | 工具调用失败时 | 错误告警、自动修复 |
| `PostToolBatch` | 一批并行工具**全部**解析后 | 批量注入上下文 |
| `Notification` | Claude发送通知时 | 桌面通知、Slack推送 |
| `SubagentStart` | 子Agent被spawn时 | 注入子Agent上下文 |
| `SubagentStop` | 子Agent完成时 | 验证子Agent输出 |
| `TaskCreated` | TaskCreate工具创建任务时 | 强制执行命名规范 |
| `TaskCompleted` | 任务标记为完成时 | 检查完成标准 |
| `Stop` | Claude**完成回复**时 | 任务完成校验、持续工作循环 |
| `StopFailure` | API错误导致turn结束时 | 错误告警 |
| `TeammateIdle` | Agent团队队友即将空闲时 | 质量门禁 |
| `InstructionsLoaded` | CLAUDE.md等指令文件加载时 | 审计追踪 |
| `ConfigChange` | 配置文件变更时 | 配置变更审计/阻止 |
| `CwdChanged` | 工作目录变更时 | 环境变量重载 (direnv) |
| `FileChanged` | 被监听的文件发生变化时 | 自动重载.envrc等 |
| `WorktreeCreate` | worktree被创建时 | 替代默认git worktree逻辑 |
| `WorktreeRemove` | worktree被移除时 | 自定义VCS清理 |
| `PreCompact` | 上下文压缩**之前** | 阻止压缩、保存关键信息 |
| `PostCompact` | 上下文压缩**之后** | 重新注入关键指令 |
| `Elicitation` | MCP服务器请求用户输入时 | 编程化响应MCP对话 |
| `ElicitationResult` | 用户响应MCP elicitation后 | 观察/修改/阻止响应 |
| `SessionEnd` | 会话终止时 | 清理临时文件、记录会话统计 |

> **关键理解**：事件按节奏分为三类——一次性（SessionStart/SessionEnd）、每次turn（UserPromptSubmit/Stop）、每次工具调用（PreToolUse/PostToolUse）。`UserPromptSubmit`、`PostToolBatch`、`Stop`、`TeammateIdle`、`TaskCreated`、`TaskCompleted`、`WorktreeCreate`、`WorktreeRemove`、`CwdChanged` 不支持matcher，总是触发。

---

## 2. 实战场景一：代码自动格式化 —— Auto-format Code After Edits

**原理**：在`PostToolUse`事件中监听`Edit|Write`工具，提取文件路径后自动运行格式化工具。

**配置代码**（放入项目根目录 `.claude/settings.json`）：

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "jq -r '.tool_input.file_path' | xargs npx prettier --write"
          }
        ]
      }
    ]
  }
}
```

**工作原理**：
1. Claude执行`Edit`或`Write`工具修改文件
2. `PostToolUse`事件触发，matcher匹配`Edit|Write`
3. Hook通过stdin接收JSON，`jq`提取`tool_input.file_path`
4. 输出传递给`npx prettier --write`格式化文件

**适用工具**：Prettier (JS/TS/CSS)、Black (Python)、rustfmt (Rust)、gofmt (Go)

**注意事项**：
- `PostToolUse`无法撤销操作——工具已经执行完毕，Hook只是后续处理
- 需要安装`jq` (`brew install jq` / `apt-get install jq`)
- 如果Claude通过Bash修改文件（如`sed`命令），PostToolUse on Edit|Write不会触发。需要额外加`Stop` hook扫描working tree
- 要匹配MCP工具的文件写入，使用 `"matcher": "mcp__.*__write.*"` 这样的正则

---

## 3. 实战场景二：变更通知推送 —— Notification to Slack/WeCom

**原理**：`Notification`事件在Claude等待用户输入时触发，通过`matcher`区分不同类型的通知。

### macOS 桌面通知

```json
{
  "hooks": {
    "Notification": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "osascript -e 'display notification \"Claude Code needs your attention\" with title \"Claude Code\"'"
          }
        ]
      }
    ]
  }
}
```

### Linux（需要`notify-send`）

```json
{
  "hooks": {
    "Notification": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "notify-send 'Claude Code' 'Claude Code needs your attention'"
          }
        ]
      }
    ]
  }
}
```

### Windows PowerShell

```json
{
  "hooks": {
    "Notification": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "powershell.exe -Command \"[System.Reflection.Assembly]::LoadWithPartialName('System.Windows.Forms'); [System.Windows.Forms.MessageBox]::Show('Claude Code needs your attention', 'Claude Code')\""
          }
        ]
      }
    ]
  }
}
```

### matcher可选值

| Matcher | 触发时机 |
|---|---|
| `permission_prompt` | Claude需要你批准工具使用时 |
| `idle_prompt` | Claude完成工作，等待你的下一个输入 |
| `auth_success` | 认证完成时 |
| `elicitation_dialog` | MCP服务器打开elicitation表单时 |
| `elicitation_complete` | Elicitation表单提交或关闭时 |
| `elicitation_response` | Elicitation响应发回服务器时 |

### macOS通知权限注意事项

如果通知没有弹出：`osascript`通过Script Editor路由通知。如果Script Editor没有通知权限，命令会静默失败，macOS也不会提示你授权。解决方法：

```bash
# 先运行一次让Script Editor出现在通知设置中
osascript -e 'display notification "test"'
# 然后打开 系统设置 > 通知 > Script Editor，开启"允许通知"
```

---

## 4. 实战场景三：敏感文件保护 —— Block Edits to Protected Files

**原理**：在`PreToolUse`事件中拦截`Edit|Write`工具调用，检查目标文件路径是否匹配保护模式。使用**exit code 2**阻止操作。

### 保护脚本（`.claude/hooks/protect-files.sh`）

```bash
#!/bin/bash
# protect-files.sh — 阻止对敏感文件的修改

INPUT=$(cat)
FILE_PATH=$(echo "$INPUT" | jq -r '.tool_input.file_path // empty')

PROTECTED_PATTERNS=(".env" "package-lock.json" ".git/")

for pattern in "${PROTECTED_PATTERNS[@]}"; do
  if [[ "$FILE_PATH" == *"$pattern"* ]]; then
    echo "Blocked: $FILE_PATH matches protected pattern '$pattern'" >&2
    exit 2  # exit code 2 = block the action
  fi
done

exit 0  # allow the action
```

**注意**：脚本必须可执行 —— `chmod +x .claude/hooks/protect-files.sh`

### 配置

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "\"$CLAUDE_PROJECT_DIR\"/.claude/hooks/protect-files.sh"
          }
        ]
      }
    ]
  }
}
```

### `if`字段进阶用法（v2.1.85+）

`if`字段使用permission rule语法，在matcher**已经匹配**后进一步精确过滤：

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "if": "Bash(git *)",
            "command": "\"$CLAUDE_PROJECT_DIR\"/.claude/hooks/check-git-policy.sh"
          }
        ]
      }
    ]
  }
}
```

只有Bash命令的子命令匹配`git *`时才spawn hook进程，避免不必要的开销。对于`npm test && git push`这样的复合命令，因为`git push`匹配，hook仍然触发。

`if`只支持工具事件：PreToolUse、PostToolUse、PostToolUseFailure、PermissionRequest、PermissionDenied。加到其他事件上hook永远不会执行。

### Exit code 2行为速查

| Hook event | 能否阻止？ | Exit code 2效果 |
|---|---|---|
| `PreToolUse` | Yes | **阻止**工具调用，stderr返回给Claude |
| `PermissionRequest` | Yes | 拒绝权限 |
| `UserPromptSubmit` | Yes | 阻止prompt处理并**擦除**prompt |
| `UserPromptExpansion` | Yes | 阻止展开 |
| `Stop` | Yes | 防止Claude停止，继续会话 |
| `SubagentStop` | Yes | 防止子Agent停止 |
| `TeammateIdle` | Yes | 防止队友空闲 |
| `TaskCreated` | Yes | 回滚任务创建 |
| `TaskCompleted` | Yes | 阻止标记为完成 |
| `ConfigChange` | Yes | 阻止配置变更生效（policy_settings除外） |
| `PreCompact` | Yes | 阻止压缩 |
| `Elicitation` | Yes | 拒绝elicitation |
| `PostToolUse` | **No** | 工具已执行，stderr返回给Claude |
| `PostCompact` | No | stderr仅展示给用户 |
| `SessionStart` | No | stderr仅展示给用户 |
| `SessionEnd` | No | 无法阻止会话终止 |
| `Notification` | No | stderr仅展示给用户 |
| `StopFailure` | No | 输出和exit code都被忽略 |

> **注意**：对于大多数事件，只有exit code 2阻止操作。exit code 1被视为非阻塞错误，操作继续执行——即便1是Unix惯例的失败码。如果你的hook是要强制执行策略，**必须**用`exit 2`。

---

## 5. 实战场景四：上下文自动注入 —— Re-inject Context After Compaction

**原理**：Claude上下文窗口满了时，压缩（compaction）会总结对话释放空间。`SessionStart`事件的`compact`matcher在压缩后触发——Hook的stdout内容会被加入到Claude的上下文中。

```json
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "compact",
        "hooks": [
          {
            "type": "command",
            "command": "echo 'Reminder: use Bun, not npm. Run bun test before committing. Current sprint: auth refactor.'"
          }
        ]
      }
    ]
  }
}
```

### 动态信息注入

可以用任何能产生动态输出的命令替换`echo`：

```json
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "compact",
        "hooks": [
          {
            "type": "command",
            "command": "echo \"Current branch: $(git rev-parse --abbrev-ref HEAD)\nRecent commits:\n$(git log --oneline -5)\""
          }
        ]
      }
    ]
  }
}
```

### JSON方式（用于更精细的控制）

```json
{
  "hookSpecificOutput": {
    "hookEventName": "SessionStart",
    "additionalContext": "Deployment target: production. CI status: green."
  }
}
```

### `additionalContext` 使用指南

| 方面 | 说明 |
|---|---|
| **适合注入什么** | 环境状态（当前分支、部署目标、活跃feature flag）、条件性项目规则、外部数据（未解决的issue、CI结果） |
| **写入风格** | 事实性陈述而非命令句式。如"The deployment target is production"而非"Remember the deployment target"。命令式陈述可能触发Claude的prompt注入防御 |
| **长度限制** | 最多10,000字符。超限后写入session目录文件，传递文件路径+预览给Claude |
| **持久性** | 注入文本保存在session transcript中。resume时重放保存的文本而非重跑Hook，所以时间戳/commit SHA等值在resume后可能是旧的 |
| **SessionStart例外** | `SessionStart` hook在resume时会重跑（`source`为`resume`），所以可以刷新上下文 |
| **事件差异** | SessionStart/Setup/SubagentStart：注入到对话开头。UserPromptSubmit/UserPromptExpansion：随prompt一起。PreToolUse/PostToolUse：随tool result一起 |
| **多Hook合并** | 多个Hook返回`additionalContext`时，Claude会收到**所有**值 |

> **对比**：静态不变的指令放在CLAUDE.md中（无运行开销），动态变化的上下文用`SessionStart` hook注入。环境变量用`CLAUDE_ENV_FILE`持久化。

---

## 6. 实战场景五：权限自动审批 —— Auto-approve Permission Prompts

**原理**：`PermissionRequest`事件在权限对话框显示前触发。Hook返回JSON决策自动批准或拒绝。

### 自动批准ExitPlanMode

```json
{
  "hooks": {
    "PermissionRequest": [
      {
        "matcher": "ExitPlanMode",
        "hooks": [
          {
            "type": "command",
            "command": "echo '{\"hookSpecificOutput\": {\"hookEventName\": \"PermissionRequest\", \"decision\": {\"behavior\": \"allow\"}}}'"
          }
        ]
      }
    ]
  }
}
```

效果：Claude完成plan后直接退出plan mode，不再询问你确认。Transcript中显示"Allowed by PermissionRequest hook"取代权限对话框。

### 批准并切换权限模式

```json
{
  "hookSpecificOutput": {
    "hookEventName": "PermissionRequest",
    "decision": {
      "behavior": "allow",
      "updatedPermissions": [
        { "type": "setMode", "mode": "acceptEdits", "destination": "session" }
      ]
    }
  }
}
```

### 权限更新条目类型

| type | 功能 |
|---|---|
| `addRules` | 添加权限规则 — `rules`数组 + `behavior` (allow/deny/ask) |
| `replaceRules` | 替换指定行为的全部规则 |
| `removeRules` | 移除匹配规则 |
| `setMode` | 更改权限模式 — `default`/`acceptEdits`/`dontAsk`/`bypassPermissions`/`plan` |
| `addDirectories` | 添加工作目录 |
| `removeDirectories` | 移除工作目录 |

`destination`决定变更范围：

| destination | 写入到 |
|---|---|
| `session` | 仅内存，会话结束丢弃 |
| `localSettings` | `.claude/settings.local.json` |
| `projectSettings` | `.claude/settings.json` |
| `userSettings` | `~/.claude/settings.json` |

> **安全提示**：
> - matcher尽量具体，不要用`""`或`.*`匹配所有——这会自动批准所有权限请求，包括文件写入和shell命令
> - Hook返回`"allow"`不会覆盖deny规则——deny规则始终优先
> - `bypassPermissions` mode需要session启动时就已可用（`--dangerously-skip-permissions`等），且永远不会被持久化为`defaultMode`

---

## 7. 实战场景六：任务完成通知 —— Session End Cleanup & Notification

**原理**：`SessionEnd`事件在会话结束时触发，用于清理、记录。

### 清理临时文件

```json
{
  "hooks": {
    "SessionEnd": [
      {
        "matcher": "clear",
        "hooks": [
          {
            "type": "command",
            "command": "rm -f /tmp/claude-scratch-*.txt"
          }
        ]
      }
    ]
  }
}
```

### 会话日志审计

```json
{
  "hooks": {
    "SessionEnd": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "jq -c '{timestamp: now | todate, reason: .reason}' >> ~/claude-session-log.jsonl"
          }
        ]
      }
    ]
  }
}
```

### SessionEnd支持的理由matcher

| 值 | 说明 |
|---|---|
| `clear` | `/clear`清空会话 |
| `resume` | 通过`/resume`切换会话 |
| `logout` | 用户登出 |
| `prompt_input_exit` | 用户在prompt输入中退出 |
| `bypass_permissions_disabled` | Bypass权限模式被禁用 |
| `other` | 其他退出原因 |

> **注意**：SessionEnd hook默认超时1.5秒。如果Hook需要更多时间，设置per-hook `timeout`。总预算自动提升到所有settings文件中配置的最大per-hook timeout，上限60秒。可通过`CLAUDE_CODE_SESSIONEND_HOOKS_TIMEOUT_MS`环境变量覆盖：
> ```bash
> CLAUDE_CODE_SESSIONEND_HOOKS_TIMEOUT_MS=5000 claude
> ```

---

## 8. Hook Type 对比 —— Command vs Prompt vs Agent vs HTTP vs MCP Tool

Claude Code支持五种Hook类型，各有适用场景：

| Type | 执行方式 | 适用场景 | 默认超时 | 能否阻止操作 |
|---|---|---|---|---|
| **command** | Shell命令 | 确定性逻辑、文件操作、通知 | 600s | Yes (exit code 2) |
| **http** | HTTP POST | 调用外部API、团队共享审计服务 | 600s | Yes (JSON body) |
| **mcp_tool** | MCP服务器工具 | 调用已连接的MCP工具 | 600s | Yes |
| **prompt** | LLM单轮评估 | 需要判断力的决策 | 30s | Yes (`ok: false`) |
| **agent** | 多轮子Agent验证 | 需要检查实际代码/文件状态 | 60s | Yes (`ok: false`) |

### Command Hook —— 最常用、最可靠

```json
{
  "type": "command",
  "command": "jq -r '.tool_input.file_path' | xargs npx prettier --write",
  "timeout": 30
}
```

stdin → JSON输入 | exit code → 决策信号 | stdout/stderr → 信息传递

**Exec form vs Shell form**：添加`"args": []`使用exec form（直接spawn，无shell），避免shell引用问题：
```json
{
  "type": "command",
  "command": "${CLAUDE_PROJECT_DIR}/.claude/hooks/check-style.sh",
  "args": []
}
```

### Prompt-based Hook —— 需要AI判断的场景

```json
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "prompt",
            "prompt": "Evaluate if Claude should stop: $ARGUMENTS. Check if all tasks are complete.",
            "timeout": 30
          }
        ]
      }
    ]
  }
}
```

**模型配置**：
- 默认使用Haiku（快速）
- 可通过`"model"`字段指定其他模型
- `$ARGUMENTS`占位符会被Hook的JSON input替换
- 返回格式：`{"ok": true/false, "reason": "..."}`

**`ok: false`在不同事件上的行为**：

| 事件 | `ok: false`效果 |
|---|---|
| `Stop` / `SubagentStop` | reason返回给Claude作为下一条指令，继续执行 |
| `PreToolUse` | 拒绝工具调用，reason作为工具错误返回给Claude |
| `PostToolUse` | 默认结束turn + reason显示为警告行。可设置`continueOnBlock: true`让Claude继续 |
| `PostToolBatch` / `UserPromptSubmit` / `UserPromptExpansion` | 结束turn，reason显示为警告行 |
| `PermissionRequest` | `ok: false`无效果——需要用command hook返回`decision.behavior: "deny"` |

### Agent-based Hook —— 实验性

```json
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "agent",
            "prompt": "Verify that all unit tests pass. Run the test suite and check the results. $ARGUMENTS",
            "timeout": 120
          }
        ]
      }
    ]
  }
}
```

**特点**：spawn子Agent，可以调用Read、Grep、Glob等工具，最多50轮tool-use，适合需要实际验证代码库状态的场景。

### HTTP Hook

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "hooks": [
          {
            "type": "http",
            "url": "http://localhost:8080/hooks/tool-use",
            "headers": {
              "Authorization": "Bearer $MY_TOKEN"
            },
            "allowedEnvVars": ["MY_TOKEN"]
          }
        ]
      }
    ]
  }
}
```

**特点**：POST JSON到HTTP endpoint，通过response body返回决策。仅2xx + JSON body可以阻止操作——HTTP status code不能阻止。

### MCP Tool Hook

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "mcp_tool",
            "server": "my_server",
            "tool": "security_scan",
            "input": { "file_path": "${tool_input.file_path}" }
          }
        ]
      }
    ]
  }
}
```

**特点**：调用已连接的MCP服务器上的工具。如果服务器未连接或工具返回`isError: true`，产生非阻塞错误，执行继续。SessionStart/Setup通常在MCP连接前触发，这些事件的MCP tool hook可能遇到"not connected"错误。

### 选择建议

> - 能用command hook搞定的事，**优先用command hook**（最可靠、最快）
> - 需要AI判断但不需要访问文件 → **prompt hook**
> - 需要AI判断且需要访问文件系统 → **agent hook**（实验性，可能变化）
> - 需要调用外部服务 → **HTTP hook**
> - 需要调用已连接的MCP工具 → **mcp_tool hook**

### 支持矩阵

**支持全部5种类型**：`PermissionRequest`、`PostToolBatch`、`PostToolUse`、`PostToolUseFailure`、`PreToolUse`、`Stop`、`SubagentStop`、`TaskCompleted`、`TaskCreated`、`UserPromptExpansion`、`UserPromptSubmit`

**支持command/http/mcp_tool，不支持prompt/agent**：`ConfigChange`、`CwdChanged`、`Elicitation`、`ElicitationResult`、`FileChanged`、`InstructionsLoaded`、`Notification`、`PermissionDenied`、`PostCompact`、`PreCompact`、`SessionEnd`、`StopFailure`、`SubagentStart`、`TeammateIdle`、`WorktreeCreate`、`WorktreeRemove`

**仅支持command/mcp_tool**：`SessionStart`、`Setup`

---

## 9. 异步Hook —— Async Hooks

**原理**：默认Hook**阻塞**Claude的执行直到完成。设置`"async": true`后，Hook在后台运行，Claude立即继续工作。

### 配置异步Hook

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "\"$CLAUDE_PROJECT_DIR\"/.claude/hooks/run-tests-async.sh",
            "args": [],
            "async": true,
            "timeout": 300
          }
        ]
      }
    ]
  }
}
```

### 异步Hook脚本示例（`.claude/hooks/run-tests-async.sh`）

```bash
#!/bin/bash
# run-tests-async.sh

INPUT=$(cat)
FILE_PATH=$(echo "$INPUT" | jq -r '.tool_input.file_path // empty')

# Only run tests for source files
if [[ "$FILE_PATH" != *.ts && "$FILE_PATH" != *.js ]]; then
  exit 0
fi

# Run tests and report results to Claude via additionalContext
RESULT=$(npm test 2>&1)
EXIT_CODE=$?

if [ $EXIT_CODE -eq 0 ]; then
  MSG="Tests passed after editing $FILE_PATH"
else
  MSG="Tests failed after editing $FILE_PATH: $RESULT"
fi
jq -nc --arg msg "$MSG" '{hookSpecificOutput: {hookEventName: "PostToolUse", additionalContext: $msg}}'
```

### asyncRewake —— 后台失败时唤醒Claude

`asyncRewake: true`隐含`async: true`，且当进程exit code为2时立即唤醒Claude：

- Hook的stderr（或stdout若stderr为空）作为system reminder传递给Claude
- 即使session空闲也会立即唤醒
- 适合部署监控、长期测试观察等场景

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "./watch-deployment.sh",
            "asyncRewake": true
          }
        ]
      }
    ]
  }
}
```

### 异步Hook的限制

- 仅`type: "command"`支持`async`
- **无法阻止操作或返回决策**——hook完成时触发动作已经执行
- 输出在**下一个conversation turn**交付。如果session空闲，等待下次用户交互才交付
- 每次触发创建一个独立后台进程，**不会去重**

---

## 10. Hook调试与常见坑 —— Debugging & Common Pitfalls

### 调试方法

| 方法 | 操作 | 能看什么 |
|---|---|---|
| `/hooks` 命令 | Claude Code中直接输入 | 所有配置的Hook列表（事件、matcher、类型、来源文件） |
| Transcript视图 | `Ctrl+O`切换 | 每个Hook的单行摘要（成功/阻止/错误） |
| Debug日志 | `claude --debug-file /tmp/claude.log` | 完整执行细节：哪个Hook匹配、exit code、stdout、stderr |
| mid-session调试 | 运行`/debug` | 找到log路径，后续tail观察 |
| 详细模式 | `CLAUDE_CODE_DEBUG_LOG_LEVEL=verbose` | hook matcher计数、query匹配等额外日志 |

### 常见坑

#### (1) JSON输出格式被shell profile破坏

```bash
# 问题：shell profile输出了额外内容
echo "Shell ready on arm64"  # 这会破坏JSON解析
echo '{"decision": "block"}'
```

**解决**：shell profile中的echo需要加interactive判断：
```bash
if [[ $- == *i* ]]; then
  echo "Shell ready"
fi
```
Hook运行在non-interactive shell中，所以echo被跳过。

#### (2) Exit code 2 vs 1 混淆

- Exit code 2 = 阻止操作（stdout的JSON被忽略，stderr返回给Claude）
- Exit code 1 = **非阻塞错误**（操作继续执行，transcript显示"hook error"）
- Exit code 0 = 成功（stdout的JSON被解析处理）
- **如果要阻止操作，必须用exit 2**

#### (3) matcher失效排查

- matcher**区分大小写**
- 纯字母数字+下划线+`|`的matcher做精确字符串匹配，不是正则
- 要使用正则，matcher中必须包含非字母数字字符（如`mcp__.*`）
- MCP工具的命名模式：`mcp__<server>__<tool>`，如`mcp__github__search_repositories`
- 匹配MCP工具时`.*`是必需的：`"matcher": "mcp__memory"`作为一个精确字符串不会匹配任何工具

#### (4) Stop Hook无限循环

Stop Hook每次Claude完成回复时触发。如果总是返回`decision: "block"`，形成无限循环。

**解决**：检查输入的`stop_hook_active`字段：
```bash
#!/bin/bash
INPUT=$(cat)
if [ "$(echo "$INPUT" | jq -r '.stop_hook_active')" = "true" ]; then
  exit 0  # Allow Claude to stop
fi
# ... rest of your hook logic
```

#### (5) SessionStart Hook超时问题

SessionStart在每个会话上都执行，必须保持快速。不要在SessionStart中运行耗时操作。

#### (6) 多Hook决策合并规则

多个同事件Hook**并行执行**。对于PreToolUse权限决策：**deny > defer > ask > allow**（最严格的胜出）。一个Hook返回deny**不会阻止其他Hook继续执行**——所有Hook都完成后才合并结果。

当多个PreToolUse Hook返回`updatedInput`修改工具参数时，**最后一个完成的胜出**。由于并行执行顺序不确定，避免多个Hook修改同一个工具的输入。

#### (7) "hook not firing"（Hook不触发）

1. 运行`/hooks`确认Hook出现在正确的事件下
2. 检查matcher是否精确匹配工具名（区分大小写）
3. 确认触发了正确的事件类型（PreToolUse vs PostToolUse）
4. 若用`PermissionRequest` + `-p`非交互模式：**PermissionRequest在非交互模式下不会触发**，改用`PreToolUse`
5. 如果是settings文件手动编辑，文件watcher可能漏掉了变更——重启session强制重载
6. 检查JSON是否有效（trailing commas和comments不允许）

#### (8) 文件路径引用问题

```bash
# 错误：路径中有空格导致shell解析错误
"command": "$CLAUDE_PROJECT_DIR/.claude/hooks/my script.sh"

# 正确：shell form用双引号包裹
"command": "\"$CLAUDE_PROJECT_DIR\"/.claude/hooks/my-script.sh"

# 最佳：exec form不需要引号
"command": "${CLAUDE_PROJECT_DIR}/.claude/hooks/my-script.sh",
"args": []
```

---

## 总结与实践要点

### 6大实战场景速查

| 场景 | 使用事件 | Hook类型 | 关键配置 |
|---|---|---|---|
| 代码格式化 | PostToolUse | command | matcher: `Edit\|Write` |
| 桌面通知 | Notification | command | matcher按通知类型 |
| 敏感文件保护 | PreToolUse | command | exit code 2 |
| 上下文注入 | SessionStart | command | matcher: `compact` |
| 权限审批 | PermissionRequest | command | JSON decision body |
| 会话清理 | SessionEnd | command | matcher按退出原因 |

### 扩展场景速查

| 场景 | 使用事件 | Hook类型 | 关键配置 |
|---|---|---|---|
| 任务完成验证 | Stop | prompt/agent | `ok: false` + reason |
| 后台测试 | PostToolUse | async command | `async: true` |
| 配置变更审计 | ConfigChange | command | `decision: "block"` |
| 环境变量重载 | CwdChanged | command | `CLAUDE_ENV_FILE` |
| 命令日志记录 | PostToolUse | command | matcher: `Bash` |
| 子Agent上下文 | SubagentStart | command | `additionalContext` |

### 配置位置选择

| 位置 | 作用域 | 可分享 |
|---|---|---|
| `~/.claude/settings.json` | 所有项目 | 否（本机） |
| `.claude/settings.json` | 单个项目 | 是（可提交到仓库） |
| `.claude/settings.local.json` | 单个项目 | 否（gitignored） |
| Managed policy settings | 组织范围 | 是（管理员控制） |
| Plugin `hooks/hooks.json` | 插件启用时 | 是（与插件捆绑） |
| Skill/Agent frontmatter | 组件活动期间 | 是（定义在组件文件中） |

> **最佳实践**：先在`~/.claude/settings.json`中测试Hook，确认无误后移入项目`.claude/settings.json`。项目级配置可以提交到仓库供团队共享。使用`/hooks`浏览所有配置的Hook验证是否正确注册。

### 安全提醒

- Command hook以你的完整用户权限运行——可以修改/删除任何你有权限访问的文件
- 始终使用`"$VAR"`而非`$VAR`引用变量
- 检查路径遍历攻击（file path中的`..`）
- 使用绝对路径引用脚本文件
- 避开敏感文件（`.env`、`.git/`、keys等）

---

## 学习笔记（留空）

### 关键概念理解

-
-
-

### 遇到的实际问题

-
-
-

### 额外想了解的内容

-
-
-
