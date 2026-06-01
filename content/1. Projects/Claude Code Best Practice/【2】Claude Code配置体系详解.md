---
publish: true
---

# 【2】Claude Code配置体系详解：从CLAUDE.md到settings.json

> 覆盖：配置层级（5层）、CLAUDE.md加载机制、rules lazy-load、settings.json关键配置项、MCP server配置。

## 先修知识
- [[【序】LLM基础速通]]

## 学习目标
- 理解配置层级（5层）
- 理解CLAUDE.md加载机制
- 理解rules lazy-load
- 掌握settings.json关键配置项
- 理解MCP server配置

---

## 1. 配置层级

### 全球层

### 项目层

### 命令行参数

### local.json覆盖

### 优先级总结

---

## 2. CLAUDE.md加载机制

### ancestor/descendant加载

### 加载顺序

### 200行限制的意义

### \<important if>标签

---

## 3. Rules lazy-load

### paths: frontmatter的作用

### 全局加载 vs 按需加载

### 不带paths时

---

## 4. settings.json关键配置项

### permissions

### hooks

### env

### outputStyle

### 其他重要配置

---

## 5. MCP server配置

### .mcp.json结构

### Playwright/Context7/DeepWiki配置

### MCP server使用场景

---

## 练习题

## 相关概念
- [[【4】Subagents深度指南]]
- [[【8】Hooks自动化工作流]]
- [[【9】上下文管理策略]]

## 参考
