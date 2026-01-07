---
timezone: UTC+8
---

# Leo

**GitHub ID:** 7fpb5574ck-maker

**Telegram:** @runes_leo

## Self-introduction

AI × Crypto 实践者，关注 AI Agent、自动化与工具构建，正在用 AI 提升个人效率与真实执行能力，偏好从真实问题出发做产品。

## Notes

<!-- Content_START -->
# 2026-01-07
<!-- DAILY_CHECKIN_2026-01-07_START -->
````markdown
# Day 11: Google Managed MCP (Connecting to Services)

> **日期**: 2026-01-07
> **主题**: Model Context Protocol (MCP) & Google Cloud Action API
> **状态**: ✅ 完成

---

## 🎯 核心目标

1.  **理解 MCP (Model Context Protocol)**: 为什么它是 AI Agent 连接世界的标准？
2.  **Google Managed MCP**: 如何不写复杂的 Server 代码，直接让 Agent 拥有安全连接数据的能力。

---

## 🧠 概念预热

### 1. 什么是 MCP？(The "USB" for AI)

在 MCP 出现之前，每个 Agent 都要自己写特定的 integration 代码来连接 Notion, Github, Slack...
MCP 定义了一种**标准协议**：
*   **MCP Server**: 提供资源 (Resources)、提示 (Prompts) 和工具 (Tools)。
*   **MCP Client (Your Agent)**: 通过标准方式消费这些能力。

这就好比 **USB 接口**：你不需要为每个鼠标单独焊死在电脑主板上，只要它是 USB 的，插上就能用。

### 2. 这个 Protocol 解决了什么问题？

*   **Fragmentation**: 不再需要 `langchain-notion`, `llama-index-notion`... 只需要一个 `mcp-server-notion`。
*   **User Context**: 让 AI 安全地访问你的本地文件、数据库或私有 SaaS 数据。

### 3. "Google Managed" 是什么意思？

通常运行 MCP Server 需要你自己在本地或服务器跑一个进程 (`npx @modelcontextprotocol/server-notion`)。
Google 的 Managed MCP (通常指 Vertex AI Extensions 或其生态集成) 试图让这个过程更“云原生”，或者就在 ADK 层面无缝支持。

*(注：由于 MCP 是 Anthropic 发起的开源标准，Google ADK 的支持方式可能是通过通用 Tool 接口兼容，或者特定的 Adapter。我们将探索 ADK 如何作为 MCP Client 使用)*

---

## 💻 代码实现 (day11/agent.py)

利用 `McpToolset` 和 `uvx` 运行 Python 版 MCP SQLite Server：

```python
from google.adk.tools import McpToolset
from google.adk.tools.mcp_tool.mcp_session_manager import StdioConnectionParams, StdioServerParameters

sqlite_mcp_toolset = McpToolset(
    connection_params=StdioConnectionParams(
        server_params=StdioServerParameters(
            command="uvx",
            args=["mcp-server-sqlite", "--db-path", db_path],
        )
    ),
    tool_name_prefix="db_" 
)
```

## 📊 验证结果

运行 `day11/test_mcp.py` 结果表明 Agent 完美继承了 MCP 工具：

1.  **动态发现**: Agent 自动识别并调用了 `db_list_tables`。
2.  **数据交互**: Agent 成功从 SQLite `users` 表中读取了初始化的测试数据。

---

