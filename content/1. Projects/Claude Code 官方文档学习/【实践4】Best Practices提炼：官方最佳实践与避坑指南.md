---
title: 【实践4】Best Practices提炼：官方最佳实践与避坑指南
publish: true
---

# 【实践4】Best Practices 提炼：官方最佳实践与避坑指南

> 将官方文档中分散的最佳实践提炼为可操作的检查清单——验证手段、上下文管理、避免常见失败模式。

**参考文档**：
- `code.claude.com/docs/en/best-practices.md` — Best practices for Claude Code
- `code.claude.com/docs/en/costs.md` — Manage costs effectively

**先修知识**：[[【10】开发工作流总览]]

**学习目标**：掌握 test → verify 循环、理解三段论流程、学会 prompt 技巧、了解成本控制

---

## 1. 给 Claude 验证手段：test → verify 循环

> **核心原则**：Include tests, screenshots, or expected outputs so Claude can check itself. This is the single highest-leverage thing you can do.

Claude 在能够自我验证时表现会显著更好。没有清晰的 success criteria，它可能写出看起来正确但实际上不工作的代码——你就成了唯一的反馈回路。

### 常见策略对照

| 策略 | 错误示例 | 正确示例 |
|------|---------|---------|
| 提供验证标准 | "implement a function that validates email addresses" | "write a validateEmail function. example test cases: user@example.com is true, invalid is false, user@.com is false. run the tests after implementing" |
| UI 视觉验证 | "make the dashboard look better" | "[paste screenshot] implement this design. take a screenshot of the result and compare it to the original. list differences and fix them" |
| 根治问题而非症状 | "the build is failing" | "the build fails with this error: [paste error]. fix it and verify the build succeeds. address the root cause, don't suppress the error" |

### 验证手段可以是

- 测试套件（test suite）
- Linter
- Bash 命令检查输出
- UI 截图对比（配合 Claude in Chrome extension）
- 任何能回答"改对了没有"的手段

> **Key Insight**：If you can't verify it, don't ship it.

### 个人笔记空间

```
在此记录你的验证策略实践：
- 哪些场景适合写测试用例验证？
- 哪些适合截图对比？
- 你常用的验证命令是什么？
```

---

## 2. 探索 → 规划 → 编码三段论

> **核心原则**：Separate research and planning from implementation to avoid solving the wrong problem.

完整的推荐流程分为四个阶段，但**不是所有场景都需要**。

### 四个阶段

#### (1) Explore（探索）

进入 plan mode，让 Claude 读取文件、回答问题而不做修改。

```txt
read /src/auth and understand how we handle sessions and login.
also look at how we manage environment variables for secrets.
```

#### (2) Plan（规划）

让 Claude 制定详细的实现计划。

```txt
I want to add Google OAuth. What files need to change?
What's the session flow? Create a plan.
```

> 按下 `Ctrl+G` 可在文本编辑器中直接编辑计划，然后再让 Claude 执行。

#### (3) Implement（实现）

退出 plan mode，让 Claude 按照计划编码，并对照计划验证。

```txt
implement the OAuth flow from your plan. write tests for the
callback handler, run the test suite and fix any failures.
```

#### (4) Commit（提交）

```txt
commit with a descriptive message and open a PR
```

### 何时跳过规划

Plan mode 有开销。对于以下场景可以跳过：
- 修复 typo
- 添加一行 log
- 重命名变量
- 任何你能一句话描述出 diff 的任务

Planning is most useful when：不确定方案、改动涉及多文件、不熟悉被修改的代码。

### 常见错误

| 错误 | 表现 | 修正 |
|------|------|------|
| 跳过探索直接编码 | 产生解决问题的代码，但解决的是错误的问题 | 先进入 plan mode |
| 过度规划 | 小改动也走完整流程，浪费 token | 判断 scope，一句话能说清的跳过 |
| 规划后不验证 | 计划完美但实现偏离 | Implement 阶段要反查 plan |

