---
title: Bonus 3 - 游戏中的 AI Agent
publish: false
status: 🟢 已完成
course_url: https://hf.co/learn/agents-course/bonus-unit3/introduction
publish: true
---

这个 Bonus Unit 是 Hugging Face Agents Course 的最后一个附加单元，内容非常有意思——用 LLM Agent 玩 Pokemon 对战。整个单元的实践性很强，从理解 LLM 在游戏中的应用现状，到亲手构建一个能参与在线对战的 Pokemon Agent。笔记记录了核心概念、架构分析和完整的代码实践。

## 课程章节清单

- [x] 介绍
- [x] LLM 在游戏中的应用现状（State of the Art）
- [x] 从 LLM 到 AI Agent
- [x] 构建你的 Pokemon 对战 Agent
- [x] 启动 Agent 对战
- [x] 总结

## 核心概念

### 游戏 AI 的发展：从规则脚本到自主 Agent

游戏 AI 的演化路径是一条从"确定性"走向"自主性"的渐进线：

| 阶段 | 特点 | 局限 |
|------|------|------|
| 规则引擎 / 状态机 | 预设行为，完全可控 | 脚本化，可预测，无适应性 |
| 行为树（Behavior Tree） | 模块化决策，复杂行为组合 | 仍需手动设计行为逻辑 |
| 强化学习（RL） | 从经验中学习策略 | 训练成本高，泛化能力有限 |
| LLM 增强的 NPC | 自然对话，动态文本生成 | NPC 仍然被动响应，不会自主行动 |
| Agentic AI（Agent） | 自主决策、规划、适应环境 | 速度慢，不适用于实时场景 |

课程引用了一个非常直观的对比来区分 LLM 和 Agent 在游戏中的角色：

> 想象一个经典 RPG 里的 NPC：
> - 用 LLM：NPC 能用更自然、更多样的方式回答你的问题。但 NPC 仍然是**静态**的——除非你主动和它互动，否则它什么也不会做。
> - 用 Agentic AI：NPC 可以**自主决定**去求助、设陷阱、或者干脆避开你——即使你根本没有和它交互。

这个转变非常关键。我们正在从"脚本化的响应者"走向"游戏世界中的自主行动者"。Agentic AI 赋予了 NPC 三个核心能力：

- **Autonomy（自主性）**：基于游戏状态独立做出决策
- **Adaptability（适应性）**：根据玩家的行动实时调整策略
- **Persistence（持续性）**：记住过去的交互，影响未来的行为

这彻底改变了 NPC 的定位——从被动反应实体转变为游戏世界中的主动参与者。

### LLM 在游戏中的现状：四个代表性案例

课程介绍了四个已经在实践中使用 LLM 的游戏/技术演示，但有意思的是——它们严格来说还没有用到 Agent。

**1. Covert Protocol（NVIDIA + Inworld AI）**

2024 年 GDC 上公布的演示。玩家扮演私家侦探，与 AI 驱动的 NPC 进行实时对话互动，NPC 的反应会影响剧情走向。基于 Unreal Engine 5，使用 NVIDIA ACE（Avatar Cloud Engine）和 Inworld AI 技术。

**2. NEO NPCs（Ubisoft）**

同样是 GDC 2024 上，育碧展示了他们的原型。NPC 能够感知环境、记住过去的交互、与玩家进行有意义的对话。目标是创造更沉浸、更 responsive 的游戏世界。

**3. Mecha BREAK（NVIDIA ACE 集成）**

一款多人机甲对战游戏，集成了 NVIDIA ACE 技术。玩家可以用自然语言与 NPC 交互，NPC 还能通过摄像头识别玩家和物体——集成了 GPT-4o。

**4. Suck Up!（Proxima Enterprises）**

真正上架销售的商业化游戏。玩家扮演吸血鬼，需要说服 AI 驱动的 NPC 邀请你进入他们的房子。每个角色都由生成式 AI 驱动，导致每次游玩的交互都不可预测。

课程的点评值得注意：这些案例展示的是 LLM 在游戏中的应用，但它们本质上还不是 Agent——NPC 仍然是被动等待玩家触发交互，不会主动规划或执行目标。这正是 Agentic AI 要填补的空白。

### 从 LLM 到 Game Agent：Agentic AI 的独特价值

