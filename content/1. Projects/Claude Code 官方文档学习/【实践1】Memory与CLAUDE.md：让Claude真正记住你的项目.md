---
title: 【实践1】Memory与CLAUDE.md：让Claude真正记住你的项目
publish: true
---

# 【实践1】Memory与CLAUDE.md：让Claude真正记住你的项目

> 不只是写几条规则——理解CLAUDE.md的加载层级、auto memory机制、rules目录，让Claude在正确的时机拿到正确的上下文。

**参考文档**：
- `code.claude.com/docs/en/memory.md` — How Claude remembers your project
- `code.claude.com/docs/en/claude-directory.md` — Explore the .claude directory

**先修知识**：
- [[【2】Claude Code配置体系详解]]

**学习目标**：
- 理解 CLAUDE.md 与 Auto Memory 的定位差异与互补关系
- 掌握 CLAUDE.md 的加载层级和作用域
- 学会编写有效的 CLAUDE.md 指令
- 了解 .claude/rules/ 目录的路径特定规则
- 理解 Auto Memory 原理与审计方法

---

## 1. CLAUDE.md vs Auto Memory：各自定位与互补

Claude Code 每次会话都以全新的 context window 开始。两种机制负责跨会话传递知识：

| 维度 | CLAUDE.md | Auto Memory |
| --- | --- | --- |
| **谁来写** | 你（人为编写） | Claude（自动记录） |
| **内容性质** | 指令与规则（coding standards, workflows, architecture） | 学习与模式（build commands, debugging insights, preferences） |
| **作用域** | Project / User / Org 三层 | 每个 git 仓库独立，跨 worktree 共享 |
| **加载方式** | 每会话完整加载 | 每会话加载前 200 行或前 25KB |
| **适用场景** | 编码标准、项目架构、工作流约定 | Claude 自己发现的构建命令、调试习惯、修复偏好 |

**核心原则**：两者是互补关系，而非替代关系。CLAUDE.md 存放你"不想重复解释"的确定性规则，Auto Memory 存放 Claude 从对话中归纳出的隐含模式。前者是你主动告诉 Claude 的，后者是 Claude 自己学会的。

> 注意：CLAUDE.md 以用户消息（user message）而非系统提示（system prompt）的形式注入上下文。Claude 会尽力遵循，但不是硬性约束。如果某条指令必须在特定时机执行，应当写成 hook。

---

## 2. CLAUDE.md 的加载层级

CLAUDE.md 存在四个作用域层级，按加载顺序从宽到窄：

| 层级 | 位置 | 用途 | 共享范围 |
| --- | --- | --- | --- |
| **Managed Policy** | macOS: `/Library/Application Support/ClaudeCode/CLAUDE.md`<br>Linux: `/etc/claude-code/CLAUDE.md`<br>Windows: `C:\Program Files\ClaudeCode\CLAUDE.md` | 组织级强制指令 | 全组织所有用户 |
| **User（全局）** | `~/.claude/CLAUDE.md` | 个人偏好（所有项目生效） | 仅自己 |
| **Project（项目）** | `./CLAUDE.md` 或 `./.claude/CLAUDE.md` | 团队共享的项目指令 | 团队成员（通过 git） |
| **Local（本地）** | `./CLAUDE.local.md` | 个人项目偏好，建议加入 `.gitignore` | 仅自己 |

**目录树加载行为**：Claude Code 从当前工作目录向上遍历目录树，逐级发现 CLAUDE.md 和 CLAUDE.local.md。所有文件**拼接而非覆盖**进入上下文，顺序从文件系统根目录到当前目录。对于 `foo/bar/` 作为工作目录的情况，`foo/CLAUDE.md` 出现在 `foo/bar/CLAUDE.md` 之前；同一目录内 CLAUDE.local.md 追加在 CLAUDE.md 之后，确保你的个人笔记最后被读到。

**子目录加载**：工作目录下的子目录中的 CLAUDE.md 不会在启动时加载，而是在 Claude 读取该子目录中的文件时按需载入。

**跨工作目录**：使用 `--add-dir` 标志可以附加额外目录，但默认不会加载这些目录的 CLAUDE.md。需要设置环境变量 `CLAUDE_CODE_ADDITIONAL_DIRECTORIES_CLAUDE_MD=1` 来启用。

---

## 3. 写一份有效的 CLAUDE.md

### 什么时候应该添加

