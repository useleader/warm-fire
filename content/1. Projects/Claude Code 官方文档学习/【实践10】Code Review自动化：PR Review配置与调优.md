---
title: "【实践10】Code Review自动化：PR Review配置与调优"
publish: true
---

# 【实践10】Code Review 自动化：PR Review 配置与调优

> 让 Claude 自动审查 PR——多 Agent 分析、严重度分级、自定义规则、Ultrareview 云端深度审查。

**参考文档**：
- `code.claude.com/docs/en/code-review.md` — Set up automated PR reviews
- `code.claude.com/docs/en/ultrareview.md` — /ultrareview

**先修知识**：[[【4】Subagents深度指南]]

**学习目标**：理解 Code Review 工作原理、掌握配置与触发、学会自定义 Review 规则、了解 Ultrareview

---

## 1. Code Review 工作原理

> Code Review is in research preview, available for Team and Enterprise subscriptions. Not available for organizations with Zero Data Retention enabled.

### 多 Agent 分析流程

Code Review 的工作原理是：部署一组专门的 Agent 分析 GitHub PR，并将发现的问题以内联评论（inline comments）的形式发布在对应代码行上。

```
PR 提交 → 多个 Agent 并行分析 diff 与周边代码
    ↓
每个 Agent 负责不同类别的问题（逻辑错误、安全漏洞、边界情况、回归等）
    ↓
Verification 步骤过滤假阳性（false positives）
    ↓
结果去重、按严重度排序
    ↓
发布为 inline comments + 总结评论
```

关键特性：

- **Multi-agent parallel analysis**：多个审查 Agent 在 Anthropic 基础设施上并行工作，每个 Agent 查找不同类别的问题
- **Full codebase context**：审查不仅看 diff，还结合整个代码库的上下文
- **Verification step**：验证步骤会对照实际代码行为检查候选问题，过滤假阳性
- **Average completion time**：约 20 分钟

### 严重度分级（Severity Levels）

每个发现（finding）都带有严重度标签：

| Marker | Severity | 含义 |
| --- | --- | --- |
| 🔴 | Important | A bug that should be fixed before merging |
| 🟡 | Nit | A minor issue, worth fixing but not blocking |
| 🟣 | Pre-existing | A bug that exists in the codebase but was not introduced by this PR |

每个 finding 包含可折叠的 extended reasoning 区域，展开后可查看 Claude 为什么标记该问题以及如何验证。

### Check Run 输出

除了 inline review comments，每次审查还会填充 Claude Code Review check run（与 CI checks 并列显示）。点击 Details 链接可以看到按严重度排序的发现汇总表：

```
| 严重度 | File:Line | Issue |
|--------|-----------|-------|
| 🔴 Important | src/auth/session.ts:142 | Token refresh races with logout... |
| 🟡 Nit | src/auth/session.ts:88 | `parseExpiry` silently returns 0... |
```

Annotations 也会出现在 Files changed 选项卡的 diff 行上：
- 🔴 Important → 红色标记
- 🟡 Nit → 黄色警告
- 🟣 Pre-existing → 灰色提示

**Check run 始终以 neutral 结论结束**，不会通过 branch protection 阻止合并。如果你想基于 Code Review findings 做门槛，可以在自己的 CI 中解析 check run 输出：

```bash
gh api repos/OWNER/REPO/check-runs/CHECK_RUN_ID \
  --jq '.output.text | split("bughunter-severity: ")[1] | split(" -->")[0] | fromjson'
```

返回 JSON 格式：`{"normal": 2, "nit": 1, "pre_existing": 0}`。`normal` 表示 Important 发现的数量。

### 默认检查范围

By default, Code Review focuses on **correctness**: bugs that would break production, not formatting preferences or missing test coverage. 可以通过在仓库中添加 `CLAUDE.md` 或 `REVIEW.md` 来扩展检查范围。

---

## 2. 配置与触发方式

### 前置条件

