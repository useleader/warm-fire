---
publish: true
---

# YASA-VulnAgent 程序分析学习

> 目标：10 天内完成申报书可提交版，后续 4-5 周系统补齐 YASA-VulnAgent 所需的程序分析、污点分析、查询语言和 Agent 集成能力。

## 当前定位

这个项目是 `~/yasa-vuln-agent` 的学习与申报书支撑项目，不直接承载 YASA-VulnAgent 实现代码。

短期目标是支撑申报书成稿；长期目标是形成 Phase 1 最小验证链路的知识基础。

## 成功标准

1. 能解释 `源码 -> UAST -> 数据流/污点路径 -> LLM 推理 -> PoC 验证 -> 报告` 的完整链路。
2. 能拆解一条 source-to-sink 污点路径，并说明 source、propagator、sanitizer、sink 和误报来源。
3. 能说明 YASA 的价值来自 UAST 多语言统一表示与高精度分析能力，而不是简单替换一个 SAST 后端。
4. 能写出 SQLi、SSRF、命令注入的伪查询规则。
5. 能设计 YASA-MCP 风格的 Agent tool contract，并明确哪些接口细节仍需官方材料校准。
6. 能在 10 天内形成申报书可提交版。

## 路线

| 阶段 | 时间 | 核心目标 | 主要产出 |
|---|---:|---|---|
| 申报书冲刺 | D1-D10 | 快速建立框架并完成申报书 | `01-10天申报书冲刺.md`、`outputs/申报书素材清单.md` |
| 程序表示 | W1 | 学 AST、CFG、DFG、PDG、SSA、CPG、UAST | `02-程序表示与UAST.md` |
| 数据流基础 | W2 | 学 points-to、alias、敏感性维度 | `03-指针别名与数据流分析.md` |
| 污点分析 | W3 | 学 source、sink、sanitizer、propagator、taint path | `04-污点分析.md`、`cases/SQLi贯穿案例.md` |
| 查询语言 | W4 | 学 CodeQL、Datalog、伪 UQL 和 checker | `05-查询语言与漏洞规则.md`、`outputs/漏洞伪查询集.md` |
| Agent 集成 | W5 | 学 YASA-MCP tool contract、SkillResult、PEV | `06-YASA与Agent集成.md`、`outputs/Phase1最小验证链路.md` |

## 导航

- [[00-学习地图]]
- [[01-10天申报书冲刺]]
- [[02-程序表示与UAST]]
- [[03-指针别名与数据流分析]]
- [[04-污点分析]]
- [[05-查询语言与漏洞规则]]
- [[06-YASA与Agent集成]]
- [[cases/SQLi贯穿案例]]
- [[cases/SSRF污点路径案例]]
- [[cases/命令注入污点路径案例]]
- [[outputs/申报书素材清单]]
- [[outputs/漏洞伪查询集]]
- [[outputs/Phase1最小验证链路]]

## 可信边界

`~/yasa-vuln-agent` 中关于 YASA-MCP 的具体 API 名称、参数和返回结构在官方材料确认前都视为推断设计。申报书中可以描述“原子级程序分析工具能力”，但不能把未确认接口写成已发布 API。
