---
publish: true
---

# SSRF污点路径案例

## 示例代码

```python
import requests

def fetch_preview(request):
    target = request.args["url"]
    response = requests.get(target)
    return response.text[:200]
```

## 污点标注

| 元素 | 代码 | 说明 |
|---|---|---|
| Source | `request.args["url"]` | 用户可控 URL |
| Propagator | `target = request.args["url"]` | 赋值传播 |
| Sink | `requests.get(target)` | 服务端 HTTP client 请求 |
| Sanitizer | 无 | 没有 allowlist、协议限制、内网地址阻断 |

## Source-to-Sink 路径

```text
request.args["url"]
  -> target
  -> requests.get(target)
```

## 误报检查

1. 如果存在域名 allowlist，需检查是否覆盖重定向、DNS rebinding、IP literal 和私网地址。
2. 如果请求只发生在不可达内部任务中，外部可利用性下降。
3. 如果 HTTP client 禁止重定向且限制协议，风险可能降低。

## 伪查询目标

查找用户可控 URL 流向服务端 HTTP client，且路径上没有有效 allowlist 或内网地址阻断。
