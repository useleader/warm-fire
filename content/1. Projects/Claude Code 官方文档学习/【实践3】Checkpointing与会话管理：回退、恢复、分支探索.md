---
title: 【实践3】Checkpointing与会话管理：回退、恢复、分支探索
publish: true
---

# 【实践3】Checkpointing 与会话管理：回退、恢复、分支探索

> 理解 Claude Code 的文件快照机制，学会用 rewind、fork、resume 管理探索性工作，不丢进度。

**参考文档**：
- `code.claude.com/docs/en/checkpointing.md` — Track, rewind, and summarize
- `code.claude.com/docs/en/how-claude-code-works.md` — How Claude Code works (sessions section)

**先修知识**：[[【1】LLM Agents机制]]、[[【9】上下文管理策略]]

**学习目标**：理解 Checkpoint 原理、区分 Rewind/Restore/Summarize、掌握 Fork 分支探索、了解会话生命周期

---

## 1. Checkpoint 工作原理

> Checkpointing 是 Claude Code 内置的文件快照机制 —— 每次 Claude 执行文件编辑工具前，会自动对受影响的文件做快照（snapshot），形成一个可回溯的"安全网"。

### 自动追踪时机

- **每次用户 Prompt 后**自动创建一个新的 checkpoint
- Checkpoint **跨会话持久化**（resume 后的会话仍可访问之前的 checkpoint）
- 默认 **30 天后自动清理**（可配置保留时长）
- 只追踪 **文件编辑工具**（Read / Edit / Write 等）产生的变更，**不追踪 bash 命令修改的文件**

### Rewind 入口

- 按 `Esc` + `Esc`（连续两次）
- 或使用 `/rewind` 命令
- 弹出可滚动的 prompt 列表，选择目标节点后执行操作

---

## 2. Rewind / Restore / Summarize 三者对比

| 操作 | 代码回退 | 对话回退 | 适用场景 |
|------|----------|----------|----------|
| **Restore code and conversation** | 是 | 是 | 完全回到某个历史节点，重新开始 |
| **Restore conversation** | 否 | 是 | 对话思路乱了但代码没问题 |
| **Restore code** | 是 | 否 | 代码改错了但对话分析有价值 |
| **Summarize from here** | 否（压缩对话） | 选中点之前保留，之后替换为摘要 | 剔除旁支讨论，保留早期上下文 |
| **Summarize up to here** | 否（压缩对话） | 选中点之前替换为摘要，之后保留 | 压缩早期 setup 讨论，保留近期工作 |

### 关键区别

- **Restore** 系列：**有副作用** —— 撤销代码 / 回退对话状态
- **Summarize** 系列：**无副作用** —— 仅压缩上下文，不改文件，保留原始消息在 session transcript 中，需要时 Claude 仍可引用
- **Restore** 后，选中点的原始 prompt 会恢复到输入框中，可重新发送或编辑
- **Summarize up to here** 后将停留在对话末尾，输入框为空

### 与 `/compact` 的区别

- `/compact` 压缩整个对话
- Summarize 系列是**定向压缩** —— 只压缩选中点之前或之后的部分

---

## 3. Fork 会话

> 当你想**尝试不同方案**但**保留原始会话完整**时，使用 fork。

### 如何 Fork

- CLI：`claude --continue --fork-session`
- 会话内：`/branch` 命令
- 本质：将当前会话历史**复制**到一个**新的 Session ID** 下，原会话保持不变

### 适用场景

- 对同一问题尝试多种实现方案
- 从某个历史节点分支出一条探索路径
- 对比主分支与分支的效果

### 与 Summarize 的区别

| | Summarize | Fork |
|---|-----------|------|
| 会话是否改变 | 同一次会话，上下文压缩 | 创建**新会话**（新 ID） |
| 原会话是否保留 | 原会话被压缩 | 原会话**完全保留** |
| 适用场景 | 清理上下文继续工作 | 分支探索、方案对比 |

---

## 4. Resume：跨设备跨时间继续会话

### Resume 方式

- `claude --continue`：恢复上一次会话（追加消息到原 Session ID）
- `claude --resume`：打开会话选择器
- `/resume` 命令：在会话内选择历史会话

### 跨设备（Remote Control）

- Remote Control 模式：在浏览器中使用 Claude Code，代码在你的机器上执行
- 会话文件存储在 `~/.claude/projects/`，跨设备继续需要同步该目录

### 同会话多终端

- 同一个 Session ID 在两个终端打开时，后启动的终端会**检测到冲突**并提示选择：
  - 继续当前会话（fork 为新会话）
  - 或使用新的独立会话

### 会话存储路径

```
~/.claude/projects/<project-hash>/sessions/<session-id>.jsonl
```

- 每个消息、工具调用及其结果都以 JSONL 格式持久化
- 纯文本格式，可直接查看和 grep

---

## 5. 会话生命周期管理

### 会话与分支的关系

- 每个 Claude Code 会话（session）**绑定到当前工作目录**
- **不绑定到 Git 分支** —— 切换 Git 分支后，对话历史仍然保留
- 使用 `git worktree` 可以在不同目录运行并行会话

### 会话管理操作

| 操作 | 方式 | 说明 |
|------|------|------|
| 查看历史会话 | `/resume` 选择器 | 默认显示当前 worktree 的会话 |
| 恢复会话 | `claude --continue` | 追加到原会话 |
| 分支会话 | `--fork-session` 或 `/branch` | 创建新 ID 的副本 |
| 命名会话 | `--name <name>` | 方便后续查找 |
| 存档 / 删除 | 手动管理 `~/.claude/projects/` | 无内置 archive/delete 命令 |

### 关键生命周期要点

- 新会话 = 全新 context window，不含历史对话
- Claude 通过 **Auto Memory**（`MEMORY.md`）跨会话持久化学习内容
- **CLAUDE.md** 提供持久化项目指令（不受会话影响）
- 会话文件默认保留 30 天后自动清理

---

## 6. 限制与边界

### Bash 命令变更不可追踪

> Checkpointing **不追踪** bash 命令对文件的修改。

```bash
rm file.txt        # 无法通过 rewind 撤销
mv old.txt new.txt # 不产生 checkpoint
cp source.txt dest.txt # 不可恢复
```

只有 Claude 通过文件编辑工具（Read / Edit / Write 等）产生的变更才被追踪。

### 外部变更不可追踪

- 手动在编辑器中的修改
- 其他并行 Claude Code 会话的修改
- 除非恰好修改了**同一文件**且当前会话也编辑过它

### 不替代 Git

| | Checkpoint | Git |
|---|-----------|-----|
| 作用域 | 单会话、本地 | 项目、协作、长期 |
| 生命周期 | 默认 30 天 | 永久（除非主动删除） |
| 共享 | 不可共享 | Push / Pull |
| 语义 | "本地撤销" | "永久历史" |

> **建议**：Checkpoint 作为快速撤销的"安全网"，Git 作为正式版本管理，两者互补使用。

### 远程副作用不可回滚

- 数据库写入
- API 调用
- 部署操作

这些操作不在 checkpoint 范围内，Claude 会在执行前通过权限系统（`Shift+Tab` 切换权限模式）要求确认。

---

## 实践笔记

> （学习过程中记录你的心得、遇到的问题、总结的技巧）

### 

### 

### 

---

## 参考链接

- [Checkpointing 官方文档](https://code.claude.com/docs/en/checkpointing.md)
- [How Claude Code Works 官方文档](https://code.claude.com/docs/en/how-claude-code-works.md)
