---
title: Unit 6 - Nano Harness 迷你 Agent 循环
publish: true
course_url: https://huggingface.co/learn/context-course/unit6/introduction
---

最近开始学习 Hugging Face 的 **Context Course**，这是 Unit 6（Bonus: Nano Harness）的学习笔记，也是整个课程的收官单元。

前面五个单元分别讲了 Skills（可移植知识）、MCP（工具协议）、Plugins（插件打包）、Sub-agents（多 Agent 编排）、Hooks（生命周期钩子）——每一层都在外围构建 Agent 的能力。而 Unit 6 反其道而行：**掀开引擎盖，用约 220 行 Python 从零搭建一个最小可运行的 Agent 循环**。

这 220 行就是所有 Agent 框架（Claude Code、Codex、OpenCode）的缩影。理解了 Nano Harness，就理解了所有 Agent 的本质：一个循环里调用 LLM、解析输出、执行工具、观察结果、再调用 LLM——直到任务完成。

笔记保留了我学习过程中的疑问和思考。

## 课程章节清单

- [x] Introduction — 为什么要从零理解 Agent Loop
- [x] The Agentic Loop — 调用 LLM → 解析代码 → 执行 → 观察 → 循环
- [x] Code-First Agent — 模型直接输出 Python 代码
- [x] Constrained Tools — list_dir / read_file / write_file / exec_cmd
- [x] Sandboxed Execution — 路径限制、命令白名单、超时控制
- [x] Model via HF Inference Providers — 使用 GLM-5.1 驱动
- [x] Extending the Harness — 添加 web_fetch 和 HF Hub 搜索
- [x] Walking Through the Code — ~220 行源码解读
- [x] Unit 6 测验
- [x] 总结

## 核心概念

### 为什么要从零构建

主流 Agent 框架封装得太好了。用户只看得到输入和输出，看不到中间发生了什么。这就像开一辆自动变速箱的车——你不需要知道换挡逻辑，但如果你想修车或造车，就必须理解它。

Nano Harness 就是为了教学而生的。它的核心价值在于三件事：

- **完整代码只需约 220 行** —— 不比一个 CRUD 脚本复杂，读完只需十分钟
- **所有设计决策是可见的** —— 为什么用代码而不用 JSON？为什么安全边界在工具层？错误怎么恢复？每一步都有清晰的答案
- **零依赖** —— 只用 Python 标准库，不需要 `pip install`，不需要 Docker，拿到就能跑

这个单元使用 Hugging Face Inference Providers 上的 `zai-org/GLM-5.1` 模型驱动循环。只需要一个 `HF_TOKEN`，不需要 GPU。

### Agent Loop 核心循环

Nano Harness 的核心是一个最多运行 50 次的循环。这是整个 AI Agent 最基本的控制结构：

```
                      ┌─────────────────────────────────────┐
                      │         1. User Input / Task         │
                      └──────────────────┬──────────────────┘
                                         │
                                         ▼
                      ┌─────────────────────────────────────┐
                      │      2. Call LLM (with history)      │
                      │         GLM-5.1 via HF Router         │
                      └──────────────────┬──────────────────┘
                                         │
                                         ▼
                      ┌─────────────────────────────────────┐
                      │    3. Parse: 从模型输出提取 Python    │
                      │    正则匹配 ```python ... ``` 块      │
                      └──────────────────┬──────────────────┘
                                         │
                                         ▼
                      ┌─────────────────────────────────────┐
                      │  4. Execute: 在沙箱中 exec() 代码     │
                      │  list_dir / read_file / write_file   │
                      │  exec_cmd / final_answer 可用         │
                      └──────────────────┬──────────────────┘
                                         │
                                         ▼
                      ┌─────────────────────────────────────┐
                      │    5. Observe: 捕获 stdout, stderr,  │
                      │    异常、或者 final_answer 返回值      │
                      └──────────────────┬──────────────────┘
                                         │
                                         ▼
                      ┌─────────────────────────────────────┐
                      │   6. final_answer() called?           │
                      └──────┬──────────────────┬───────────┘
                             │                  │
                            Yes                No
                             │                  │
                             ▼                  ▼
                      ┌──────────┐    ┌──────────────────────┐
                      │  Done!   │    │  Append result to     │
                      │  Return  │    │  message history      │
                      └──────────┘    └──────────┬───────────┘
                                                 │
                                                 ▼
                                       (Back to Step 2)
```