### 个人笔记空间

```
记录你使用三段论的经验：
- 什么场景下你发现跳过探索会导致走弯路？
- 你的"一句话判断标准"是什么？
```

---

## 3. Prompt 实战技巧

> **核心原则**：The more precise your instructions, the fewer corrections you'll need.

### 四大策略

| 策略 | 错误示例 | 正确示例 |
|------|---------|---------|
| 限定范围 | "add tests for foo.py" | "write a test for foo.py covering the edge case where the user is logged out. avoid mocks." |
| 指向来源 | "why does ExecutionFactory have such a weird api?" | "look through ExecutionFactory's git history and summarize how its api came to be" |
| 引用已有模式 | "add a calendar widget" | "look at how existing widgets are implemented on the home page... HotDogWidget.php is a good example. follow the pattern..." |
| 描述症状 | "fix the login bug" | "users report that login fails after session timeout. check the auth flow in src/auth/, especially token refresh. write a failing test that reproduces the issue, then fix it" |

> 但注意：vague prompts 在探索阶段也有价值——"what would you improve in this file?" 能发现你没想到的问题。

### 提供富内容的手段

| 手段 | 用途 | 说明 |
|------|------|------|
| `@` 引用文件 | 精确指向代码位置 | 比口述路径更可靠 |
| 粘贴图片 | 视觉对比 / UI 参考 | 复制粘贴或拖拽 |
| 提供 URL | 文档 / API 引用 | 用 `/permissions` 放行常用域名 |
| 管道传数据 | 错误日志 / 分析 | `cat error.log \| claude` |
| 让 Claude 自己抓 | 灵活探索 | Bash / MCP / 读文件 |

### 个人笔记空间

```
你常用的 prompt 模板：
1. 新功能类：
2. Bug 修复类：
3. 代码审查类：
4. 探索学习类：
```

---

## 4. 会话管理最佳实践

> **核心原则**：Conversations are persistent and reversible. Use this to your advantage!

### 关键操作速查

| 操作 | 作用 | 使用时机 |
|------|------|---------|
| `Esc` | 停止 Claude 当前操作，保留上下文 | 发现走偏时立刻打断 |
| `Esc + Esc` / `/rewind` | 打开回退菜单 | 需要恢复到之前某个 checkpoint |
| `"Undo that"` | 让 Claude 撤销更改 | 改错了但不想回退太多 |
| `/clear` | 清空上下文 | 切换到不相关任务时 |
| `/compact` | 压缩会话历史 | 上下文接近上限时精细控制 |
| `/rename` | 命名会话 | 方便后续 resume |

### 上下文管理原则

- **及时纠偏**：发现 Claude 走偏就立刻打断。越快越好。
- **两次原则**：同一个问题纠正超过两次，直接 `/clear` 换一个更精确的 prompt 重新开始。脏上下文中的纠正往往不如干净的 prompt 效果好。
- **自动 compact**：Claude 会在接近上下文限制时自动压缩，保留关键信息。
- **自定义 compact**：`/compact Focus on the API changes` 指定压缩时应保留的重点。
- **在 CLAUDE.md 中预设 compact 说明**：

```markdown
# Compact instructions
When you are using compact, please focus on test output and code changes
```

- **`/btw` 快速提问**：答案出现在可关闭的浮层中，不会进入对话历史，适合查个细节又不增长上下文。
- **Subagent 做调查**：Subagent 在独立上下文中运行，不污染主会话。

### Checkpoint 机制

- 每条 prompt 都会创建一个 checkpoint
- 可以恢复：仅对话 / 仅代码 / 两者都恢复
- Checkpoint 跨 session 持久化（关终端后仍可用）
- 注意：只跟踪 Claude 的修改，不是 git 替代品

### 个人笔记空间

```
你常用的会话管理流程：
- 什么时候用 /clear，什么时候用 /rewind？
- 有没有因为没及时 compact 导致 session 变慢的经历？
- 你的命名习惯是什么？
```