1. Team 或 Enterprise 订阅
2. 组织管理员权限（claude.ai 管理员 + GitHub 组织权限）
3. 安装 Claude GitHub App（需要 Contents: read & write, Issues: read & write, Pull requests: read & write）

### 配置步骤

1. 访问 `claude.ai/admin-settings/claude-code` → Code Review 区域
2. 点击 Setup → 安装 Claude GitHub App
3. 选择要启用 Code Review 的仓库
4. 为每个仓库选择 Review Behavior（触发方式）

### 三种触发模式

| 模式 | 行为 | 适用场景 |
| --- | --- | --- |
| **Once after PR creation** | PR 创建或标记为 ready for review 时执行一次 | 标准流程，成本可控 |
| **After every push** | PR 分支每次推送都执行，自动解析已修复的线程 | 需要持续追踪的高风险项目 |
| **Manual** | 仅通过 `@claude review` 评论手动触发 | 高流量仓库，按需开启 |

### 手动触发命令

| Command | 行为 |
| --- | --- |
| `@claude review` | 启动一次审查，并将该 PR 订阅为 push-triggered（后续推送自动审查） |
| `@claude review once` | 启动单次审查，不订阅后续推送 |

**手动触发注意事项**：
- 必须是 PR 的顶级评论（不是 inline comment）
- 命令必须放在评论开头
- 需要仓库的 owner / member / collaborator 权限
- PR 必须是 open 状态
- 手动触发可以在 draft PR 上运行

### GitHub Actions / GitLab CI 集成

如果希望在自己的 CI 基础设施中运行 Claude，参考：
- **GitHub Actions**: `code.claude.com/docs/en/github-actions.md`
- **GitLab CI/CD**: `code.claude.com/docs/en/gitlab-ci.md`

这些方式适用于自托管场景（self-hosted），与 Code Review 托管服务不同。

---

## 3. 自定义 Review：CLAUDE.md 与 REVIEW.md

Code Review 读取仓库中的两个文件来指导审查行为。它们的区别和影响力层级不同：

### CLAUDE.md

- **作用范围**：Claude Code 所有任务共享的项目指令（不仅是 Code Review）
- **优先级**：作为 project context 读取
- **违规处理**：新引入的违反 `CLAUDE.md` 规则的行为标记为 **Nit** 级别
- **目录层级**：支持目录层级，子目录的 `CLAUDE.md` 仅适用于该路径下的文件
- **双向影响**：如果 PR 修改导致 `CLAUDE.md` 中的描述过时，Claude 也会标记需要更新文档

> 对于只想影响 Code Review 而不影响普通 Claude Code 会话的规则，使用 REVIEW.md。

### REVIEW.md

**文件位置**：仓库根目录

**优先级**：最高优先级——内容直接注入到每个审查 Agent 的 system prompt 中，覆盖默认审查指令

**重要限制**：`REVIEW.md` 是纯文本指令，不支持 `@` 导入语法，引用的文件不会被读入 prompt。需要把规则直接写在文件里。

#### 可调优的方向

| 维度 | 说明 |
| --- | --- |
| **Severity 重定义** | 重新定义 🔴 Important 的含义。对文档仓库、配置仓库、原型项目可以收窄定义；也可以将某些规则违反升级为 Important |
| **Nit 数量上限** | 限制单次审查的 🟡 Nit 评论数量，如 "at most five nits, mention the rest as a count in the summary" |
| **跳过规则** | 指定哪些路径、分支模式、发现类别不需要报告。常见目标：生成的代码、lockfile、vendor 依赖、机器生成的分支、CI 已检查的内容（lint/formatting/type errors） |
| **仓库特定检查** | 添加每个 PR 都要检查的规则，如 "new API routes must have an integration test" |
| **验证门槛** | 要求提供证据才能报告某类发现，如 "behavior claims need a file:line citation" |
| **Re-review 收敛** | 已审查过的 PR 只报告 Important，不报告新的 Nit |
| **Summary 格式** | 要求 review body 以一行统计开头，如 `2 factual, 4 style` |