LLM 让 NPC 会说人话了，但 Agent 让 NPC 会**做事**了。区别在于 Agent 具备自主规划、决策和行动的能力，可以和游戏环境进行多轮交互。

但课程也坦诚指出了当前最大的局限：**速度**。

Agent 的推理和规划过程会引入明显的延迟。以 Claude Plays Pokemon 为例，考虑到思考所需的 token 数量，加上执行动作所需的 token，会发现现有的解码策略根本无法支撑实时游戏。大多数游戏需要 30 FPS 的帧率，意味着 AI Agent 每秒需要行动 30 次——以现在的 Agent 节奏来说完全不现实。

但这恰恰说明了**回合制游戏**的独特价值：每回合给 AI 留出了充分的思考时间，可以做出真正有策略性的决策。Pokemon 对战就是最典型的例子。

这个观察其实是 Agent 落地的一个重要启示：**不是所有场景都适合 Agent。在需要毫秒级响应的场景中，传统规则引擎依然不可替代。Agent 最适合的是那些"慢思考、快执行"的任务——给足够的时间推理，然后一次性执行。**

### Pokemon 对战 Agent 架构分析

这是课程的核心实践部分。整个系统由四个组件构成：

**1. Poke-env**

一个 Python 库，最初由 Haris Sahovic 构建用于训练强化学习 Pokemon 机器人。课程将其改造为 Agentic AI 的基础。它提供了 `Player` 基类，封装了与 Pokemon Showdown 通信的所有底层细节。

**2. Pokemon Showdown**

开源的 Pokemon 对战模拟器。Agent 在这里进行实时对战，就像人类玩家一样，每回合选择技能或切换 Pokemon。课程搭建了一个专用服务器，所有参与者都能在上面互相挑战。

**3. LLMAgentBase**

核心桥梁类，继承自 Poke-env 的 `Player`。它的职责是把 LLM 连接到对战环境，处理输入输出的格式化，维护对战上下文。

关键内部方法：

| 方法 | 功能 | 返回类型 |
|------|------|---------|
| `_format_battle_state(battle)` | 将当前对战状态转为字符串（供 LLM 理解） | `str` |
| `_get_llm_decision(battle_state)` | 抽象方法——查询 LLM 并解析返回结果 | `Dict[str, Any]` |
| `_find_move_by_name(battle, move_name)` | 根据名称定位对战中的技能 | `Optional[Move]` |
| `_find_pokemon_by_name(battle, pokemon_name)` | 根据名称定位可切换的 Pokemon | `Optional[Pokemon]` |
| `choose_move(battle)` | 主要决策入口，每回合调用一次 | `str` |

这个设计模式很有意思：`LLMAgentBase` 是一个典型的**模板方法**模式——框架固定了决策流程（格式状态 -> 查询 LLM -> 解析决策 -> 执行动作），但具体的 LLM 查询逻辑留给子类实现。这意味着你可以用完全不同的模型（OpenAI、Mistral、Gemini 甚至本地模型）来驱动 Agent，只要实现 `_get_llm_decision` 这一个方法。

**4. TemplateAgent**

一个待填充的模板，继承了 `LLMAgentBase`。你需要实现 `_get_llm_decision` 方法，在其中定义 system prompt、构造 API 调用、解析 LLM 的返回值。

### 决策循环与失败兜底

课程的代码中有非常健壮的失败处理机制。当 LLM 的决策无法执行时（比如选择了当前不可用的技能），Agent 不会崩溃，而是按这个优先级降级：

1. LLM 返回了有效的 `choose_move` 或 `choose_switch` -> 执行
2. LLM 选择的技能/精灵在当前对战中不可用 -> 记录错误原因
3. LLM 调用了未知函数 -> 记录错误
4. API 调用失败 -> 记录错误
5. 兜底方案：如果还有可用技能或精灵，随机选择；否则使用默认动作（Struggle）

这一整套错误处理逻辑本身就值得学习——实际部署中 LLM 必然会犯错，优雅地降级比追求完美更重要。

### 对战系统与竞赛机制

课程提供了一个完整的在线对战平台：

1. 参与者将自己的 Agent 部署到 Hugging Face Spaces
2. 在专用的 Pokemon Showdown 服务器上注册
3. 通过 Web 界面向其他 Agent 发送对战邀请
4. 实时观看 AI vs AI 的对战

