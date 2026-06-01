---
title: 【实践5】Common Workflows：探索→规划→实现→验证的日常流
publish: true
---

# 【实践5】Common Workflows：探索→规划→实现→验证的日常流

> 覆盖日常开发中最常见的任务模式：理解新代码库、Bug修复、重构、测试、PR工作流，以及非代码场景。

**参考文档**：
- `code.claude.com/docs/en/common-workflows.md` — Step-by-step guides
- `code.claude.com/docs/en/how-claude-code-works.md` — How Claude Code works
- `code.claude.com/docs/en/headless.md` — Run Claude Code programmatically

**先修知识**：[[【4】Subagents深度指南]]、[[【6】Skills与Commands]]

**学习目标**：掌握日常任务模式、学会高效引用文件和目录、了解管道与自动化输入

---

## 1. 理解新代码库（快速概览、定位关键文件、/context策略）

当你加入一个新项目或需要理解不熟悉的代码库时，Claude Code 可以帮助你快速建立整体认知。

### 快速概览

在项目根目录启动 Claude Code，直接提问：

```text
give me an overview of this codebase
```

建议从宽泛问题开始，逐步深入到具体模块：

```text
explain the main architecture patterns used here
what are the key data models?
how is authentication handled?
```

### 定位关键文件

当需要找到特定功能对应的文件时：

```text
find the files that handle user authentication
how do these authentication files work together?
trace the login process from front-end to database
```

### Tips
- 先从高层次结构入手，再深入到具体模块
- 让 Claude 解释项目中的编码约定和模式
- 请求生成项目特有术语表（glossary）
- 为语言安装代码智能插件（code intelligence plugin），让 Claude 能使用 "go to definition" 和 "find references"

### /context 策略

使用 `/context` 命令可以查看当前上下文窗口的使用情况，了解哪些内容占用了空间。随着对话进行，上下文会被自动压缩（compact），早期对话中的指令可能丢失——因此持久性规则应当放在 `CLAUDE.md` 中，而非依赖对话历史。

---

## 2. Bug修复流（定位→分析→修复→验证四步法）

Bug 修复是 Claude Code 最擅长的场景之一。推荐的四步流程：

### Step 1: 定位（Locate）

将错误信息或描述提供给 Claude：

```text
I'm seeing an error when I run npm test
```

Tips：
- 告诉 Claude 重现问题的命令，以获取完整的 stack trace
- 说明重现步骤
- 告知问题是间歇性还是持续性的

### Step 2: 分析（Analyze）

Claude 会读取相关文件，分析错误根源：

```text
suggest a few ways to fix the @ts-ignore in user.ts
```

### Step 3: 修复（Fix）

批准 Claude 的修复建议或让其直接应用：

```text
update user.ts to add the null check you suggested
```

### Step 4: 验证（Verify）

修复后让 Claude 运行测试来确认：

```text
run the tests to verify the fix
```

> **关键原则**：Claude 的代理循环（agentic loop）会自动执行定位→分析→修复→验证。只需要给出初始描述，Claude 会决定每一步需要做什么。

---

## 3. 重构流（安全重构的步骤、checkpoint保护、review验证）

重构旧代码时，安全是最重要的考量。

### 识别需要重构的代码

```text
find deprecated API usage in our codebase
```

### 获取重构建议

```text
suggest how to refactor utils.js to use modern JavaScript features
```

### 应用变更

```text
refactor utils.js to use ES2024 features while maintaining the same behavior
```

### 验证重构

```text
run tests for the refactored code
```

### Checkpoint 保护机制

Claude Code 为每次文件编辑创建快照（checkpoint）。如果出现问题：
- 按 `Esc` 两次回退到之前的状态
- 或者直接告诉 Claude "undo that change"

Checkpoint 是独立于 git 的会话级机制，仅覆盖文件变更。影响远程系统（数据库、API、部署）的操作无法被 checkpoint 保护。

### 安全重构 Tips
- 让 Claude 解释现代方法的好处
- 需要在保持向后兼容（backward compatibility）时明确说明
- 以小而可验证的增量进行重构（small, testable increments）
- 重构后务必运行测试套件
- 使用 plan mode（Shift+Tab 切换）先审阅方案再执行

---

## 4. 测试流（写测试、跑测试、修测试的循环）

Claude Code 可以分析测试覆盖、生成测试脚手架、添加有意义的测试用例。

### 定位未覆盖代码

```text
find functions in NotificationsService.swift that are not covered by tests
```

### 生成测试脚手架

```text
add tests for the notification service
```

### 添加有意义的测试用例

```text
add test cases for edge conditions in the notification service
```

### 运行并验证

```text
run the new tests and fix any failures
```

### Claude Code 的测试能力

- Claude 会自动分析项目现有的测试文件，匹配已有的风格、框架和断言模式
- 可以识别你可能遗漏的边缘情况（error conditions, boundary values, unexpected inputs）
- 测试流完美体现了 agentic loop 的多轮交互模式：Claude 会运行测试看到失败 → 读取出错输出 → 搜索源文件 → 编辑修复 → 再次验证

---