- Claude 第二次犯同一个错误时
- Code review 指出 Claude 本应知道的代码库知识时
- 你发现自己输入了和之前会话一样的修正说明时
- 新同事需要同样的上下文才能高效工作时

### 编写原则

| 原则 | 坏的例子 | 好的例子 |
| --- | --- | --- |
| **具体性** | "Format code properly" | "Use 2-space indentation" |
| **可验证性** | "Test your changes" | "Run `npm test` before committing" |
| **位置明确** | "Keep files organized" | "API handlers live in `src/api/handlers/`" |

**篇幅控制**：每个 CLAUDE.md 文件建议控制在 **200 行以内**。超过 200 行的文件会消耗更多 context 并降低遵循度。如果是大型项目，用 `.claude/rules/` 做路径限定加载。

**结构**：使用 markdown 标题和列表来分组指令。Claude 扫描结构的方式和人类读者一样：有组织的段落比密集的文字更容易遵循。

**一致性**：两条冲突的规则会导致 Claude 任意选择其一。定期审查 CLAUDE.md 及其子目录文件，移除过时或冲突的指令。

### 导入外部文件

使用 `@path/to/file` 语法可以在 CLAUDE.md 中导入其他文件：

```markdown
See @README for project overview and @package.json for available npm commands.

# Additional Instructions
- git workflow @docs/git-instructions.md
```

- 支持相对路径和绝对路径，相对路径基于**包含该 import 的文件**解析
- 支持递归导入，最大深度为 5 层
- 首次导入时会弹出批准对话框
- 被导入的文件**仍然占用 context 空间**，仅做组织整理，不会减少 token 消耗

### AGENTS.md 共存

如果仓库已有 AGENTS.md，可以创建 CLAUDE.md 来导入它：

```markdown
@AGENTS.md

## Claude Code
Use plan mode for changes under `src/billing/`.
```

也可以用符号链接：`ln -s AGENTS.md CLAUDE.md`（Windows 上需要管理员权限或 Developer Mode，建议用 `@` 导入）。

---

## 4. .claude/rules/ 目录：路径特定规则

在 `.claude/rules/` 目录中放置 markdown 文件，每个文件聚焦一个主题（如 `testing.md`、`api-design.md`），可以按子目录组织：

```
your-project/
├── .claude/
│   ├── CLAUDE.md
│   └── rules/
│       ├── code-style.md
│       ├── testing.md
│       └── security.md
```

### 路径限定规则

通过 YAML frontmatter 的 `paths` 字段将规则限定到特定文件模式：

```markdown
---
paths:
  - "src/api/**/*.ts"
---

# API Development Rules
- All API endpoints must include input validation
- Use the standard error response format
- Include OpenAPI documentation comments
```

**匹配规则**：

| 模式 | 匹配范围 |
| --- | --- |
| `**/*.ts` | 所有目录下的 TypeScript 文件 |
| `src/**/*` | `src/` 目录下所有文件 |
| `*.md` | 项目根目录的 markdown 文件 |
| `src/**/*.{ts,tsx}` | 多个扩展名的匹配（brace expansion） |

**行为**：不设定 `paths` 的规则在启动时无条件加载。设定了 `paths` 的规则在 Claude **读取匹配文件时**触发，而非每次工具调用都触发。

### 用户级规则

`~/.claude/rules/` 中的规则对所有项目生效：

```
~/.claude/rules/
├── preferences.md    # 个人编码偏好
└── workflows.md      # 个人工作流
```

**加载顺序**：用户级别规则先于项目规则加载，因此项目规则优先级更高。

### 跨项目共享规则

`.claude/rules/` 支持符号链接：

```bash
ln -s ~/shared-claude-rules .claude/rules/shared
ln -s ~/company-standards/security.md .claude/rules/security.md
```

循环符号链接会被检测并优雅处理。

---

## 5. Auto Memory 原理与审计

### 工作原理

Auto Memory 让 Claude 在工作过程中自行记录知识，无需你手动编写。Claude 会判断哪些信息对未来对话有用——构建命令、调试洞察、架构笔记、代码风格偏好、工作流习惯等。

> 需要 Claude Code v2.1.59 或更高版本。检查版本：`claude --version`

### 存储位置

每个项目有独立的 memory 目录：

```
~/.claude/projects/<project>/memory/
├── MEMORY.md          # 精炼索引（每会话加载前 200 行或 25KB）
├── debugging.md       # 调试模式的详细笔记
├── api-conventions.md # API 设计决策
└── ...
```

