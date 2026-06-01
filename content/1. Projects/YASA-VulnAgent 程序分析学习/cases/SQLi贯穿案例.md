---
publish: true
---

# SQLi贯穿案例

## 示例代码

```python
def search_user(request, db):
    name = request.args["name"]
    sql = "select * from users where name = '" + name + "'"
    return db.execute(sql)
```

## 污点标注

| 元素 | 代码 | 说明 |
|---|---|---|
| Source | `request.args["name"]` | HTTP query 参数，不可信输入 |
| Propagator | `name = request.args["name"]` | 赋值传播 |
| Propagator | 字符串拼接生成 `sql` | 用户输入进入 SQL 字符串 |
| Sink | `db.execute(sql)` | SQL 执行 API |
| Sanitizer | 无 | 没有参数化查询或转义 |

## Source-to-Sink 路径

```text
request.args["name"]
  -> name
  -> sql = "select ... '" + name + "'"
  -> db.execute(sql)
```

## 误报检查

1. 如果 `db.execute` 实际执行参数化查询包装器，风险需要重新判断。
2. 如果 `name` 在进入拼接前经过严格 allowlist，路径需要标注 sanitizer。
3. 如果该 handler 不可由外部请求到达，需要结合入口点和路由信息。

## 伪查询目标

查找 HTTP 参数流向 SQL 执行，并且路径上没有参数化查询 sanitizer 的路径。
