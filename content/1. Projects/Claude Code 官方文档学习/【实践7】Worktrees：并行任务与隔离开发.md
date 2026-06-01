---
title: "【实践7】Worktrees：并行任务与隔离开发"
publish: true
---

# 【实践7】Worktrees：并行任务与隔离开发

> 用 git worktree 让多个 Claude Code session 并行工作而不互相干扰——隔离、合并、清理全流程。
> Isolate parallel Claude Code sessions in separate git worktrees so changes don't collide.

**参考文档**：
- `code.claude.com/docs/en/worktrees.md` — Run parallel sessions with worktrees
- `code.claude.com/docs/en/agents.md` — Run agents in parallel

**先修知识**：[[【4】Subagents深度指南]]

**学习目标**：理解 worktree 隔离需求、学会创建和使用、了解 subagent 隔离、掌握清理管理

---

## 1. 为什么需要 Worktree（并行 session 的文件冲突问题、场景举例）

A git worktree is a separate working directory with its own files and branch, sharing the same repository history and remote as your main checkout. Running each Claude Code session in its own worktree means edits in one session never touch files in another.

**典型场景：**
- 一个 terminal 在 build feature，另一个 terminal 在 fix bug
- 同一个 repo 上不同的 PR 或者 issue
- Subagent 之间需要编辑重叠的文件

Worktrees handle **file isolation**, while subagents and agent teams coordinate the work itself.

> **对比不同并行方式：**
>
> | 方式 | 特点 | 适用场景 |
> | --- | --- | --- |
> | Subagents | 单 session 内委托子任务 | 不想主对话被搜索/日志冲掉 |
> | Agent view (`claude agents`) | 一个界面监控多个 session | 独立任务，随时切入查看 |
> | Agent teams | 多个 session 协同（实验性） | 拆分项目，worker 间互通消息 |
> | **Worktrees** | **独立 git checkout** | **多 session 编辑同一 repo，避免文件冲撞** |
> | `/batch` | 一次性拆分为 5-30 个 PR | 全仓迁移、机械性重构 |

---

## 2. 创建与使用（--worktree 标志、基础分支选择 fresh/head、附加到已有 worktree）

### 基本命令

```bash
# 创建并进入一个命名 worktree（新分支 worktree-<name>）
claude --worktree feature-auth

# 简写
claude -w bugfix-123

# 不指定名称，自动生成随机名（如 bright-running-fox）
claude --worktree

# 从特定 PR 创建 worktree
claude --worktree "#1234"
claude --worktree "https://github.com/owner/repo/pull/1234"
```

### 基础分支选择（base branch）

| 配置 | 效果 |
| --- | --- |
| `"fresh"`（默认） | 从 `origin/HEAD` 即 remote default branch 分支，起始为干净树 |
| `"head"` | 从当前本地 HEAD 分支，携带未推送的 commit 和 feature branch 状态 |

```json
{
  "worktree": {
    "baseRef": "head"
  }
}
```

> **注意**：`baseRef` 只接受 `"fresh"` 或 `"head"`，不支持任意 git ref。

### 在已有 worktree 中启动 Claude

直接 `cd` 进 worktree 目录再运行 `claude` 即可。Desktop app 对每个新 session 自动创建 worktree。

### EnterWorktree tool

在 session 中也可以让 Claude 自己创建 worktree。告诉 Claude "work in a worktree" 即可触发 `EnterWorktree` tool。

### 首次使用前注意

第一次在目录中使用 `--worktree` 之前，需要先在该目录下运行一次 `claude` 接受 workspace trust。否则 `--worktree` 会报错退出。

### 建议的 gitignore

将 `.claude/worktrees/` 加入 `.gitignore`，避免 worktree 内容在主 checkout 中显示为 untracked files。

```
.claude/worktrees/
```

---

## 3. 复制 gitignored 文件（.worktreeinclude 机制、env 文件等场景）

Worktree 是全新的 checkout，像 `.env`、`.env.local` 这种 untracked 文件不会自动存在。

**解决方案：** 在项目根目录创建 `.worktreeinclude` 文件，使用 `.gitignore` 语法。