## 🔗 参考资源
- [Model Context Protocol (MCP) Official Site](https://modelcontextprotocol.io/)
- [ADK McpToolset Documentation](https://google.github.io/adk-python/tools/mcp/)
````
<!-- DAILY_CHECKIN_2026-01-07_END -->

# 2026-01-06
<!-- DAILY_CHECKIN_2026-01-06_START -->

````markdown
# Day 10: Big Context ≠ Better Memory (Caching & Compaction)

> **日期**: 2026-01-06
> **主题**: Context Caching + Event Compaction
> **状态**: ✅ 完成

---

## 🎯 一句话总结

> **用"缓存"换取速度，用"压缩"换取记忆。**

---

## 🧠 核心概念：通俗理解

### 1. Context Caching (上下文缓存) —— "书摊开在桌上"

*   **问题**: 就像每次回答问题前都要重新从头阅读一本 500 页的书（Token 计费贵，处理慢）。
*   **解决**: 把处理过的长文档（Context）**缓存**在服务端（书摊开放在桌上）。
*   **效果**: 下次提问时，直接基于缓存回答，**延迟极低，成本大幅下降**。
*   **适用**: 法律文档问答、长篇小说续写、拥有复杂 System Prompt 的角色扮演。

| 配置项 | 作用 | 推荐值 |
|:---|:---|:---|
| `min_tokens` | 只有超过这个长度才触发缓存 | 32,768 (32k) |
| `ttl_seconds` | 缓存多久不因使用而失效 | 3600 (1小时) |

### 2. Event Compaction (历史压缩) —— "定期写会议纪要"

*   **问题**: Agent 的脑容量（Context Window）有限。如果对话无限变长，最早的记忆（如用户的核心指令）会被挤出并遗忘。
*   **解决**: 定期暂停，回顾最近的 N 轮对话，将其**总结**成一段摘要（Summary），替换掉原始的废话。
*   **效果**: 腾出了脑容量，同时**保留了关键信息**。
*   **核心机制**: 原始对话 A, B, C, D -> 压缩为 [Summary(A,B)] + C, D。

| 配置项 | 作用 | 推荐值 |
|:---|:---|:---|
| `compaction_interval` | 每隔几轮对话触发一次压缩 | 5~10 |
| `overlap_size` | 压缩时保留最近几条原始消息 | 2 (保证上下文连贯) |

---

## 💻 代码实现 (day10/agent.py)

```python
from google.adk.apps import App
from google.adk.apps.app import EventsCompactionConfig
from google.adk.agents.context_cache_config import ContextCacheConfig

day10_app = App(
    name="day10_app",
    root_agent=reader_agent,
    
    # 🌟 启用缓存
    context_cache_config=ContextCacheConfig(
        min_tokens=32768, 
        ttl_seconds=3600
    ),
    
    # 🌟 启用压缩
    events_compaction_config=EventsCompactionConfig(
        compaction_interval=5,
        overlap_size=2
    )
)
```

---

## 📊 验证结果

通过 `day10/test_compaction.py` 我们验证了：

1.  **Caching 生效**: 日志显示 Agent 检查了 context 长度（当前测试文本较短，提示 `Previous request too small`，证明检查逻辑在线）。
2.  **Compaction 生效**: 
    - 模拟了 5 轮对话。
    - 在第 3 轮附近，观察到了额外的 LLM 请求（摘要生成）。
    - 最终历史记录 (`session.events`) 包含了 Summary 事件，而非全是原始 Message。

---

## 🔗 参考资源
- [ADK Context Caching](https://google.github.io/adk-python/context-caching/)
- [ADK Event Compaction](https://google.github.io/adk-python/memory/compaction/)
````
<!-- DAILY_CHECKIN_2026-01-06_END -->

# 2026-01-05
<!-- DAILY_CHECKIN_2026-01-05_START -->


````markdown
# Day 09: Undo Buttons for Agents (Time Travel)

> **日期**: 2026-01-05  
> **主题**: Session Rewind + Resumability  
> **状态**: ✅ 完成

---

## 🎯 一句话总结

> **给 Agent 装上「撤销」和「断点续传」按钮**

---

## 🧠 核心概念：两个比喻

| 比喻 | ADK 概念 | 作用 | 代码入口 |
|------|----------|------|----------|
| ⏪ 撤销 | `rewind_async()` | 回滚到历史某一轮对话 | `runner.rewind_async()` |
| 💾 断点续传 | `ResumabilityConfig` | 中断后从失败点恢复 | `App(resumability_config=...)` |

---

## ⏪ 撤销：Session Rewind

**场景**：用户说"刚才那个答案不对，帮我重新回答"

### 工作原理

1. **指定回滚点**：通过 `invocation_id` 标识要撤销的那一轮对话
2. **计算逆向 Delta**：ADK 遍历事件历史，计算 State 和 Artifact 的回滚差值
3. **追加 Rewind Event**：不删除历史，而是追加一个 `rewind_event`，后续 LLM 不再「看」被撤销的内容

```python
from google.adk import Runner

# 回滚到某个 invocation 之前
await runner.rewind_async(
    user_id="user_123",
    session_id="session_abc",
    rewind_before_invocation_id="inv_to_undo"  # ← 要撤销的 invocation_id
)
```

### 限制 (重要!)

| 限制 | 说明 |
|------|------|
| ❌ 不回滚全局状态 | `app:` 和 `user:` 前缀的 State 不会被撤销 |
| ❌ 不回滚外部系统 | 如果 tool 调用了外部 API（如下单），需自己处理回滚 |
| ⚠️ 非原子操作 | 回滚中途若有其他写入，可能不一致 |

---

## 💾 断点续传：Resumability

**场景**：Agent 执行到一半，网络断了或服务重启了

### 启用方式

```python
from google.adk.apps import App, ResumabilityConfig

app = App(
    name="my_resumable_app",
    root_agent=my_agent,
    resumability_config=ResumabilityConfig(is_resumable=True)  # ← 核心配置
)
```

### 恢复执行

```python
# 通过 invocation_id 恢复之前中断的执行
async for event in runner.run_async(
    user_id="user_123",
    session_id="session_abc",
    invocation_id="interrupted_inv_id"  # ← 指定要恢复的 invocation
):
    print(event)
```

### 限制

| 限制 | 说明 |
|------|------|
| ⚠️ 至少一次执行 | 恢复时 tool 可能被重复调用，需保证幂等性 |
| ❌ 内存状态丢失 | 临时变量、`temp:` 前缀 State 会丢失 |

---

## 📊 对比总结

| 特性 | Rewind | Resumability |
|------|--------|--------------|
| 目的 | 回滚历史 | 从中断恢复 |
| 触发方式 | 用户主动 | 系统自动检测 |
| 数据影响 | 追加逆向事件 | 继续追加事件 |
| tool 行为 | 无重复调用 | 可能重复调用 |

---

## 💡 最佳实践

1. **Tool 设计幂等性**：恢复时 tool 可能被重复调用
2. **避免回滚活跃 Session**：确保没有并发写入
3. **全局状态用 `app:` / `user:`**：这些不会被 rewind 影响

---

## ✏️ 自测题（明天复习用）

1. 如何让用户"撤销"上一轮对话？
2. `ResumabilityConfig` 在哪里配置？
3. Rewind 不会回滚哪些 State？

---

## 📂 今日产出

- `day09/agent.py` - Resumable App 示例
- `day09/test_rewind.py` - Rewind 测试脚本 (概念性)
- `logs/day09/learning_notes.md` - 本文件

---

## 🔗 参考资源

- [ADK Rewind Sessions](https://google.github.io/adk-python/sessions/rewind-sessions/)
- [ADK Resuming Agents](https://google.github.io/adk-python/resuming-agents/)
````
<!-- DAILY_CHECKIN_2026-01-05_END -->

# 2026-01-04
<!-- DAILY_CHECKIN_2026-01-04_START -->



````markdown
# Day 08: Effective Context Management (ADK Layers)

> **日期**: 2026-01-04  
> **主题**: ADK 的上下文分层管理  
> **状态**: ✅ 完成

---

## 🎯 一句话总结

> **教 Agent 用便签纸传话 + 用笔记本记长期偏好 + 用文件柜存文档**

---

## 🧠 核心概念：三个比喻

| 比喻 | ADK 概念 | 代码示例 | 生命周期 |
|------|----------|----------|----------|
| 📝 便签 | `output_key` + `{state_key}` | Agent A 写，Agent B 读 | 当前会话 |
| 📓 笔记本 | `user:` 前缀 | `state["user:preference"]` | **跨会话永久** |
| 🗄️ 文件柜 | Artifacts | `save_artifact()` / `load_artifact()` | 按需配置 |

---

## 📝 便签：Agent 之间传话

**场景**：Summarizer 生成摘要 → Translator 读取并翻译

```python
# Agent A: 写便签
summarizer = Agent(
    name="summarizer",
    instruction="Summarize the text.",
    output_key="summary"  # ← 输出自动保存到 state["summary"]
)

# Agent B: 读便签
translator = Agent(
    name="translator",
    instruction="Translate this: {summary}"  # ← 自动从 state 注入
)

# 组合为流水线
pipeline = SequentialAgent(sub_agents=[summarizer, translator])
```

---

## 📓 笔记本：跨会话记忆

**场景**：记住用户偏好，下次对话还知道

```python
def remember_preference(preference: str, tool_context: ToolContext):
    # user: 前缀 = 跨会话持久化
    tool_context.state["user:preference"] = preference
    return {"status": "saved"}
```

**State 前缀速查**：
| 前缀 | 作用域 | 用途 |
|------|--------|------|
| (无) | 当前会话 | 工作流临时数据 |
| `user:` | 跨会话 | 用户偏好 |
| `app:` | 全局 | 应用设置 |
| `temp:` | 仅当前执行 | 一次性中间结果 |

---

## 🗄️ 文件柜：存储二进制文件

**场景**：生成报告并保存为 Artifact

```python
async def generate_report(topic: str, tool_context: ToolContext):
    content = f"# Report: {topic}\n..."
    artifact = types.Part.from_bytes(data=content.encode(), mime_type="text/markdown")
    
    # ⚠️ 异步方法，必须 await
    version = await tool_context.save_artifact(filename="report.md", artifact=artifact)
    return {"version": version}
```

---

## ⚠️ 今日踩坑

1. **`adk web` 启动位置**：必须从项目根目录运行，否则找不到 agent
2. **异步方法忘记 `await`**：`save_artifact()` 和 `load_artifact()` 是异步的，不加 `await` 会返回 `coroutine` 对象导致序列化错误

---

## ✏️ 自测题（明天复习用）

1. Agent A 的输出怎么让 Agent B 看到？
2. 怎么让数据在用户换个对话后还记得？
3. 存文件用什么方法？要注意什么？

---

## 📂 今日产出

- `day08/agent.py` - SequentialAgent Pipeline 示例
- `day08/tools.py` - User State + Artifacts 工具
- `day08/.env` - API 配置

---

## 🔗 参考资源

- [ADK Session & State 官方文档](https://google.github.io/adk-python/sessions/)
- [ADK Artifacts 官方文档](https://google.github.io/adk-python/artifacts/)
````
<!-- DAILY_CHECKIN_2026-01-04_END -->

# 2026-01-03
<!-- DAILY_CHECKIN_2026-01-03_START -->




# **📅 2026-01-03 Day 07 学习日记**

## **📝 学习主题：LLMs Can Execute Code (Autonomous Problem Solving)**

今天是 Week 1 的最后一天，我们将学习如何让 AI Agent **自主编写并执行代码** 来解决问题。通过

```
BuiltInCodeExecutor
```

，LLM 获得了一个强大的计算器和逻辑执行引擎，真正实现了 "Neuro-Symbolic"（神经符号）AI。

### **🧠 核心概念**

**1\. 为什么需要代码执行能力？**

LLM 的"思考"是基于概率的，擅长语言生成，但有以下先天局限：

-   **精确计算**: 复杂的数学运算容易出错（LLM 是"文科生"）。
    
-   **数据处理**: 处理大量 CSV/JSON 数据时，Token 上下文有限且容易产生幻想。
    
-   **可复现性**: 自然语言推理是不可控的，而代码逻辑是确定性的。
    
-   **复杂算法**: 排序、搜索、图遍历等算法用代码实现远比 Prompt 高效。
    

**关键洞见**:

-   **Prompt**: 负责 "Intent" (意图) 和 "Design" (设计)。
    
-   **Code**: 负责 "Action" (行动) 和 "Precision" (精确)。
    

**2\. ADK 中的 Code Executor 机制**

ADK 通过

```
CodeExecutor
```

接口将 LLM 生成的代码发送到沙盒环境运行。

| 执行器 | 底层机制 | 适用场景 | 安全性 |
| --- | --- | --- | --- |
| BuiltInCodeExecutor | Docker 容器或本地受限进程 (gVisor) | 开发测试、简单脚本、本地工具 | ⭐⭐⭐ (依赖环境配置) |
| AgentEngineSandboxCodeExecutor | Google Cloud 全托管沙盒 (MicroVMs) | 生产环境、企业级应用、高并发 | ⭐⭐⭐⭐⭐ (硬件级隔离) |

> **_⚠️ 安全警示_**_: 即使是本地沙盒，也绝对_**_禁止_**_将 Code Executor 开放在未受信任的输入源（如允许用户直接让 Agent 执行任意代码）。这是 "Prompt Injection" 的高危区。_

**3\. 进阶知识点 (Deep Dive)**

🔄 The "Self-Correction" Loop (自我纠错循环)

这是 Agent 最强大的能力之一。当代码执行报错时，Agent 能够：

1.  **捕获 Stderr**: 读取 Traceback 错误信息。
    
2.  **Analysis**: 即时分析错误原因（是语法错误、库缺失，还是逻辑漏洞？）。
    
3.  **Rewrite**: 重新编写代码。
    
4.  **Retry**: 再次执行。
    

这种"试错"能力让 Agent 能够解决它从未见过的复杂问题。

💾 State Persistence (状态持久化)

代码执行通常是**无状态**的（Stateless），特别是在

```
BuiltInCodeExecutor
```

中。

-   **Block A**:
    
    ```
    x = 5
    ```
    
-   **Block B**:
    
    ```
    print(x)
    ```
    
    \-> **NameError!**
    

**解决方案**:

-   **单次执行**: 尽量将逻辑封装在一个完整的代码块中。
    
-   **Session Context**: 高级执行器（如 Agent Engine Sandbox）支持 Session 级别的内存持久化，允许跨代码块共享变量。
    

### **💡 Week 1 总结 & Week 2 展望**

回顾这一周，我们完成了从"零"到"准生产"的旅程：

-   Day 01-02: 基础与配置
    
-   Day 03-04: Gemini 模型与云端部署
    
-   Day 05-06: 可观测性与工具集成
    
-   **Day 07**: 自主代码执行 (Agent 的"大脑"长出了"手")
    

**Next (Week 2): Context & Orchestration** 我们将解决 Agent 的"记忆"问题。如何让 Agent 记住长对话？如何管理庞大的文档上下文？Day 08 将深入 **Effective Context Management**。
<!-- DAILY_CHECKIN_2026-01-03_END -->

# 2026-01-02
<!-- DAILY_CHECKIN_2026-01-02_START -->





**📅 Day 06 打卡：ADK Ready & Context Engineering**

**📝 核心收获** 今天不写代码，而是“磨刀”。从手搓代码转向了 **Agent 工程化** 思维。

**🚀 工具链解锁**

1.  **Antigravity (IDE)**: 不止是编辑器，而是能自主规划任务的 AI 伙伴。
    
2.  **Context Files (**
    
    [**GEMINI.md**](http://GEMINI.md)**)**: 本日MVP。这是写给 AI 看的项目说明书（System Prompt 扩展包）。只要维护好这个文件，新来的 AI 助手立马就能懂我的代码规范和技术栈。
    
3.  **Agent Starter Pack**: Google 官方的 Agent 脚手架，生产级标准，开箱即用。
    

**💡 认知升级 (Aha Moment)**

-   **Neuro-Symbolic**: 用代码逻辑（Symbolic）去约束发散的大模型（Neuro）。
    
-   未来的核心竞争力：**Context Engineering**。谁能给 AI 提供更精准的上下文，谁就能驾驭更强的 Agent。
    

**🔜 Next Step**: Day 07，赋予 AI 执行代码的能力！(LLMs Can Execute Code)

#GoogleAI #Agent开发 #GEMINI #100DaysOfCode
<!-- DAILY_CHECKIN_2026-01-02_END -->

# 2026-01-01
<!-- DAILY_CHECKIN_2026-01-01_START -->






# 📅 2026-01-01 Day 05 学习日记

## 📝 学习主题：Production Observability (Agent Starter Pack)

新年第一天，学习了 Google Cloud 的 **Agent Starter Pack**，重心放在了 Agent 的**生产级可观测性（Observability）**。

### 💡 核心收货

1.  **Agent Starter Pack 的价值**：
    
    -   它不仅仅是一个模板，而是一套**工程化标准**。通过 `agent-starter-pack create` 可以快速生成包含后端、部署配置和观测性代码的项目，极大提升了从 Demo 到生产的效率。
        
2.  **可观测性的两层深度理解**：
    
    -   **Agent Telemetry (Cloud Trace)**：这是"骨架"。记录了请求怎么走、每个工具跑了多久。
        
    -   **Prompt-Response Logging (GCS/BigQuery)**：这是"细节"。记录了 AI 到底说了什么，用了多少 Token。
        
    -   **隐私权衡**：学到了 `NO_CONTENT` 模式，在记录元数据进行成本/性能监控的同时，保护用户隐私。
        
3.  **ReAct 模式的实战感悟**：
    
    -   通过本地 Playground 的交互，清晰地看到了模型如何自主决定调用 `get_weather`。这种"思考 -> 行动 -> 观察 -> 总结"的闭环是当前最实用的 Agent 范式。
        

### 🚀 对 Prediction Copilot 的启发

-   **决策溯源**：在预测市场中，AI 为什么判断为 YES 必须是可追溯的。利用 Cloud Trace 这种工具，可以把"新闻检索 -> 鲸鱼分析 -> 逻辑推理"这个过程可视化，增加产品的透明度和信任度。
    
-   **成本控制**：通过观测性面板看到的 Token 统计，可以帮助精准核算每次分析的成本，从而制定合理的定价或使用限制策略。
    

### 🔧 技术细节记录

-   **环境配置**：遇到 SSL 问题，通过 `venv` 环境和 `uv` 工具成功解决。
    
-   **配置切换**：学会了如何在 `agent.py` 中通过环境变量 `GOOGLE_GENAI_USE_VERTEXAI` 在 GCP Vertex AI 和 Google AI Studio 之间快速切换认证模式。
    

### 🌟 总结

如果说前几天是在学"怎么做 Agent"，今天是学"怎么管 Agent"。在一个真实的生产项目中，能看清 Agent 的运行状态比做对一个功能更重要。

* * *

**下一站**：Day 06 - ADK Ready (Antigravity, Gemini CLI, Cursor, Firebase)
<!-- DAILY_CHECKIN_2026-01-01_END -->

# 2025-12-31
<!-- DAILY_CHECKIN_2025-12-31_START -->







# **Day 04 学习笔记: Source-Based Deployment (基于源的部署)**

## **1\. 核心问题：为什么需要 Source-Based Deployment？**

### **传统部署方式的痛点**

传统 Agent 部署通常使用 **序列化 (Serialization)**，如 Python 的 `pickle` 或 `cloudpickle`：

```
# 传统方式：把整个 Agent 对象"腌制"成二进制文件
```

import cloudpickle

with open("agent.pkl", "wb") as f:

cloudpickle.dump(my\_agent, f)

**问题**：

1.  **Python 版本敏感**：本地 3.11，云端 3.10？💥 报错
    
2.  **依赖地狱**：本地 numpy 1.24，云端 1.21？💥 不兼容
    
3.  **调试困难**：出错后看到的是一堆二进制乱码
    
4.  **安全隐患**：`pickle` 可被利用执行恶意代码
    

### **Source-Based 的解法**

**直接上传源码文件**，让云端从头重建 Agent：

```
本地代码 → 打包 .tar.gz → 上传 → 云端 pip install → 云端 import agent
```

* * *

## **2\. 核心 API 解析**

### `AdkApp`**: Agent 的"容器"**

```
from vertexai.preview.reasoning_engines import AdkApp
```

adk\_app = AdkApp(

agent=retail\_agent, # 你的 ADK Agent

enable\_tracing=True, # 启用 Cloud Trace 追踪

)

-   **作用**：把 ADK Agent 包装成一个符合 Vertex AI 规范的"应用"
    
-   **类比**：就像把你的代码放进一个标准化的 Docker 容器
    

### `agent_engines.create()`**: 部署命令**

```
from vertexai import agent_engines
```

remote\_agent = agent\_engines.create(

adk\_app, # AdkApp 实例

display\_name="my\_agent", # 显示名称

requirements=\["google-adk", ...\], # pip 依赖

extra\_packages=\["./day03"\], # 本地源码目录

)

**关键参数**：

| 参数 | 作用 |
| --- | --- |
| requirements | Python 依赖列表，等同于 requirements.txt |
| extra_packages | 本地源码目录，会被打包上传 |
| env_vars | 环境变量（如 API Key） |

* * *

## **3\. 部署流程图解**

```
┌──────────────────┐
```

│ 本地开发环境 │

│ day03/[agent.py](http://agent.py) │

└────────┬─────────┘

│ AdkApp 包装

▼

┌──────────────────┐

│ agent\_engines │

│ .create() │

└────────┬─────────┘

│ 打包上传

▼

┌──────────────────┐

│ Cloud Storage │

│ (Staging Bucket)│

└────────┬─────────┘

│ Agent Engine 拉取

▼

┌──────────────────┐

│ Agent Engine │

│ pip install → │

│ import agent → │

│ 运行服务 │

└──────────────────┘

* * *

## **4\. 为什么这种方式更好？**

| 特性 | Pickle 序列化 | Source-Based |
| --- | --- | --- |
| 版本兼容性 | ❌ 易出错 | ✅ 云端重建 |
| 调试体验 | ❌ 二进制难读 | ✅ 看源码 |
| CI/CD 友好 | ❌ 难以集成 | ✅ 像部署普通应用 |
| 安全性 | ❌ pickle 风险 | ✅ 标准代码 |

* * *

## **5\. 代码已就绪 (无需运行)**

我们为 Day 04 创建了完整的部署脚本：

-   [**deploy.py**](http://deploy.py): 完整的 create/list/delete 逻辑
    
-   **.env.example**: 环境变量模板
    

**未来如需实际部署**，只需配置 GCP 项目和 Bucket，运行：

```
python day04/deploy.py --create
```

* * *

## **6\. 关键 Takeaway**

1.  **AdkApp 是桥梁**：连接 ADK Agent 和 Vertex AI Agent Engine
    
2.  **extra\_packages 是核心**：你的源码通过它被打包上传
    
3.  **requirements 定义环境**：确保云端有正确的依赖
    
4.  **Source > Pickle**：现代部署的最佳实践
<!-- DAILY_CHECKIN_2025-12-31_END -->

# 2025-12-30
<!-- DAILY_CHECKIN_2025-12-30_START -->








# **Day 03 学习笔记: Gemini 3 与 神经符号智能体 (Neuro-Symbolic Agents)**

## **1\. 核心理念: 神经符号 AI (Neuro-Symbolic AI)**

Day 03 标志着我们从“聊天机器人”迈向了“解决问题的智能体”。

-   **Neuro (神经)**: **Gemini 3 Pro**。大模型作为“大脑”，负责理解非结构化的用户意图（“我要开店”）、制定高层战略（“需要做差异化竞争”）。
    
-   **Symbolic (符号)**: **代码执行 (Code Execution)**。Python 解释器作为“计算器”，负责处理结构化数据、精确计算（平均分 4.46）、逻辑推导。
    

**价值**: 这种结合解决了 LLM 的两大死穴：**幻觉 (Hallucination)** 和 **计算能力弱 (Poor Math)**。

## **2\. 架构设计 (Architecture)**

我们的 Retail Agent 采用了经典的 **ReAct (Reason + Act)** 变体模式：

1.  **感知 (Perception)**: 用户输入 "分析 KR Puram 的健身房竞争"。
    
2.  **行动 1 (Tool 1)**: 调用
    
    **search\_places** 获取外部数据（Competitor Data）。
    
3.  **推理 (Reasoning)**: 拿到数据后，发现需要量化分析。
    
4.  **行动 2 (Tool 2)**: 编写并调用
    
    **execute\_code**，计算市场饱和度、加权评分。
    
5.  **决策 (Decision)**: 综合数据与常识，输出最终战略报告。
    

## **3\. 工程实践 (Engineering Highlights)**

### **A. 模拟数据 (Mocking Strategy)**

由于没有 Google Maps Billing 权限，我们采用了 **Mock（模拟）** 策略。

-   **实现**: 编写
    
    **mock\_**[**data.py**](http://data.py) 和假的
    
    **search\_places** 工具。
    
-   **启发**: 在 Agent 开发初期，不要被外部 API 卡住。先用 Mock 数据跑通逻辑，架构稳定后再接入真实 API。
    

### **B. 自定义工具 (Custom Tools)**

官方 ADK 的 `BuiltInCodeExecutor` 与部分模型存在兼容性问题（400 Error）。

-   **解决方案**: 我们手动实现了一个
    
    **execute\_code(code: str)** 工具，底层使用 Python 的
    
    **exec()**。
    
-   **启发**: Agent 的本质只是 LLM + Functions。不必迷信官方封装，只要能把函数描述清楚（Docstring）并正确返回结果，任何代码都可以成为 Agent 的手。
    

### **C. 模型探索 (Model Discovery)**

我们发现即便 UI 显示 "Gemini 3 Pro"，API 标识符可能不同。

-   **Gemini 3 Flash**: `models/gemini-3-flash-preview`
    
-   **Gemini 3 Pro**: `models/gemini-3-pro-preview`
    
-   **经验**: 永远使用 `client.models.list()` 来确认当前可用的真实 Model ID，而不是盲猜。
    

## **4\. 关键成果 (Key Outcomes)**

-   **Data-Driven Insight**: Agent 不是瞎编建议，而是基于算出来的 "4.46 平均分" 警告我们 "市场竞争极度激烈"。
    
-   **Autonomous Chaining**: Agent 自主完成了 `Search` -> `Code` 的数据流转，全程无需人类干预。
    

* * *

## **5\. 代码片段 (Golden Snippets)**

**Agent 如何自我思考并写代码：**

```
# Agent 生成的用来分析市场的代码
```

import pandas as pd

data = \[...\] # 自动注入了上一步搜索到的数据

\# 自动计算加权平均分

df\['weighted\_score'\] = df\['rating'\] \* df\['reviews'\]

overall\_weighted\_avg = df\['weighted\_score'\].sum() / total\_reviews

\# 自动进行细分市场切片

market\_leaders = df\[(df\['rating'\] >= 4.5) & (df\['reviews'\] >= 100)\]

niche\_players = df\[(df\['rating'\] >= 4.5) & (df\['reviews'\] < 100)\]

这段逻辑完全由 Gemini 3 Pro 自主生成，证明了其强大的逻辑推理能力。
<!-- DAILY_CHECKIN_2025-12-30_END -->

# 2025-12-29
<!-- DAILY_CHECKIN_2025-12-29_START -->









````markdown
# Day 02: Introduction to Declarative Agents (2025-12-29)

## 🎯 今日目标 (Goal)
- 学习 **Declarative Agents (声明式智能体)**：无需编写 Python 代码，通过 YAML 配置文件即可定义 Agent。
- 掌握 **Google Agent Development Kit (ADK)** 的基础命令：CLI 与 Web UI。
- 解决模型配额 (Quota) 与工具调用 (Tool Use) 的兼容性挑战。

## 📝 核心知识点 (Key Concepts)

### 1. YAML Configuration
Agent 的“灵魂”都在 `root_agent.yaml` 中定义：
```yaml
name: search_agent
model: gemini-2.5-flash  # 选择适合自己配额的模型
description: A helpful assistant that can search the web.
instruction: |
  You are a helpful assistant.
  Use Google Search for current events and factual information.
tools:
  - name: google_search  # 内置工具，声明即可使用
```

### 2. 运行模式
- **命令行 (CLI)**: 快速测试
  ```bash
  venv/bin/adk run day02
  ```
- **网页界面 (Web UI)**: 图形化交互与调试
  **关键点**: 必须在**项目根目录**运行，否则找不到 Agent！
  ```bash
  venv/bin/adk web .
  ```
  *(注: 在 Web UI 左上角下拉菜单选择 `day02` 即可开始对话)*

## 🛠️ 踩坑与解决方案 (Troubleshooting)

### 1. 模型选择与配额 (429/404 Errors)
- **现象**: `gemini-1.5-flash` 报 404 (Not Found)，`gemini-2.0-flash` 报 429 (Resource Exhausted).
- **原因**: 账号处于 Early Access 阶段，旧模型未授权，新模型无免费配额。
- **解决**: 使用 `list_models` 脚本扫描可用模型，最终锁定 **`gemini-2.5-flash`** 或 **`gemini-3-flash-preview`** (取决于账号权限)。

### 2. 工具兼容性 (400 Invalid Argument)
- **现象**: `Tool use with function calling is unsupported`.
- **原因**: ADK 当前版本在封装 `gemini-2.5` 系列请求时，如果混合使用 Python 工具与 Google Search，可能触发协议不兼容。
- **解决**: 暂时移除自定义 Python 工具，仅保留 `google_search`，回归课程最基础的“纯 YAML + 搜索”配置，成功跑通。

### 3. Web UI 找不到 Agent
- **现象**: `Warning: No agents found in current folder`.
- **原因**: 在 `day02` 目录下运行 `adk web .`，ADK 会去搜子目录而忽略当前目录。
- **解决**: 回到项目根目录 (`cd ..`)，运行 `adk web .`，ADK 就能正确扫描到 `day02` 作为一个 App。

## ✅ 成果 (Outcome)
成功创建了一个只需几行 YAML 配置就能通过 Google Search 回答实时问题（如“谁是 Leo”）的智能体，跑通了全流程。

---
*Next: Day 03 - Gemini Search Agent (Deep Dive into Grounding)*
````
<!-- DAILY_CHECKIN_2025-12-29_END -->

# 2025-12-28
<!-- DAILY_CHECKIN_2025-12-28_START -->










**\[Day 01\] Google AI Agent 开发环境搭建与初探**

**📅 日期**：2025-12-28 **🎯 目标**：跑通 Google ADK 基础流程，解决网络与鉴权问题。

**1\. 技术栈概览**

-   **Framework**: Google ADK (Agent Development Kit)
    
-   **Model**: Gemini 1.5 Flash (平衡速度与成本)
    
-   **Runtime**: Python 3.11
    

**2\. 核心步骤记录**

-   **依赖安装**：如果不使用 uv/poetry，直接
    
    ```
    pip install
    ```
    
    核心包，注意
    
    ```
    pydantic
    ```
    
    的版本兼容性。
    
-   **代理配置 (Crucial)**：
    
    -   国内开发需注意 SDK 的连接问题。
        
    -   解决方案：在代码入口处显式注入
        
        ```
        http_proxy
        ```
        
        /
        
        ```
        https_proxy
        ```
        
        环境变量（指向本地代理端口）。
        

**3\. 遇到的坑与解决**

-   **Runner 接口**：
    
    ```
    Runner.run()
    ```
    
    在新版 ADK 中推荐使用
    
    ```
    run_debug()
    ```
    
    进行快速的单轮/多轮对话测试，更加直观。
    
-   **鉴权**：
    
    **.env** 文件必须配置
    
    ```
    GOOGLE_API_KEY
    ```
    
    ，且需要安装
    
    ```
    python-dotenv
    ```
    
    自动加载。
    

**4\. 下一步计划**

-   深入研究
    
    ```
    SequentialAgent
    ```
    
    (顺序执行 Agent)。
    
-   探索 Tool Use (工具调用)，让 Agent 具备联网或执行代码的能力。
<!-- DAILY_CHECKIN_2025-12-28_END -->
<!-- Content_END -->