- 路径 `<project>` 源自 git 仓库标识，因此同一仓库的所有 worktree 和子目录共享一份 auto memory
- 非 git 仓库则使用项目根目录
- **仅限本机**，不会跨机器或云环境同步

### 自定义存储目录

可以在 `~/.claude/settings.json`（仅限 policy 和 user 层级）中设置：

```json
{
  "autoMemoryDirectory": "~/my-custom-memory-dir"
}
```

不接受 project/local 层级的此设置，防止克隆的仓库重定向 memory 写入到敏感位置。

### 启用/禁用

默认启用。可以通过以下方式控制：

- 会话中 `/memory` 命令的 toggle 开关
- 项目 settings.json：

```json
{
  "autoMemoryEnabled": false
}
```

- 环境变量：`CLAUDE_CODE_DISABLE_AUTO_MEMORY=1`

### 审计与编辑

Auto Memory 文件是纯文本 markdown，随时可以编辑或删除：

- 运行 `/memory` 命令浏览当前会话加载的所有 CLAUDE.md、CLAUDE.local.md、rules 文件
- 在 `/memory` 中可以直接打开 auto memory 文件夹
- 当你说 "remember to ..." 时，Claude 会写入 auto memory；说 "add this to CLAUDE.md" 则会写入 CLAUDE.md

---

## 6. 大型团队中的 CLAUDE.md 管理策略

### 组织级部署

通过配置管理系统（MDM、Group Policy、Ansible 等）分发 Managed Policy CLAUDE.md 到所有开发者机器。这个文件无法被个人设置排除，确保组织级指令始终生效。

另一种方式：将 CLAUDE.md 内容直接放入 `managed-settings.json`：

```json
{
  "claudeMd": "Always run `make lint` before committing.\nNever push directly to main."
}
```

**Managed settings vs CLAUDE.md 的分工**：

| 关注点 | 配置位置 |
| --- | --- |
| 阻止特定工具、命令或文件路径 | Managed settings: `permissions.deny` |
| 强制 sandbox 隔离 | Managed settings: `sandbox.enabled` |
| 环境变量和 API 路由 | Managed settings: `env` |
| 代码风格和质量指南 | Managed CLAUDE.md |
| 数据处理与合规提醒 | Managed CLAUDE.md |
| Claude 的行为指导 | Managed CLAUDE.md |

### 排除特定 CLAUDE.md 文件

大型 monorepo 中，祖先目录可能包含无关指令。使用 `claudeMdExcludes` 跳过特定文件（建议放在 `.claude/settings.local.json` 中，保持本地）：

```json
{
  "claudeMdExcludes": [
    "**/monorepo/CLAUDE.md",
    "/home/user/monorepo/other-team/.claude/rules/**"
  ]
}
```

模式基于**绝对路径**匹配。Managed Policy CLAUDE.md 无法被排除。

---

## 7. 常见问题排查

### Claude 不遵循 CLAUDE.md

1. **运行 `/memory`** 确认文件已加载。如果文件不在列表中，Claude 看不到它。
2. 检查文件是否放在正确的加载位置（参见第 2 节的层级表）。
3. 让指令更具体——"Use 2-space indentation" 比 "format code nicely" 有效得多。
4. 查找冲突指令——多个 CLAUDE.md 给出矛盾的指导时，Claude 会任意选择。
5. 如果是**必须在某个时机执行**的指令，应当写成 hook 而非 CLAUDE.md。

### 不知道 Auto Memory 保存了什么

运行 `/memory`，选择 auto memory 文件夹浏览。所有内容都是纯文本，可以读取、编辑或删除。

### CLAUDE.md 太大

超过 200 行的文件会降低遵循度。解决方案：
- 用路径限定的 `.claude/rules/` 让指令只在处理匹配文件时加载
- 裁剪不必要的内容
- 注意：拆分成 `@path` import 只帮助组织，不节省 context 空间

### Compact 后指令丢失

- **项目根目录的 CLAUDE.md**：在 `/compact` 后会被重新从磁盘读取并注入会话
- **子目录 CLAUDE.md**：不会自动重新注入，需要等到 Claude 再次读取该子目录的文件
- **纯对话中的指令**：完全丢失。解决方案是及时写入 CLAUDE.md 或 Auto Memory

---

**延伸阅读**：
- `.claude/` 目录完整文件参考：[[CLAUDE.md]]、`rules/`、`settings.json`、`hooks`、`skills/`、`agents/`
- 官方文档：Debug your configuration、Skills、Settings