```
# .worktreeinclude
.env
.env.local
config/secrets.json
```

- 只匹配 `.gitignore` 中的文件（不会复制 tracked files）
- 适用于 `--worktree`、subagent worktrees、desktop app 并行 session

> 对于非 git VCS，worktree 通过 hook 创建，`.worktreeinclude` 不会被处理。需要在 hook 脚本中自行复制。

---

## 4. Subagent 隔离（让子 agent 在独立 worktree 中运行、配置方式）

让 subagent 在各自的 worktree 中运行，避免并行编辑冲突。

### 临时指定

告诉 Claude："use worktrees for your agents"

### 永久配置

在自定义 subagent 的 frontmatter 中添加 `isolation: worktree`：

```yaml
---
name: my-agent
isolation: worktree
---
```

子 agent 获得一个临时 worktree，完成后自动移除（若无改动）。

---

## 5. 清理与管理（手动/自动清理、git worktree 管理命令）

### 退出时自动清理

| 情况 | 行为 |
| --- | --- |
| 无 uncommitted changes、untracked files、新 commits | worktree + branch 自动删除；有命名的 session 会提示 |
| 有改动 | 提示 keep 或 remove。Keep 保留目录和分支，Remove 删除两者并丢弃所有改动 |
| 非交互运行（`--worktree` + `-p`） | 不会自动清理，需手动 `git worktree remove` |

### 崩溃/中断恢复

启动时清理孤立的 subagent worktrees（超过 `cleanupPeriodDays` 配置），条件：
- 无 uncommitted changes
- 无 untracked files
- 无 unpushed commits

通过 `--worktree` 创建的 worktree 不会被自动 sweep。

### Git 原生管理命令

```bash
# 列出所有 worktree
git worktree list

# 手动创建 worktree（新分支）
git worktree add ../project-feature-a -b feature-a

# 从已有分支创建
git worktree add ../project-bugfix bugfix-123

# 手动移除
git worktree remove ../project-feature-a

# 清理过期的 worktree（git 2.17+）
git worktree prune
```

> 手动创建 worktree 后，需要自行初始化开发环境（安装依赖、设置 virtual env 等）。

---

## 6. 非 Git VCS 的 worktree 支持（hooks: WorktreeCreate/WorktreeRemove）

对于 SVN、Perforce、Mercurial 等系统，配置 hooks 提供自定义创建和清理逻辑。

### WorktreeCreate hook

从 stdin 读取 worktree name，创建新的工作副本，输出目录路径给 Claude Code：

```json
{
  "hooks": {
    "WorktreeCreate": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "bash -c 'NAME=$(jq -r .name); DIR=\"$HOME/.claude/worktrees/$NAME\"; svn checkout https://svn.example.com/repo/trunk \"$DIR\" >&2 && echo \"$DIR\"'"
          }
        ]
      }
    ]
  }
}
```

### WorktreeRemove hook

配合 `WorktreeRemove` hook 在 session 结束时清理。详见 hooks reference。

> **注意**：使用 `WorktreeCreate` hook 时，`.worktreeinclude` 不会被自动处理，需要在 hook 脚本中自行复制文件。

---

## 总结与最佳实践

1. **何时用 Worktree**：多个 session/subagent 需要编辑同一 repo 中可能重叠的文件
2. **何时不需要**：任务完全独立且不共享 repo
3. **分支策略**：默认 `fresh` 从 remote 最新状态出发；`head` 携带当前 WIP
4. **Subagent 隔离**：`isolation: worktree` 或口头告诉 Claude
5. **清理**：尽量让 Claude 自动清理；手动 `git worktree prune` 兜底
6. **非 git**：用 hooks 适配，注意 `.worktreeinclude` 失效

---

## 个人笔记

> （在此记录你的实践心得、踩坑记录、自定义配置等）

- 

---

## 延伸阅读

- [[【4】Subagents深度指南]]
- [Git Worktree Documentation](https://git-scm.com/docs/git-worktree)
- Desktop parallel sessions（code.claude.com/docs）