这个循环对应 AI Agents 课程中的 **Thought-Action-Observation** 模式：

| 阶段 | Nano Harness 中的对应 | 描述 |
|------|----------------------|------|
| **Thought** | LLM 输出 Python 代码 | 模型推理下一步该做什么，以代码形式表达 |
| **Action** | `exec()` 执行代码 | 沙箱执行代码，调用工具函数 |
| **Observation** | 捕获 stdout/stderr/异常 | 执行结果作为 observation 喂回给模型 |

每一轮都是"想 -> 做 -> 看结果 -> 再想"的过程。

消息历史（message history）是整个循环的记忆：

```python
messages = [
    {"role": "system", "content": "You are a code-first agent..."},
    {"role": "user", "content": "Inspect workspace and summarize"},
    {"role": "assistant", "content": "list_dir('.')"},
    {"role": "user", "content": "Found: ['README.md', 'src/', 'tests/']"},
    {"role": "assistant", "content": "read_file('README.md')"},
    {"role": "user", "content": "README.md contains: ..."},
]
```

每次调用 LLM 时，整个历史都会被发送。模型可以看到之前发现了什么、哪些方法失败了、接下来该怎么做。这是最朴素的记忆机制——每个 Agent 框架的核心都在做同样的事，只不过规模更大、架构更复杂。

### Code-First Agent 设计

这是 Nano Harness 最关键的设计决策：**模型直接输出 Python 代码，而不是 JSON 工具调用**。

为什么是代码而不是 JSON？三个原因：

1. **表达能力更强** —— 模型可以写 `for file in list_dir("src/")` 循环处理文件，而不是一条一条发 JSON 工具调用。代码天然支持组合、分支、循环
2. **解析更简单** —— 从模型输出中提取 Python 代码块只需要一个正则表达式。不需要定义复杂的 JSON Schema 或工具描述
3. **模型天生擅长写代码** —— LLM 的训练数据中包含海量 Python。模型"用 Python 思考"比"用 JSON 描述动作"更自然，输出质量更高

典型的工作流：

```python
# 模型生成这样的代码：
files = list_dir(".")
for f in files:
    if f.endswith(".py"):
        content = read_file(f, max_chars=500)
        print(f"{f}: {len(content)} chars")
final_answer(f"Scanned {count} Python files")
```

这里的分工非常清晰：**模型是推理引擎，Harness 是执行引擎**。模型只负责"想"（写代码），Harness 只负责"做"（执行代码、捕获结果、管理安全）。这个分离是理解所有 Agent 框架的关键。

### 受限工具集

Nano Harness 只提供 4 个工具，每个工具都有明确的安全边界：

| 工具 | 签名 | 安全约束 |
|------|------|---------|
| `list_dir` | `list_dir(path=".")` | `safe_path()` 路径限制 |
| `read_file` | `read_file(path, max_chars=4000)` | `safe_path()` + 字符上限 `MAX_CHARS=8000` |
| `write_file` | `write_file(path, content)` | `safe_path()` + **默认禁用** (`ALLOW_WRITE=False`) |
| `exec_cmd` | `exec_cmd(args)` | 命令白名单 + 超时控制 |

#### list_dir

```python
def list_dir(path="."):
    p = safe_path(path)
    if not p.is_dir():
        raise NotADirectoryError(str(p))
    return sorted(x.name + ("/" if x.is_dir() else "") for x in p.iterdir())
```

返回的是文件名而不是绝对路径，目录会加上 `/` 后缀。这样可以防止 Agent 通过返回的路径信息推断 workspace 外的文件结构。

#### read_file

```python
def read_file(path, max_chars=4000):
    p = safe_path(path)
    content = p.read_text(encoding="utf-8", errors="replace")
    return clip(content, min(max_chars, MAX_CHARS))
```

双层限制：用户请求的 `max_chars` 和框架级的 `MAX_CHARS=8000`，`min()` 取较小者。即使模型要求 `max_chars=999999`，最终也只能读到 8000 字符，防止 context 被撑爆。`errors="replace"` 确保二进制文件不会导致读取崩溃。

#### write_file