---

## 5. 扩展与自动化

### Headless / Non-interactive Mode

```bash
# One-off queries
claude -p "Explain what this project does"

# Structured output for scripts
claude -p "List all API endpoints" --output-format json

# Streaming for real-time processing
claude -p "Analyze this log file" --output-format stream-json

# CI 中自动修复
claude --permission-mode auto -p "fix all lint errors"
```

### 多 Session 并行

**Writer / Reviewer 模式**：

| Session A (Writer) | Session B (Reviewer) |
|-------------------|-------------------|
| 实现功能 | |
| | Review 实现，查找 edge case 和竞态条件 |
| 根据反馈修改 | |

类似的还有：Session A 写测试，Session B 写实现。

### Fan-out 批量操作

```bash
for file in $(cat files.txt); do
  claude -p "Migrate $file from React to Vue. Return OK or FAIL." \
    --allowedTools "Edit,Bash(git commit *)"
done
```

先测试 2-3 个文件，精炼 prompt，再全量跑。`--allowedTools` 限制 Claude 的权限范围，适合无人值守运行。

### /loop 循环

需要轮询或重复执行时使用 `/loop <interval> <command>`。

### 个人笔记空间

```
自动化场景清单：
- CI pipeline 集成：
- 批量迁移任务：
- 并行审查流程：
```

---

## 6. 常见失败模式与避免方法

> **核心原则**：Recognizing them early saves time.

### 五大常见失败模式

#### (1) The Kitchen Sink Session —— 大杂烩会话

**表现**：从一个任务开始，中途插入不相关的问题，再回去继续。上下文被无关信息污染。

**对策**：`/clear` 切换不相关任务。

#### (2) Correcting Over and Over —— 反复纠正

**表现**：Claude 做错了，你纠正，它还是错，再纠正。上下文被失败方法污染。

**对策**：两次纠正后 `/clear` + 更好的 prompt（把学到的东西写进新 prompt）。

#### (3) The Over-specified CLAUDE.md —— 过度指定的说明文件

**表现**：CLAUDE.md 太长，Claude 忽略了一半，重要的规则被淹没在噪音中。

**对策**：Ruthlessly prune。For each line, ask："Would removing this cause Claude to make mistakes?" If not, cut it.

#### (4) The Trust-Then-Verify Gap —— 信任而未验证的缺口

**表现**：Claude 生成了看似正确的实现，但未处理边界情况。

**对策**：Always provide verification（tests, scripts, screenshots）。

#### (5) The Infinite Exploration —— 无限探索

**表现**：要求 Claude "调查"某问题但没限定范围，Claude 读了上百个文件，填满上下文。

**对策**：限定调查范围，或用 Subagent 隔离探索。

### 其他注意

- **上下文污染（Context Contamination）**：一个 session 中的问题会影响 Claude 对其他问题的判断。
- **过度委托（Over-delegation）**：把太简单的任务也交给 Subagent，增加了开销却无实质收益。
- **Loop 陷阱**：Claude 在某些情况下可能陷入重复循环。用 `Esc` 打断，检查 prompt 是否缺少明确的终止条件。

### 个人笔记空间

```
你遇到过哪些失败模式？如何解决的？
1.
2.
3.
```

---

## 7. 培养直觉

> **核心原则**：The patterns in this guide aren't set in stone. They're starting points.

### 何时做什么 —— 判断框架

| 场景 | 建议做法 |
|------|---------|
| 深度解决一个复杂问题 | 让上下文积累，历史有价值 |
| 探索性任务 | 跳过规划，让 Claude 自由发挥 |
| 想发现未知问题 | 用 vague prompt："what would you improve?" |
| 明确要改什么 | 精准 prompt + test/verify |

### 如何培养直觉

- **好输出时**：记下你做了什么（prompt 结构、提供的上下文、使用的模式）
- **Claude 挣扎时**：问为什么（上下文噪音？prompt 太模糊？任务太大？）
- Over time, you'll develop intuition that no guide can capture.