Leaderboard 的存在让这个实践有了竞争性——你可以不断改进 Agent 的策略，争取更好的排名。

## 代码实践

### 构建 Pokemon 对战 Agent

**LLMAgentBase 核心代码**

这是课程提供的基类，负责格式化对战状态、调用 LLM、解析决策、执行对战动作：

```python
from poke_env.player import Player
from typing import Dict, Any, Optional

class LLMAgentBase(Player):
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.standard_tools = STANDARD_TOOL_SCHEMA
        self.battle_history = []

    def _format_battle_state(self, battle: Battle) -> str:
        # 格式化当前对战双方的 Pokemon 状态、可用技能、天气地形等
        active_pkmn = battle.active_pokemon
        active_pkmn_info = f"Your active Pokemon: {active_pkmn.species} " \
                           f"(Type: {'/'.join(map(str, active_pkmn.types))}) " \
                           f"HP: {active_pkmn.current_hp_fraction * 100:.1f}% " \
                           f"Status: {active_pkmn.status.name if active_pkmn.status else 'None'} " \
                           f"Boosts: {active_pkmn.boosts}"

        opponent_pkmn = battle.opponent_active_pokemon
        opp_info_str = "Unknown"
        if opponent_pkmn:
            opp_info_str = f"{opponent_pkmn.species} " \
                           f"(Type: {'/'.join(map(str, opponent_pkmn.types))}) " \
                           f"HP: {opponent_pkmn.current_hp_fraction * 100:.1f}% " \
                           f"Status: {opponent_pkmn.status.name if opponent_pkmn.status else 'None'} " \
                           f"Boosts: {opponent_pkmn.boosts}"

        # 格式化可用技能和可切换 Pokemon
        available_moves_info = "Available moves:\n" + ...
        available_switches_info = "Available switches:\n" + ...

        state_str = f"{active_pkmn_info}\n{opponent_pkmn_info}\n\n" \
                    f"{available_moves_info}\n\n{available_switches_info}\n\n" \
                    f"Weather: {battle.weather}\nTerrains: {battle.fields}\n" \
                    f"Your Side Conditions: {battle.side_conditions}\n" \
                    f"Opponent Side Conditions: {battle.opponent_side_conditions}"
        return state_str.strip()

    async def choose_move(self, battle: Battle) -> str:
        battle_state_str = self._format_battle_state(battle)
        decision_result = await self._get_llm_decision(battle_state_str)
        # 解析决策并执行，失败则回退到随机动作
        ...

    async def _get_llm_decision(self, battle_state: str) -> Dict[str, Any]:
        raise NotImplementedError("Subclasses must implement _get_llm_decision")
```

**TemplateAgent——你需要填充的模板**

```python
class TemplateAgent(LLMAgentBase):
    """Uses Template AI API for decisions."""
    def __init__(self, api_key: str = None, model: str = "model-name", *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.model = model
        self.template_client = TemplateModelProvider(api_key=...)
        self.template_tools = list(self.standard_tools.values())

    async def _get_llm_decision(self, battle_state: str) -> Dict[str, Any]:
        system_prompt = (
            "You are a Pokemon battle expert. Given the current battle state, "
            "choose the best action using the available tools. "
            "Consider type matchups, stat boosts, and opponent's possible moves."
        )
        user_prompt = f"Current battle state:\n{battle_state}"

        try:
            response = await self.template_client.chat.completions.create(
                model=self.model,
                messages=[
                    {"role": "system", "content": system_prompt},
                    {"role": "user", "content": user_prompt},
                ],
            )
            # 解析 LLM 返回的工具调用
            message = response.choices[0].message
            return {"decision": {"name": function_name, "arguments": arguments}}
        except Exception as e:
            return {"error": f"Unexpected error: {e}"}
```

**课程还提供了三个完整示例：OpenAI、Mistral、Gemini 模型的具体实现。** 实际使用中只需要替换 client 和 model 名称即可。

### 启动对战

部署流程：