## 5. PR工作流（从分支到PR、写commit message、创建PR）

Claude Code 可以帮助从变更到创建 PR 的完整流程。

### 步骤一：总结变更

```text
summarize the changes I've made to the authentication module
```

### 步骤二：生成 PR

```text
create a pr
```

### 步骤三：审查和优化

```text
enhance the PR description with more context about the security improvements
```

### 注意事项

- 创建 PR 后，会话会自动链接到该 PR
- 之后可以通过 `claude --from-pr <URL>` 或粘贴 PR URL 到 `/resume` 选择器返回到该 PR
- 在提交前应审阅 Claude 生成的 PR 描述，并让 Claude 标注潜在风险
- 仅使用 `-p` 模式时，用户调用的技能（如 `/commit`）不可用，需要直接描述你想要完成的任务

### Commit 消息与管道模式

在非交互模式（`-p`）下创建 commit：

```bash
claude -p "Look at my staged changes and create an appropriate commit" \
  --allowedTools "Bash(git diff *),Bash(git log *),Bash(git status *),Bash(git commit *)"
```

`--allowedTools` 的 `*` 后缀启用了前缀匹配，例如 `Bash(git diff *)` 允许所有以 `git diff` 开头的命令。

---

## 6. 非代码场景（写文档、处理Markdown笔记、处理图片）

Claude Code 不仅限于代码——它在任何目录下都能工作。

### 识别未文档化的代码

```text
find functions without proper JSDoc comments in the auth module
```

### 生成文档

```text
add JSDoc comments to the undocumented functions in auth.js
```

### 审查和增强

```text
improve the generated documentation with more context and examples
```

### 验证文档风格

```text
check if the documentation follows our project standards
```

### 处理 Markdown 笔记

Claude Code 可以直接在笔记库（Obsidian vault）、文档文件夹或任何 Markdown 文件集合中工作——搜索、编辑和重新组织内容，与处理代码的方式完全相同。

`.claude/` 目录和 `CLAUDE.md` 可以与其他工具的配置目录并存，不会冲突。Claude 在每次工具调用时重新读取文件，因此你在其他应用中做的编辑在下一次读取时就会被看到。

### 处理图片

支持三种方式添加图片到对话：
1. 拖放图片到 Claude Code 窗口
2. 复制图片后 `Ctrl+V` 粘贴到 CLI（注意是 Ctrl+V 不是 Cmd+V）
3. 提供图片路径：`Analyze this image: /path/to/image.png`

常用场景：
```text
What does this image show?
Describe the UI elements in this screenshot
Here's a screenshot of the error. What's causing it?
Generate CSS to match this design mockup
```

当 Claude 引用图片时（例如 `[Image #1]`），`Cmd+Click`（Mac）或 `Ctrl+Click`（Windows/Linux）链接可以在默认图片查看器中打开该图片。

---

## 7. 引用文件与目录的实用技巧（@-mentions、glob patterns）

使用 `@` 符号可以快速引用文件或目录，无需等待 Claude 读取。

### 引用单个文件

```text
Explain the logic in @src/utils/auth.js
```

这会将该文件的完整内容加入对话。

### 引用目录

```text
What's the structure of @src/components?
```

这会提供目录列表和文件信息。

### 引用 MCP 资源

```text
Show me the data from @github:repos/owner/repo/issues
```

这会从已连接的 MCP 服务器获取数据。格式为 `@server:resource`。

### @-mention 的关键行为

- 文件路径可以是相对路径或绝对路径
- `@` 文件引用会将被引用文件所在目录及其父目录的 `CLAUDE.md` 加入上下文
- 目录引用显示文件列表而非文件内容
- 可以在单条消息中引用多个文件：`@file1.js and @file2.js`

### Glob Patterns

在搜索文件时，可以使用通配符模式。Claude 会使用内置的搜索工具来匹配文件模式，例如 `find files matching *.test.ts` 或 `search for files containing "TODO"`。

---

## 8. Resume、管道输入、定时运行（/resume、stdin、/loop）

### Resume 会话

当任务需要分多次完成时，可以恢复之前的对话：

```bash
# 恢复当前目录下最近的会话
claude --continue

# 从列表中选择会话
claude --resume

# 在运行中使用
# 输入 /resume 并选择
```

`/resume` 选择器默认显示当前工作树的会话，可以使用快捷键扩大范围到其他工作树或项目。

### 管道输入（Pipe Claude into scripts）

Claude Code 支持非交互模式（`-p`/`--print`），可以像 Unix 工具一样使用标准输入输出：

```bash
# 管道输入
git log --oneline -20 | claude -p "summarize these recent commits"

# 将输出重定向到文件
cat build-error.txt | claude -p 'concisely explain the root cause of this build error' > output.txt

# 获取 JSON 格式的结构化输出
claude -p "Summarize this project" --output-format json

# 使用 JSON Schema 约束输出
claude -p "Extract the main function names from auth.py" \
  --output-format json \
  --json-schema '{"type":"object","properties":{"functions":{"type":"array","items":{"type":"string"}}},"required":["functions"]}'
```