```python
ALLOW_WRITE = False  # 默认禁用

def write_file(path, content):
    if not ALLOW_WRITE:
        raise PermissionError("write_file disabled")
    p = safe_path(path)
    p.parent.mkdir(parents=True, exist_ok=True)
    p.write_text(str(content), encoding="utf-8")
```

写文件默认关闭。Agent 必须显式设置 `ALLOW_WRITE=True` 才能写入磁盘。这个设计体现了"默认拒绝"的安全原则——Agent 不应该在未经确认的情况下修改文件系统。

#### exec_cmd

```python
ALLOW_COMMANDS = ["ls", "cat", "pwd", "echo", "head", "tail", "wc", "rg"]

def exec_cmd(args):
    cmd = shlex.split(args) if isinstance(args, str) else [str(x) for x in args]
    if cmd[0] not in ALLOW_COMMANDS:
        raise PermissionError(f"command not allowed: {cmd[0]}")
    result = subprocess.run(cmd, capture_output=True, text=True, timeout=TIMEOUT_S, shell=False)
    ...
```

只允许白名单中的只读命令：`ls`, `cat`, `pwd`, `echo`, `head`, `tail`, `wc`, `rg`。任何修改系统状态或访问网络的命令（`rm`, `mv`, `curl`, `wget`）都会被拒绝。`shell=False` 防止 shell 注入，`timeout` 防止命令挂起。

### 沙箱执行

安全不是靠提示词（prompt）实现的——如果只告诉模型"不要读取 /etc/passwd"，模型完全可能忽略。Nano Harness 的沙箱是**结构性的（structural）**：安全规则写在代码里，而不是写在提示词里。

#### 路径限制 (safe_path)

```python
WORKSPACE = str(Path.cwd())

def safe_path(user_input):
    p = Path(path)
    if not p.is_absolute():
        p = ws / p
    p = p.resolve()
    if p == ws or ws in p.parents:
        return p
    raise ValueError(f"path escapes workspace: {path}")
```

所有路径都经过 `safe_path()` 过滤：

```
../../../etc/passwd  → resolve() 后不在 WORKSPACE 内 → 拒绝
/etc/passwd          → 绝对路径，不在 WORKSPACE 内  → 拒绝
data/models.txt      → 在 WORKSPACE 内             → 允许
```

`resolve()` 会展开所有 `..` 和符号链接，所以简单的目录遍历攻击无效。

#### 受限的 exec 环境

执行模型代码时，`exec()` 使用白名单式的 globals 字典：

```python
env = {
    "__builtins__": {
        "print": print, "len": len, "range": range,
        "str": str, "int": int, "list": list, "dict": dict,
        "enumerate": enumerate, "zip": zip, "sum": sum, "min": min, "max": max,
        "sorted": sorted, "Exception": Exception, "__import__": __import__,
    },
    "final_answer": final_answer,
    **tools,  # list_dir, read_file, write_file, exec_cmd
}
exec(code, env)
```

模型代码**不能**：
- `import` 标准库以外任何模块（`__import__` 在 whitelist 里但没有模块可用）
- 直接访问文件系统（只能通过工具函数）
- 使用网络（只能通过工具函数）
- 访问父进程变量或全局状态

这个设计让我想起 Unit 2 MCP 中的安全模型——同样是限制工具边界，同样是在"调用路径"上加护栏，而不是依赖模型自我约束。

#### 错误恢复

异常不会让程序崩溃，而是被捕获并作为 observation 返回给模型：

```python
try:
    exec(code, env)
except StopIteration:  # final_answer() 通过 StopIteration 信号终止循环
    return True, done["v"], clip("".join(printed)), None
except Exception as e:
    return False, None, clip("".join(printed)), clip(traceback.format_exc())
```

模型看到错误后会调整策略：找不到文件就 `list_dir` 看看有什么、写权限被拒就换方法、命令超时就分步执行。错误不是终点，而是新的输入。

#### 输出裁剪

```python
def clip(x, n=MAX_CHARS):
    s = str(x)
    return s if len(s) <= n else s[:n] + f"\n...[truncated {len(s) - n} chars]"
```

这个工具看起来很简单，但它解决了一个核心问题：**防止 context 被撑爆**。无论工具返回多少数据，`clip()` 保证每次 observation 不超过 `MAX_CHARS=8000`。这是最原始的 context 管理方式——生产框架有更复杂的 compaction 和 RAG 机制，但原理是一样的。