#### REVIEW.md 示例

```markdown
# Review instructions

## What Important means here

Reserve Important for findings that would break behavior, leak data,
or block a rollback: incorrect logic, unscoped database queries, PII
in logs or error messages, and migrations that aren't backward
compatible. Style, naming, and refactoring suggestions are Nit at
most.

## Cap the nits

Report at most five Nits per review. If you found more, say "plus N
similar items" in the summary instead of posting them inline. If
everything you found is a Nit, lead the summary with "No blocking
issues."

## Do not report

- Anything CI already enforces: lint, formatting, type errors
- Generated files under `src/gen/` and any `*.lock` file
- Test-only code that intentionally violates production rules

## Always check

- New API routes have an integration test
- Log lines don't include email addresses, user IDs, or request bodies
- Database queries are scoped to the caller's tenant
```

> **Keep it focused**: 过长的 `REVIEW.md` 会稀释最重要的规则。把通用项目上下文留在 `CLAUDE.md` 中。

---

## 4. 结果解读与反馈

### Inline Comments

- 审查结果直接发布在相关代码行的 inline comment 上
- 每个 comment 自带 👍 和 👎 按钮，用于反馈
- 回复 inline comment **不会** 触发 Claude 回应或更新 PR

### Rate 机制

- 点击 👍 表示 finding 有用（useful）
- 点击 👎 表示 finding 是错误或噪音（wrong or noisy）
- Anthropic 会在 PR merge 后收集 reaction 统计数据，用于调优审查模型
- Reactions 不会触发重新审查或改变 PR 上的任何内容

### Summary 评论

- 审查完成后 Claude 在 PR 上发布一条总结评论
- 如果未发现问题，Claude 发布简短确认评论
- 如果审查运行时又有新的 push，部分 findings 可能出现在 "Additional findings" 部分

### 修复与再审查

- **修复代码并推送**：如果 PR 订阅了 push-triggered review，下次推送会自动解析已修复的线程
- **请求重新审查**：作为顶级 PR 评论发布 `@claude review once`
- **查看 Check Run Details**：即使 GitHub 拒绝了某行的 inline comment（行已变动），findings 仍然在 check run Details 中可查

---

## 5. /ultrareview：云端深度多 Agent Review

> Ultrareview is a research preview feature available in Claude Code v2.1.86+.

### 概念

Ultrareview 是在 Claude Code on the web 基础设施上运行的深度代码审查。运行 `/ultrareview` 时，Claude Code 在远程沙箱中启动一组审查 Agent 来查找当前分支或 PR 中的 bug。

### 与 /review 的对比

| 维度 | `/review` | `/ultrareview` |
| --- | --- | --- |
| 运行位置 | 本地 session | 远程云沙箱 |
| 深度 | single-pass review | multi-agent fleet + 独立验证 |
| 时长 | 几秒到几分钟 | 约 5-10 分钟 |
| 费用 | 计入普通用量 | 免费次数后约 $5-20/次（extra usage） |
| 最佳场景 | 迭代中的快速反馈 | merge 前的大改动深度审查 |

### 核心优势

- **Higher signal**：每个 reported finding 都经过独立复现和验证，结果聚焦于真实 bug 而非风格建议
- **Broader coverage**：多个审查 Agent 并行探索变更，能发现单次审查遗漏的问题
- **No local resource use**：完全在远程沙箱运行，终端可以继续其他工作

### 使用方式

```bash
# 在 CLI 中运行
/ultrareview                      # 审查当前分支与默认分支的 diff
/ultrareview 1234                 # 审查 GitHub PR #1234
```

**PR 模式要求**：
- 仓库必须有 `github.com` remote
- 大仓库会提示使用 PR 模式

**认证要求**：
- 需要 Claude.ai 账号认证（`/login`）
- 不支持 Amazon Bedrock / Google Cloud Vertex AI / Microsoft Foundry
- 不适用于 Zero Data Retention 组织