> **注意**：截至 Claude Code v2.1.128，管道输入上限为 10MB。超限会报错并退出（非零退出码）。处理大输入时，建议将内容写入文件，然后在 prompt 中引用文件路径。

### 在 CI 中使用

将 Claude Code 集成到 CI 流程中，例如 `package.json` 脚本：

```json
{
  "scripts": {
    "lint:claude": "git diff main | claude -p \"you are a typo linter. for each typo in this diff, report filename:line on one line and the issue on the next. return nothing else.\""
  }
}
```

使用 `jq` 解析 JSON 输出中的特定字段：

```bash
# 提取文本结果
claude -p "Summarize this project" --output-format json | jq -r '.result'

# 提取结构化输出
claude -p "Extract function names from auth.py" \
  --output-format json \
  --json-schema '{"type":"object","properties":{"functions":{"type":"array","items":{"type":"string"}}},"required":["functions"]}' \
  | jq '.structured_output'
```

### 定时运行 Scheduled Tasks

Claude Code 支持多种定时运行方式：

| 选项 | 运行位置 | 最佳用途 |
|------|----------|----------|
| Routines | Anthropic 管理的基础设施 | 任务需要即使电脑关闭也运行；可配置 API 调用或 GitHub 事件触发 |
| 桌面定时任务 | 本地机器（桌面应用） | 需要直接访问本地文件、工具或未提交变更 |
| GitHub Actions | CI 管道 | 与仓库事件绑定（如 PR 打开），或 cron 定时任务 |
| `/loop` | 当前 CLI 会话 | 会话打开时的快速轮询；开始新对话后停止 |

编写定时任务 prompt 时，要明确说明成功标准和处理结果的方式——因为任务自动运行，无法提问澄清。

### 非交互模式的关键选项

| 功能 | 标志 |
|------|------|
| 减少启动时间 | `--bare`（跳过 hooks/skills/MCP 的自动发现） |
| 追加系统提示 | `--append-system-prompt "instructions"` |
| 完全替换系统提示 | `--system-prompt "instructions"` |
| 自动批准工具 | `--allowedTools "Bash,Read,Edit"` |
| 权限模式 | `--permission-mode acceptEdits` |
| 输出格式 | `--output-format text\|json\|stream-json` |
| 继续会话 | `--continue` 或 `--resume <session_id>` |

### Bare Mode 说明

`--bare` 模式适用于 CI 和脚本，它：
- 跳过 hooks、skills、plugins、MCP servers、auto memory 和 CLAUDE.md 的自动加载
- 不加载 CLAUDE.md——只有通过 flag 显式传入的上下文才生效
- 跳过 OAuth 和 keychain 读取
- 需要 `ANTHROPIC_API_KEY` 或 provider 凭证
- 是 `-p` 模式推荐的方式（未来版本将成为默认）

### 流式输出 Stream JSON

使用 `--output-format stream-json` 配合 `--verbose` 和 `--include-partial-messages` 可以实时接收 token：

```bash
claude -p "Explain recursion" --output-format stream-json --verbose --include-partial-messages

# 使用 jq 过滤文本 delta，实现连续流式输出
claude -p "Write a poem" --output-format stream-json --verbose --include-partial-messages | \
  jq -rj 'select(.type == "stream_event" and .event.delta.type? == "text_delta") | .event.delta.text'
```

流事件中包含 `system/api_retry` 事件（API 重试时发出）和 `system/init` 事件（报告 session metadata，包括模型、工具、MCP servers、加载的插件）。

---

> **参考资料**：
> - [Common Workflows — code.claude.com](https://code.claude.com/docs/en/common-workflows.md)
> - [How Claude Code Works — code.claude.com](https://code.claude.com/docs/en/how-claude-code-works.md)
> - [Headless Mode — code.claude.com](https://code.claude.com/docs/en/headless.md)

---

## 笔记区

### 关键概念速查

| 概念 | 说明 |
|------|------|
| Agentic Loop | Claude 的"收集上下文→采取行动→验证结果"循环，自动决定每一步需要的工具 |
| Checkpoint | 每次文件编辑前的自动快照，`Esc+Esc` 回退，独立于 git |
| Plan Mode | 只读模式，审阅方案后再执行（`Shift+Tab` 两次进入） |
| Compact | 上下文窗口自动压缩，保留关键信息但可能丢失早期指令 |
| Bare Mode | `--bare`，跳过所有自动发现，适用于 CI/脚本，需要手动传入所有上下文 |
| @-mention | 用 `@` 引用文件或目录，快速加入上下文 |
| -p / --print | 非交互模式，通过 stdin/stdout 管道化使用 |

### 实践记录

<!-- 在此记录你的实践过程、遇到的问题和心得 -->

### 相关主题链接

- [[【4】Subagents深度指南]] — 子代理可以独立探索代码库，避免主上下文被大量文件读取占用
- [[【6】Skills与Commands]] — Skills 和 Commands 可以封装常用工作流为可复用命令
- [[【3】Checkpointing与会话管理]] — 深挖 checkpoint 回退和会话 fork/branch 机制
- [[【7】Worktrees]] — 用 git worktree 实现真正的并行会话