### 何时干预的信号

| 信号 | 行动 |
|------|------|
| Claude 读了很多文件但没进展 | 可能陷入了无限探索，限定范围或用 Subagent |
| 同一个问题连着问两遍 | 上下文太满，compact 或 /clear |
| 重复犯同一个错误 | 两次纠正后 /clear + 更好的 prompt |
| 生成的代码看起来正确但有遗漏 | 缺少验证条件，补测试用例 |

### 个人笔记空间

```
你的直觉记录：
- 什么情况下你发现"这里应该用 plan mode"？
- 什么信号告诉你"该 /clear 了"？
- Claude 表现最好的 session，你做了什么特别的事？
```

---

## 8. 成本控制要点

> 参考：`code.claude.com/docs/en/costs.md`

### 成本概况

- 企业部署平均成本约 $13/开发者/活跃日，$150-250/开发者/月
- 90% 用户日成本不到 $30
- 订阅制用户（Max/Pro）成本包含在订阅费中

### 降低成本的核心策略

| 策略 | 说明 |
|------|------|
| **模型选择** | Sonnet 处理大部分编码任务，成本远低于 Opus；Opus 仅用于复杂架构决策 |
| **Extended Thinking 设置** | 默认启用。对简单任务可降低 effort（`/effort`）或禁用（`/config`），或设 `MAX_THINKING_TOKENS=8000` |
| **上下文管理** | 上下文越大每 token 成本越高。`/clear` 频繁清理，`/compact` 控制压缩 |
| **MCP 开销** | MCP 工具默认 deferred 加载。优先用 CLI 工具（gh, aws, gcloud）。`/mcp` 禁用不用的 server |
| **从 CLAUDE.md 迁移到 Skills** | CLAUDE.md 每次 session 都加载；技能按需加载，保持基础上下文精简（建议 < 200 行） |
| **Subagent 处理高开销操作** | 跑测试、取文档、处理日志——把 verbose 输出留在 Subagent 上下文中，只返回摘要 |
| **代码智能插件** | 精确符号跳转减少文件读取，TypeScript 等语言自动报错减少编译循环 |
| **Hooks 预处理数据** | 代替 Claude 读 10000 行日志，hook 先 grep "ERROR" 只返回匹配行 |
| **写精确的 prompt** | "improve this codebase" 触发大范围扫描；"add input validation to the login function in auth.ts" 让 Claude 高效工作 |

### Agent Teams 成本

- Agent teams 消耗约 7x 标准 session 的 token（每个 teammate 独立上下文窗口）
- 控制方法：用 Sonnet、保持小队规模、spawn prompt 简洁、完成后立即清理

### 成本跟踪

- `/usage` 查看当前 session token 用量和预估成本
- 订阅用户关注 plan usage bars 而非 dollar figure
- Console 中设置 workspace spend limits

### 模型路由建议

```text
简单查询 / Subagent → haiku
一般编码任务 → sonnet
复杂架构 / 多步推理 → opus
```

### 个人笔记空间

```
你的成本优化记录：
- 当前平均每个 session 多少 token？
- 哪些场景你用 Opus 值得？
- MCP server 配置中有没有可以禁用的？
- CLAUDE.md 当前多少行？哪些可以迁移到 skills？
```

---

> **总结 Checklist**
>
> - [ ] 每次给任务时提供了验证手段？
> - [ ] 复杂任务先探索 → 规划 → 编码？
> - [ ] Prompt 精确具体，提供了 rich context？
> - [ ] 会话及时清理，不积累脏上下文？
> - [ ] 了解 headless / 多 session / fan-out 的适用场景？
> - [ ] 能识别并避免五种常见失败模式？
> - [ ] 建立了自己的"直觉判断框架"？
> - [ ] 成本控制措施已应用（模型选择、thinking 设置、MCP 管理）？