### 非交互式运行

```bash
claude ultrareview                # CLI 子命令，适合 CI/脚本
claude ultrareview 1234
claude ultrareview origin/main
```

| Flag | 说明 |
| --- | --- |
| `--json` | 打印原始 `bugs.json` payload |
| `--timeout <minutes>` | 最长等待时间，默认 30 分钟 |

Exit codes:
- 0: review 完成（有或无 findings）
- 1: 启动失败 / 远程错误 / 超时
- 130: Ctrl-C 中断（远程 review 继续运行）

### 跟踪运行状态

```bash
/tasks        # 查看运行中和已完成的 review
```

- Review 通常 5-10 分钟
- 作为 background task 运行，可以继续工作甚至关闭终端
- 完成后通过 notification 返回 verified findings
- 停止 review 会归档云会话，partial findings 不返回

---

## 6. 用量与定价

### Code Review（PR Review 服务）

| 项目 | 说明 |
| --- | --- |
| 计费方式 | Token usage |
| 平均成本 | $15-25 / 次 review（取决于 PR 大小和代码库复杂度） |
| 是否计入套餐 | No — 通过 extra usage 单独计费 |
| 影响成本的因素 | 触发模式（After every push 成本最高）、PR 大小、issues 验证量 |

- 成本显示在 Anthropic 账单上
- 可以设置 monthly spend cap：`claude.ai/admin-settings/usage`
- 监控：`claude.ai/analytics/code-review`

### Ultrareview

| 计划 | 免费次数 | 超出资费 |
| --- | --- | --- |
| Pro | 3 次（一次性，不刷新） | Extra usage |
| Max | 3 次（一次性，不刷新） | Extra usage |
| Team / Enterprise | 无免费次数 | Extra usage |

- 典型成本：$5-20 / 次
- 需要启用 extra usage 才能进行付费 review
- 提前停止或失败的 review 仍计入免费次数
- 付费 review 只对实际运行部分计费

---

## 7. 常见问题排查

### Review 未触发

1. **确认仓库已启用**：在 `claude.ai/admin-settings/claude-code` 中检查仓库列表
2. **确认 GitHub App 权限**：Claude GitHub App 是否已获得该仓库访问权限
3. **确认触发模式**：Manual 模式下需要 `@claude review` 评论
4. **Draft PR**：自动触发不会在 draft PR 上运行，需要使用手动触发
5. **Zero Data Retention**：启用了 ZDR 的组织无法使用 Code Review
6. **Re-run 按钮无效**：GitHub Checks 标签页的 Re-run 按钮不会重新触发 Code Review。请使用 `@claude review once` 或推送新 commit

### Spend Cap 报错

- 评论显示 spend-cap 消息 → 当月 spend cap 已到
- 自动恢复：下个计费周期开始，或管理员提高 cap
- 设置位置：`claude.ai/admin-settings/usage`

### Inline Comment 不显示

如果 check run title 显示已发现 issues 但看不到 inline comments：

1. **Check Run Details**：点击 Details 查看 severity table，每个 finding 都有 file, line 和 summary
2. **Files changed annotations**：在 Files changed 选项卡中，findings 显示为 diff 行的 annotations（独立于 review comments）
3. **Additional findings**：如果 push 发生在 review 运行时，部分 findings 引用的行已不存在，会出现在 review body 的 Additional findings 部分

### Ultrareview 相关

- `claude ultrareview` 非交互模式需要和 `/ultrareview` 相同的认证和 extra usage 配置
- 中断 `claude ultrareview`（Ctrl-C）后远程 review 继续运行，可通过 session URL 跟踪
- 需要使用 `/login` 认证（如果仅使用 API key）

---

## 笔记与总结

### 关键要点

-

### 我自己的配置示例

-

### 疑问与待探索

-

---

## 相关笔记

- [[【4】Subagents深度指南]]
- 【实践11】GitHub Actions 集成（待创建）
- 【实践12】GitLab CI 集成（待创建）
