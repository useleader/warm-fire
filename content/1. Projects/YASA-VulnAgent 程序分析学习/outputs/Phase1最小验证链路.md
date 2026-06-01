---
publish: true
---

# Phase1最小验证链路

## 目标

用单语言、单漏洞类型、单靶场先验证 YASA-VulnAgent 的核心闭环：结构化污点路径 -> LLM 可利用性判断 -> PoC 验证 -> 报告输出。

## 范围

| 维度 | Phase 1 选择 |
|---|---|
| 语言 | Python |
| 漏洞 | SQL 注入 |
| 靶场 | 一个可控 Flask/FastAPI 示例项目 |
| 程序分析输入 | source-to-sink 污点路径 |
| Agent 框架 | LangGraph Supervisor + 4 Skills |
| 验证方式 | HTTP PoC + Docker 沙箱 |

## Skill Contract 草案

### Skill-1 漏洞扫描

输入：

```json
{
  "project_path": "path/to/target",
  "language": "python",
  "cwe": "CWE-89"
}
```

输出：

```json
{
  "success": true,
  "data": {
    "paths": [
      {
        "source": "request.args['name']",
        "sink": "db.execute(sql)",
        "propagators": ["name = request.args['name']", "sql = ... + name"],
        "sanitizers": [],
        "locations": []
      }
    ]
  },
  "confidence": 0.7,
  "evidence": ["source-to-sink path found"],
  "error": null
}
```

### Skill-2 污点推理

输入：Skill-1 的结构化路径、代码片段、目标 CWE。

输出：可利用性评分、证据引用、误报风险、需要 PoC 验证的请求形态。

### Skill-3 PoC 验证

输入：目标服务启动方式、PoC 请求、预期响应特征。

输出：验证是否成功、请求日志、响应片段、失败原因。

### Skill-4 报告生成

输入：污点路径、推理结论、PoC 结果。

输出：CVE 风格报告草稿、CVSS 初评分、修复建议。

## 边界

以上 contract 是学习和 Phase 1 设计草案，不代表已确认的 YASA-MCP 官方 API。
