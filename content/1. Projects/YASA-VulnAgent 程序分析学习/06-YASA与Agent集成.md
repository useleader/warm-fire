---
publish: true
---

# 06-YASA与Agent集成

## 学习目标

把程序分析能力拆成 Agent 可调用的工具 contract，理解 Supervisor、Skill、SkillResult、PEV 和 evidence grounding 的关系。

## Agent 链路

```text
Supervisor
  -> Skill-1 漏洞扫描：获取结构化污点路径
  -> Skill-2 污点推理：基于路径判断可利用性
  -> Skill-3 PoC 验证：生成并沙箱验证 PoC
  -> Skill-4 报告生成：输出证据链、CVSS、修复建议
```

## Tool contract 草案原则

1. 工具输入必须结构化，避免让 LLM 自由猜参数。
2. 工具输出必须包含 source、sink、路径、代码位置、sanitizer、错误状态。
3. 未确认的 YASA-MCP API 名称只作为推断设计记录。
4. SkillResult 必须包含 `success`、`data`、`confidence`、`evidence`、`error`。

## 本周练习

在 `outputs/Phase1最小验证链路.md` 中定义 Skill-1 到 Skill-4 的最小输入输出，并说明 Phase 1 只验证单语言、单漏洞类型、单靶场。

## 项目关联

这部分直接服务 YASA-VulnAgent Phase 1。目标是先做最小可验证链路，而不是一次覆盖 5 类漏洞和 100 个仓库。