1. **复制 Hugging Face Space** — 在课程提供的 Space 页面点击 "Duplicate this Space"
2. **添加 Agent 代码** — 将你的 Agent 实现粘贴到 `agent.py`
3. **注册 Agent** — 在 `app.py` 的下拉菜单中选择中添加 Agent 名称和逻辑
4. **选择 Agent** — 部署后，在界面的 "Select Agent" 下拉菜单中选择你的 Agent
5. **输入 Pokemon Showdown 用户名** — 确保与对战服务器中的名字一致
6. **发送对战邀请** — 点击 "Send Battle Invitation"，向对手发出挑战
7. **观看对战** — 实时观看 AI vs AI 的精彩对决

## 思考与疑问

- **游戏作为 Agent 测试床的独特价值**：游戏提供了高度结构化的环境、清晰的胜负判定、可复现的测试场景和即时反馈——这些在真实世界的 Agent 应用中很难同时具备。用游戏来验证 Agent 能力是一种低成本、高效率的方式，类似 AlphaGo 通过围棋证明了强化学习的潜力。

- **回合制 vs 实时：Agent 适用场景的本质判断标准**：课程提到 Agent 太慢不适合实时游戏，但这个洞察可以推得更广——Agent 的适用性取决于任务的时间粒度。需要毫秒级响应的场景（自动驾驶、交易系统）仍然需要传统方法；而可以容忍秒级以上延迟的场景（代码生成、数据分析、客服）才是 Agent 的主战场。

- **"LLM in games" vs "Agent in games" 的区分比我想象的更清晰**：课程用了四个案例来展示"用 LLM 但还没用 Agent"的现状，这个对比非常有说服力。很多产品宣传把"接入了 LLM"等同于"智能"，但实际只是让原有系统会说人话，并没有改变核心的决策机制。Agent 才是真正的范式转移。

- **失败兜底设计对 LLM 应用是刚需**：课程代码中的 fallback 机制（LLM 不可用时回退到随机选择）看起来简单，其实是 Agent 工程中极易被忽视的关键环节。LLM 是不稳定的，prompt 再精心设计也避免不了输出不可用的结果。如果生产级 Agent 没有设计好降级策略，上线后必然会出问题。

- **对比 Unit 1 的 Tool 机制**：Unit 1 讲了 `@tool` 装饰器、Python 自省等工具定义方式，而这节课的 Agent 工具是 `choose_move` 和 `choose_switch` 两个预定义函数。本质上是一样的：工具是 LLM 与外部环境交互的接口。只不过游戏场景中的工具被固定为回合制对战的几个动作，更加结构化。这让我想到，Agent 工具的设计其实是一个"约束空间"的问题——工具定义得越精细，LLM 的自由度越低，但可控性和可靠性越高。

- **从 Pokemon Agent 到现实 Agent 的泛化**：这个场景虽然是一个游戏，但它几乎涵盖了真实 Agent 的所有核心要素：状态感知 -> 推理规划 -> 工具选择 -> 执行 -> 观察反馈 -> 迭代。Pokemon 对战本质上是一个多步决策问题（sequential decision-making），和现实中的很多 Agent 任务（客服对话、代码调试）有相同的结构。

## 关键收获

1. **Agentic AI 在游戏中的应用才刚刚起步**，目前行业实践主要停留在"LLM 增强的 NPC"阶段，真正的 Agent 自主决策还有很大的探索空间。回合制游戏（如 Pokemon）是 Agent 入场的理想切入点，因为推理延迟在这个场景下不是瓶颈。

2. **Agent 架构的核心是状态表示 + 工具约束 + 失败兜底**——缺一不可。State 格式决定模型理解质量，Tool 定义决定行动能力边界，Fallback 决定系统鲁棒性。

3. **Poke-env + LLMAgentBase 的设计模式值得借鉴**：模板方法模式将固定的决策流程和可变的 LLM 查询逻辑解耦，使得切换底层模型变得非常简单（只需实现一个方法），这是 Agent 框架设计的有益参考。

4. **在线对战平台（HF Spaces + Pokemon Showdown）展示了 Agent 竞技的完整链路**：从本地开发到云端部署，再到与其他 Agent 实时对战，最后通过 leaderboard 量化比较——这个闭环本身就是 Agent 评估的一个范本。

5. **"慢思考、快执行"可能是 Agent 落地的正确节奏**：给 LLM 足够的推理时间，但执行阶段要足够快（甚至规则化）。这启发我在设计其他 Agent 系统时，应该把复杂的推理和快速的执行解耦成两个独立的阶段。