### ~220 行源码结构

完整的 Nano Harness 代码可以在 [GitHub Gist](https://gist.github.com/burtenshaw/4ec60226d81935b178c581b97b5fe9b1) 上看到。整个架构可以分解为几个清晰的模块：

| 模块 | 约行数 | 功能 |
|------|--------|------|
| 配置常量 | 15 | MODEL, BASE_URL, API_KEY, MAX_STEPS, TIMEOUT_S, ALLOW_COMMANDS 等 |
| 系统提示词 | 8 | 告诉模型怎么输出、有什么工具、约束条件 |
| clip + safe_path | 12 | 工具函数：输出裁剪和路径安全 |
| 工具定义 | 35 | list_dir, read_file, write_file, exec_cmd |
| call_model | 18 | HTTP 调用 HF Inference Providers，带指数退避重试 |
| parse_code | 15 | 从模型输出中提取可执行的 Python 代码 |
| exec_code | 30 | 在受限环境中执行代码，捕获输出和异常 |
| run() 主循环 | 45 | 调用模型 → 解析 → 执行 → 观察 → 循环 |

关键的设计决策：

- **为什么用 `urllib.request` 而不是 `requests`？** —— 零依赖。不需要 `pip install`，任何 Python 环境都能直接运行
- **为什么解析代码用正则？** —— 模型输出格式稳定（`` ```python \n ... \n ``` ``），正则足够且简单。如果模型输出不符合格式，`RETRY_PROMPT` 让模型重试
- **为什么 `temperature=0.2`？** —— 低温度保持输出确定性，减少随机失败
- **为什么 `ALLOW_COMMANDS` 只含只读命令？** —— Agent 只需要观察和分析，不需要修改系统
- **为什么 `MAX_STEPS=50`？** —— 保证循环终止。简单任务 1-3 步完成，探索代码库 5-10 步，复杂调试 15-30 步

完整的主循环只有约 45 行：

```python
def run(task):
    messages = [{"role": "system", "content": SYSTEM_PROMPT}, {"role": "user", "content": task}]
    for step in range(1, MAX_STEPS + 1):
        try:
            text = call_model(messages)        # 1. 调用 LLM
        except Exception as e:
            return 4
        try:
            code = parse_code(text)             # 2. 解析 Python 代码
        except Exception as e:
            messages += [{"role": "assistant", "content": text},
                         {"role": "user", "content": RETRY_PROMPT}]
            continue
        done, value, out, err = exec_code(code) # 3. 执行代码
        messages.append({"role": "assistant", "content": f"```python\n{code}\n```"})
        if done:
            return 0                            # 4. 完成
        messages.append({                       # 5. 追加 observation，继续循环
            "role": "user",
            "content": f"stdout={clip(out)} error={clip(err)} Continue.",
        })
    return 2  # 达到最大步数
```

读到这里，所有之前"黑盒"的 Agent 行为都变得清晰了：Agent 不是"智能体"——它是一个 **while 循环**。

### 扩展 Nano Harness

课程最后展示了如何给 Nano Harness 添加新工具。添加两个工具只需要各自约 15 行代码。

#### web_fetch

```python
def web_fetch(url, max_bytes=10000):
    try:
        with urllib.request.urlopen(url, timeout=TIMEOUT_S) as response:
            content = response.read(max_bytes + 1)
            if len(content) > max_bytes:
                content = content[:max_bytes] + b"\n...[truncated]"
            return content.decode("utf-8", errors="replace")
    except Exception as e:
        return f"Error: {type(e).__name__}: {str(e)}"
```

URL 没有"路径安全"的概念，所以改用字节上限 + 超时来控制风险。安全模式是一样的：限制输入（字节上限）、限制执行（超时）、错误作为字符串返回。

#### hf_search

```python
def hf_search(query, resource_type="models", limit=5):
    if not API_KEY:
        return "Error: HF_TOKEN not set"
    try:
        url = f"https://huggingface.co/api/{resource_type}?search={query}&limit={limit}"
        req = urllib.request.Request(url, headers={"Authorization": f"Bearer {API_KEY}"})
        with urllib.request.urlopen(req, timeout=TIMEOUT_S) as response:
            data = json.loads(response.read())
            return [{"id": item.get("id"), "downloads": item.get("downloads", 0)}
                    for item in data[:limit]]
    except Exception as e:
        return f"Error: {str(e)}"
```

API Key 只从环境变量读取，不在代码中硬编码。结果数量通过 `limit` 限制。异常被捕获为字符串，模型可以据此调整。

加入这两个工具后，模型就能做更多事了：

```python
# Agent 现在可以：
content = web_fetch("https://example.com")
results = hf_search("bert", resource_type="models", limit=5)
```

课程还给了让读者自己尝试的扩展练习：

1. **git_log** —— 把 `git` 加入命令白名单，获取最近提交历史
2. **json_parse** —— 安全的 JSON 解析工具
3. **compute_stats** —— 简单的数值统计工具

每个扩展都在问同一个问题：**"这个新工具有什么风险？怎么在工具边界上控制这些风险？"**

### 六个单元的完整图景

走完六个单元回头看，Context Course 的脉络非常清晰：

```
Unit 1: Skills        → 给 Agent "知识" （告诉它怎么做）
Unit 2: MCP           → 给 Agent "工具" （让它能做事）
Unit 3: Plugins       → 打包知识和工具   （可分发的能力单元）
Unit 4: Sub-agents    → 多 Agent 协作     （分工与编排）
Unit 5: Hooks         → 监控和约束        （治理与安全）
Unit 6: Nano Harness  → 从零实现引擎      （理解本质）
```

Nano Harness 是"收网"的一章。当你理解了约 220 行就能搭出一个可运行的 Agent，你就不再需要"相信"任何框架了——你知道所有框架本质上都在做同一件事：**管理一个循环，让 LLM 能做事、能看结果、能调整**。

- Skills + MCP 给这个循环提供"能力"
- Plugins 打包这些能力以便分发
- Sub-agents 让能力可以分工协作
- Hooks 监控能力的执行
- 而 Nano Harness 展示的就是这个循环本身

## 思考与疑问

1. **Code-First 与 JSON Tool Calls 的取舍。** Nano Harness 选择 Python 代码作为模型输出，解析极简、表达力强。但生产框架几乎都使用 JSON 工具调用。我理解 Code-First 的教学优势（10 行代码展示完整循环），但真实场景中结构化 JSON 的可审计性、可验证性、可约束性是代码执行难以替代的。一个极端的例子：如果模型输出 `__import__('os').system('rm -rf /')`，在 Code-First 模式中 exec() 可能阻止不了（取决于 builtins 配置），而在 JSON 模式中这个调用根本不可能是有效的工具调用。

2. **exec() 沙箱的安全性上限。** Nano Harness 的 `exec()` 沙箱有很多已知逃逸方式（通过 `().__class__.__bases__[0].__subclasses__()` 可以拿到危险模块）。课程也明确警告了这是学习工具而非生产代码。真正的 Agent 沙箱需要操作系统级隔离——这也是为什么生产框架要么运行在 Docker 中，要么使用 E2B 这样的托管沙箱服务。

3. **Message History 没有上下文管理。** Nano Harness 把全部历史塞进每次 LLM 调用，不做任何压缩或淘汰。这是教学简化。实际 Agent 需要 context compaction（摘要旧内容）、episodic memory（跨会话记忆）、RAG（按需检索）等技术。好消息是，前五个单元其实就是在教如何做这些——Skills 就是静态上下文，MCP 就是动态上下文，Sub-agents 就是分治上下文压力。

4. **工具集大小与模型行为的相互作用。** 4 个工具够吗？对于 "Inspect and analyze" 类任务，够。但一旦需要网络请求、文件写入、执行任意命令，就必须扩展工具集。有趣的是，工具集本身也在塑造模型的行为——更多工具意味着更大的决策空间，也意味着更多出错的路径。Nano Harness 的 4 个工具之所以有效，恰恰是因为它们少。

5. **约 220 行在什么时候会不够。** 当需要持久化消息历史、流式响应、多模型切换、长上下文管理、并发请求、用户确认流程时，220 行会迅速膨胀为数千行。生产框架的复杂性不是"多余的膨胀"——每一层抽象都在解决一个真实问题。但理解了这个最小循环后，每层抽象的成本和收益就变得可衡量了：你永远可以问，"这一层代码到底在 Agent 循环的哪个步骤加入了什么价值？"
